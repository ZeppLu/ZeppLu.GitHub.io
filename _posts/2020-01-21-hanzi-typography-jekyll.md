---
title: "印刷品般的漢字排版框架inside Jekyll"
header:
  image: https://i.loli.net/2020/01/21/5IOZSDpWk1B9y7R.png
categories:
  - 技术
tags:
  - 前端
  - Jekyll
  - 排版
  - 汉字
---

常言道，国际范儿与接地气儿不可得兼，在开源项目上尤为如此：
一个项目要获得世界范围内的开发者参与，就必须以英文为主体；
但是在本国开发者与用户数量占弱势的情况下，又难以针对特色国情进行优化。
排版（typography）就是其中一点，作为现存唯一强势的表意文字，
中文方块字天生被赋予了与印欧语系诸文字不同的角色，
它的定宽、高密度属性使其在一串串可拆分、可变宽的符号中格格不入。
不妨欣赏一下最原生态的中英混合排版：

![看不到图说明sm.ms又被墙了](https://i.loli.net/2020/01/21/mTSeBbLgwvc4C3D.png)

[「漢字標準格式」](https://hanzi.pro/)是台湾人[陈奕钧](https://yijun.me/)发起的开源项目
（他也是W3C东亚文字网页排版新标准制定的参与者之一），
目的是借助CSS与JavaScript，在浏览器中实现较为理想的中文排版效果。
要使用该项目，只需要在**所有其他样式表前**添加

```html
<link rel="stylesheet" media="all" href="//cdnjs.cloudflare.com/ajax/libs/Han/3.3.0/han.min.css">
```

并在HTML的body标签尾部添加

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/Han/3.3.0/han.min.js"></script>
```

但需要注意！这只是引入了CSS和JavaScript，具体的渲染过程还需要人工启动（前端小白表示自己太菜了）。
具体可选的渲染方式参考[这里](https://hanzi.pro/manual/js-api#rendering)，
我主要的目的是让正文排版起来顺眼一点，而其它地方只需要中英文之间有间隔就足够了，所以我的方法是：

```html
<script>
  Han(document.body).initCond().renderHWS();
  Han(document.querySelector('.page__content')).render();
</script>
```

完成效果图如下，注意中英文间的间隔（通过CSS插入的，而不是空格）以及行尾文字与标点的对齐。

![看不到图说明sm.ms又被墙了](https://i.loli.net/2020/01/21/zqTpvJ53nbkRWID.png)
