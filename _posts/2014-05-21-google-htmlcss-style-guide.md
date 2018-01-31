---
layout: post
title: "Google HTML/CSS Style Guide"
description: ""
category: code
tags: [代码规范, Google, HTML/CSS, style guide]
---


参译：<https://google-styleguide.googlecode.com/svn/trunk/htmlcssguide.xml>

今天在写HTML代码时稍稍反省了一下代码风格，所以决定抽空学习一下Google的代码风格标准。

## 一般风格规则

### http/https协议

对于http和https协议，建议使用双斜杠代替

对于HTML代码：

    <!-- 不建议使用 -->
    <script src="http://www.google.com/js/gweb/analytics/autotrack.js"></script>

    <!-- 建议使用 -->
    <script src="//www.google.com/js/gweb/analytics/autotrack.js"></script>

对于CSS代码：

    /* 不建议使用 */
    .example {
      background: url(http://www.google.com/images/example);
    }

    /* 建议使用 */
    .example {
      background: url(//www.google.com/images/example);
    }

## 一般格式规则

### 缩进

**不建议**使用`TAB`或者混合`TAB & 空格`作为缩进，建议使用**两个空格**作为缩进：

对于HTML代码：

    <ul>
      <li>Fantastic
      <li>Great
    </ul>

对于CSS代码：

    .example {
      color: blue;
    }

### 大小写

建议仅使用小写：包括HTML元素名称、属性、属性值（`text/CDATA`除外），CSS选择器、属性和属性值（不含字符串）

对于HTML代码：

    <!-- 不建议使用 -->
    <A HREF="/">Home</A>

    <!-- 建议使用 -->
    <img src="google.png" alt="Google">

对于CSS代码：

    /* 不建议使用 */
    color: #E5E5E5;

    /* 建议使用 */
    color: #e5e5e5;


### 尾部空格

去除尾部空格，否则会使diff复杂

对于HTML代码：

    <!-- 不建议使用 -->
    <p>What?_

    <!-- 建议使用 -->
    <p>Yes please.

## 一般meta规则

### 编码

使用UTF-8(无BOM)进行编码。使用`<meta charset="utf-8>` 来指定编码格式，不要不指定默认它为`UTF-8`

### 注释

善用注释，明确其功能和选择使用的原因

### 操作项

仅使用`TODO`来高亮`TODO`的工作，而不是使用其他常见的代码注释格式

以下两个例子均可

    {# TODO(john.doe): revisit centering #}
    <center>Test</center>

    <!-- TODO: remove optional tags -->
    <ul>
      <li>Apples</li>
      <li>Oranges</li>
    </ul>

## HTML代码风格规范

### Document Type

使用HTML5风格 `<!DOCTYPE html>`

推荐使用的HTML，为`text/html`。不要在XHTML、 XHTML使用`application/xhtml+xml`，因为他既缺乏浏览器和基础设施的支持，并且相对HTML而言，他提供了更小的优化空间。

另外，对于HTML来说，某些标签的表示结果都一样，但请不要使用自闭合空标签，例如：推荐`<br>`，而非`<br />`

### HTML Validity




























