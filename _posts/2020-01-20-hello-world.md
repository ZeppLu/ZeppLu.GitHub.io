---
title: "Discovery One: a Planning Playground for Enthusiasts of Mathematics, Physics, as well as Computer Science"
categories:
  - 技术
tags:
  - 前端
  - Jekyll
music: "https://music.163.com/song?id=1218773"
---

## Introduction

终于，也轮到我在自己的博客上写下hello world了。
开通这个博客的主要目的，还是记录所学的各种杂七杂八的东西，
毕竟人老了脑子隔三岔五就会淡忘，辛苦整理的笔记最终也难逃遗失的命运，
但git不会背叛你，commit & push之后，它就能持久存在于MOSFET阵列中，忠诚地等待下一个pull。

作为一个前端纯小白，在利用GitHub pages搭建静态博客，
并挑选合适的remote theme加上自己的一点配置，让博客堪堪够看的这个过程中，
着实花费了不少时间，弄清楚了一大堆以后可能都用不着的概念。
心中念着搞博客的初心，决定把踩的坑记录一下，以飨读者（如果有的话）。

## Jekyll

[Jekyll](https://jekyllrb.com/)是一个用Ruby写的工具，
能够根据HTML与CSS模板以及markup language写成的posts，生成整个静态网站。
GitHub pages自带Jekyll支持，再加上remote theme功能，用户可以只在repo里存放Markdown文件和自定义配置，
将网站生成的任务交给GitHub完成。

从Markdown文件生成HTML的过程其实并不复杂。首先在Markdown的顶部加入一些YAML配置（front matters）：
```yaml
---
layout: single
toc: true
---
```
其中最重要的是layout属性（其余属性可作为HTML生成过程中的变量），
这将指导Jekyll把`_layouts/single.html`作为HTML模板，
然后通过一系列变量替换、if-else语句判断、include文件的操作，生成静态HTML文件。

CSS文件则借助于[Sass](https://sass-lang.com/)，粗略地讲，可以认为这是一个CSS的模板，
有着变量、包含、展开、函数计算等功能。
相关的Sass文件都放在`_sass`下，经过编译生成单一的`min.css`文件会被放置在每一个静态网页中。

## Theme: Minimal Mistakes

Jekyll主题其实就是上面所说的那一堆HTML与CSS模板。
权衡多方后，我最终选用了[Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes)这个主题，
原因也很简单：可定制选项丰富的同时，还提供了内容详实的文档（再次说明了文档对开源项目的作用）。
它最棒的是，作者在编写了[文档](https://mmistakes.github.io/minimal-mistakes/)的同时，
还额外给出了一个实际利用GitHub pages + remote theme搭建而成的
[demo博客](https://mmistakes.github.io/mm-github-pages-starter/)，可谓感人肺腑，救小白于水火之中
（虽说后来弃用了remote theme，但这个demo还是起到了很大的帮助）

## Improvements

依旧是前端小白的艰难探索：

- [改进汉字与拉丁文字的混合排版](/articles/hanzi-typography-jekyll/)；
- [数学公式渲染](/articles/katex-jekyll/)；
- [音乐播放器](/articles/music-player/)；
- [弃用remote theme](https://github.com/ZeppLu/ZeppLu.GitHub.io/commit/97c59ebaa4303622d7b778a4d21952fdf5742229)，这还顺便解决了一些边边角角的小问题。至于主题的更新，完全可以效仿早期的Linus，手动打patch升级。