---
title: "Lightweight Math Rendering inside Jekyll"
categories:
  - 技术
tags:
  - 前端
  - Jekyll
  - KaTeX
---

[MathJax](https://www.mathjax.org/)当前已经成为HTML中数学公式渲染引擎的de facto standard，
但出于种种原因，我更愿意选用名字听起来更专业的[KaTeX](https://katex.org/)。
冠冕堂皇的理由可以有很多，
比如[更快的渲染速度](https://www.intmath.com/cg5/katex-mathjax-comparison.php)、
更LaTeX-compliant的语法集、
更全的奇奇怪怪命令（如`\mathclap`）的支持等等，
不过在我看来，抛弃MathJax的借口有一点就已足够：公式被点击时周围会出现难以忍受的蓝框。

KaTeX的优势之一便是server side rendering，减轻用户负担，极大地提升了页面加载速度。
插件[jekyll-katex](https://github.com/linjer/jekyll-katex)也确实做到了这一点，
不过它的致命弱点是不兼容GitHub pages，which不支持Jekyll插件……
此外，它还要求将Markdown中的数学公式用Liquid标签包裹起来，which也降低了写博文的体验。

KaTeX开发者当然也意识到，不能完全依赖server side，
于是[auto-render扩展](https://katex.org/docs/autorender.html)应运而生。
所谓的扩展实质上是一个JavaScript脚本，它负责分析传入的HTML节点，
通过给定的参数（左分隔符、右分隔符等）搜索出数学公式的位置，并进行渲染。
它的问题也很明显：
server side甚至不需要做特殊的工作，照原样把包含分隔符在内的TeX公式放在`<p>`里就好了，
将公式的定位都交给了client side来做。
这一方面违背了我们对渲染速度的追求，另一方面也没有充分薅到GitHub的羊毛。

于是，无产程序员们找到了压榨资本家剩余价值的方法：
server side把TeX公式放在特殊的节点中，client side只对这些节点进行渲染
[^1]<sup>,</sup>[^2]<sup>,</sup>[^3]<sup>,</sup>[^4]。
具体地讲，分为以下几步：

1. 欺骗Jekyll，让它以为你要用MathJax渲染，这样它就会把公式放在`<script type="math/tex">`中：

   ```yaml
   markdown: kramdown
   kramdown:
     math_engine: mathjax
   ```

   不过现在GitHub pages已经默认用这个配置build了，所以这步可跳过。

2. 当然要加载KaTeX的样式表和脚本：

   ```html
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.css">
   <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js"></script>
   ```

3. 最后对页面中的`<script type="math/tex">`都渲染一遍：

   ```javascript
   $("script[type='math/tex']").replaceWith(function () {
       tex = $(this).text();
       return katex.renderToString(tex, { displayMode: false });
   });
   $("script[type='math/tex; mode=display']").replaceWith(function () {
       var tex = $(this).html();
       return katex.renderToString(tex.replace(/%.*/g, ''), { displayMode: true });
   });
   ```

   有些地方[^2]说要将公式块的渲染结果放在`<div>`中，实测不放也没影响（前端小白真的不懂）。
   Anyway，准官方的扩展[^4]是处理到`<div>`里的。

最终的改动之处是HTML`<body>`的底部，可以放在`_layouts/default.html`或`_includes/scripts.html`中：

```html
<script>
  function renderMathUsingKatex() {
    $("script[type='math/tex']").replaceWith(function () {
      var tex = $(this).text();
      return katex.renderToString(tex, { displayMode: false });
    });
    $("script[type='math/tex; mode=display']").replaceWith(function () {
      var tex = $(this).html();
      return katex.renderToString(tex.replace(/%.*/g, ''), { displayMode: true });
    });
  }
</script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js"
              onload="renderMathUsingKatex()"></script>
```

嗯，看起来很完美，甚至还用到了`defer`和`onload`（没啥用的知识+1）。
然而就在我写好这些代码后，继续网上冲浪的时候，
却发现了准官方的[mathtex-script-type扩展](https://github.com/KaTeX/KaTeX/tree/master/contrib/mathtex-script-type)，这个完全替代了上面代码功能的小玩意儿。
考虑再三后，决定还是忍痛割爱，换成了这个扩展。
而一个额外的收获是顺藤摸瓜找到的[copy-tex扩展](https://github.com/KaTeX/KaTeX/tree/master/contrib/copy-tex)，
它能让你轻松地复制公式的原始TeX输入，欢迎在下面的示例中摸索👇

$$\sum_{n=1}^{\infty} n = -\frac{1}{12}$$[^5]
是数学民科最喜欢哗众取宠的领域之一，
但深究起来它其实牵涉到数学王冠上的其中一颗明珠——[黎曼猜想](https://en.wikipedia.org/wiki/Riemann_hypothesis)，
进而又与[素数定理](https://en.wikipedia.org/wiki/Prime_number_theorem)相关，
是一个非常有趣的科普领域。
素数定理涉及的是素数在自然数中的分布情况，只要是对它稍有了解的人，
相信都能看懂以下式子[^6]所代表的含义：

$$
\lim_{\mathclap{\substack{
    \left\lvert z \right\rvert\to\infty \\
    \arg(z)=\pm\frac{\pi}{2}
  }}} \;
\frac {\sum_{\scriptscriptstyle p\le\left\lvert z\right\rvert} \! 1} {\operatorname{Im}(z)}
$$

[^1]: https://xuc.me/blog/katex-and-jekyll/ （初次提出者，果然中国人善薅羊毛）
[^2]: https://nealde.github.io/blog/2017/10/20/How-to-make-a-local-Jekyll-website/
[^3]: https://www.amirasiaee.com/dailyreport/katex-for-jekyll/
[^4]: https://github.com/KaTeX/KaTeX/tree/master/contrib/mathtex-script-type （准官方，但太繁琐，~~不采用~~ 还是用了）
[^5]: 注意这个inline math需要用`$$ tex $$`框起
[^6]: 注意这个math block是用`$$ \n tex \n $$`框起的