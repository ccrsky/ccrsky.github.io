---
title: Handlebars模板使用
date: 2017-08-15 00:09:36
tags:
    - js

category: 
    - js
---


[Handlebars](http://handlebarsjs.com/) 是 JavaScript 一个语义模板库，通过对view和data的分离来快速构建Web模板。

![](http://ouonaooa5.bkt.clouddn.com/handlebarjs.jpg)

### 安装

- 直接下载 [Handlebars.js](http://builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v4.0.10.js) 和 [Handlebar Runtime](http://builds.handlebarsjs.com.s3.amazonaws.com/handlebars.runtime-v4.0.10.js)
- 通过npm下载

```bash
# 安装
npm install --save handlebars

# 引用
require('handlebars');

# 如果模板预编译后,可直接使用运行时环境
require('handlebars/runtime');
```



### 快速入门

#### Handlebar表达式

Handlebars会从当前上下文中查找title和body属性

```html
<div class="entry">
  <h1>{{title}}</h1>
  <div class="body">
    {{body}}
  </div>
</div>
```

#### 路径查询
Handlebars会从当前上下文中查找article属性,然后在查找结果中找title属性

``` html
<h1>{{article.title}}</h1>
<!-- 两者等价，下面方法deprecated-->
<h1>{{article/title}}</h1>

```
>标识符不能包含 Whitespace ! " # % & ' ( ) * + , . / ; < = > @ [ \ ] ^ ` { | } ~

如果出现不合法的属性，可以使用中括号的方式进行引用

``` html
{{#each articles.[10].[#comments]}}
  <h1>{{subject}}</h1>
  {{{body}}}
{{/each}}
```
>默认情况下 Handlebars为了安全，会对内容进行转义，如需原样输出可以使用{%raw %}`{{{ html }}}`{%endraw%}

#### Helper模板方法

```js
{{{link story}}}

Handlebars.registerHelper('link', function(object, options) {
  var url = Handlebars.escapeExpression(object.url),
      text = Handlebars.escapeExpression(object.text);

  return new Handlebars.SafeString(
    "<a href='" + url + "'>" + text + "</a>"
  );
});
```
Handlebars helper支持接收一个可选的键值对参数作为它的最后一个参数传入(hash)

```js
{{{link "See more..." href=story.url class="story"}}}
Handlebars.registerHelper('link', function(text, options) {
  var attrs = [];

  for (var prop in options.hash) {
    attrs.push(
        Handlebars.escapeExpression(prop) + '="'
        + Handlebars.escapeExpression(options.hash[prop]) + '"');
  }

  return new Handlebars.SafeString(
    "<a " + attrs.join(" ") + ">" + Handlebars.escapeExpression(text) + "</a>"
  );
});
```
Helper 支持嵌套

``` html
{{outer-helper (inner-helper 'abc') 'def'}}
```

#### 去掉空格

去掉多余空格(在大括号边添加 ~)

``` html
{{#each nav ~}}
  <a href="{{url}}">
    {{~#if test}}
      {{~title}}
    {{~^~}}
      Empty
    {{~/if~}}
  </a>
{{~/each}}
```

#### 标签转义
使用标签原内容

```html
<!-- 在前边添加 \ -->
\{{escaped}}

<!--使用4个大括号-->
{{{{raw}}}}
  {{escaped}}
{{{{/raw}}}}
```


### 编译模板

#### 模板字符串

```html
<script id="entry-template" type="text/x-handlebars-template">
  <div class="entry">
    <h1>{{title}}</h1>
    <div class="body">
      {{body}}
    </div>
  </div>
</script>
```
使用 `Handlebars.compile`编译模板输出模板函数

```js
var source   = $("#entry-template").html();
var template = Handlebars.compile(source);

// 如果预先生成模板函数了，则可以直接引用runtime库，不必将handlebar全部引入
```

#### 命令行工具
```bash
npm install handlebars -g

# 语法 
handlebars <input> -f <output>

# 查看更多使用方法
handlebars --help 

#如果输入文件是 person.handlebars，Handlebar会将文件内容作为compile方法的第一个参数，生成一个模板方法，插入到 Handlebars.templates.person上，之后可直接使用 Handlebars.templates.person(context,options)

# 若预编译，直接使用runtime库 <script src="/libs/handlebars.runtime.js"></script>
```

### 高级使用

#### complile 函数

```js
模板函数接收第二个可选参数，传以下参数:
data:传入私有变量 @variable 
helpers:传入局部helper，而不是全局定义，重名会覆盖全局。
partials: 自定义模板片段partials，与helper类似
```

#### 块级表达式
块级表达式可以改变模板局部context，块级helper以#开口，结尾以/开发，如{%raw%}`{{#hi}} {{/hi}}`{%endraw%}

```js
<div class="entry">
  <h1>{{title}}</h1>
  <div class="body">
    {{#bold}}{{body}}{{/bold}}
    {{#noop}} hello {{/noop}}
  </div>
</div>

Handlebars.registerHelper('noop', function(options) {
  return options.fn(this);
});
// 其中this执行当前上下文 options.fn 就像普通的模板函数一样，如果块级helper含有else部分，和fn类似的还有一个 options.reverse 渲染else里的模板。

Handlebars.registerHelper('bold', function(options) {
  return new Handlebars.SafeString(
      '<div class="mybold">'
      + options.fn(this)
      + '</div>');
});


<!-- with 会改变当前上下文 -->
{{#with story}}
<div class="intro">{{{intro}}}</div>
<div class="body">{{{body}}}</div>
{{/with}}



<!-- 使用each 迭代 -->
{{#each comments}}
<div class="comment">
  <h2>{{subject}}</h2>
  {{{body}}}
</div>
{{/each}}

// each 原理
Handlebars.registerHelper('each', function(context, options) {
  var ret = "";
  for(var i=0, j=context.length; i<j; i++) {
    ret = ret + options.fn(context[i]);
  }
  return ret;
});



<!-- 条件渲染 -->
{{#if isActive}}
  <img src="star.gif" alt="Active">
{{else}}
  <img src="cry.gif" alt="Inactive">
{{/if}}

Handlebars.registerHelper('if', function(conditional, options) {
  if(conditional) {
    return options.fn(this);  
  } else {
    return options.inverse(this);  // 使用else里边的模板
  }
});


<!-- hash 参数 -->
{{#list nav id="nav-bar" class="top"}}
  <a href="{{url}}">{{title}}</a>
{{/list}}

Handlebars.registerHelper('list', function(context, options) {
  var attrs = Em.keys(options.hash).map(function(key) {
    return key + '="' + options.hash[key] + '"';
  }).join(" ");

  return "<ul " + attrs + ">" + context.map(function(item) {
    return "<li>" + options.fn(item) + "</li>";
  }).join("\n") + "</ul>";
});


<!-- block helper 支持向子模板里注入自己的私有变量，如@index -->
{{#list array}}
  {{@index}}. {{title}}
{{/list}}
Handlebars.registerHelper('list', function(context, options) {
  var out = "<ul>", data;

  if (options.data) {
    data = Handlebars.createFrame(options.data);
  }

  for (var i=0; i<context.length; i++) {
    if (data) {
      data.index = i;
    }

    out += "<li>" + options.fn(context[i], { data: data }) + "</li>";
  }

  out += "</ul>";
  return out;
});

// 访问父作用域中的index 可以使用 @../index

// Handlebars 3.0 部分内置helper 支持命名参数  user为当前this,userId为index
{{#each users as |user userId|}}
  Id: {{userId}} Name: {{user.name}}
{{/each}}

Handlebars.registerHelper('block-params', function() {
  var args = [],
      options = arguments[arguments.length - 1];
  for (var i = 0; i < arguments.length - 1; i++) {
    args.push(arguments[i]);
  }

  return options.fn(this, {data: options.data, blockParams: args});
});
{{#block-params 1 2 3 as |foo bar baz|}}
  {{foo}} {{bar}} {{baz}}
{{/block-params}}

<!-- 纯渲染，不做任何处理 -->
{{{{raw-helper}}}}
  {{bar}}
{{{{/raw-helper}}}}
```



#### 路径查找

可以通过{% raw %} `../name` 或者`{{./name}} or {{this/name}} or {{this.name}}`{% endraw %}


```html
{{permalink}}
{{#each comments}}
  {{../permalink}}

  {{#if title}}
    {{../permalink}}
  {{/if}}
{{/each}}
<!-- permalink 查找上级作用域中的Permalink属性 -->
```


#### 模板注释
使用{% raw %}`{{!-- --}} or {{! }}`{% endraw %}注释内容

```html
{{!-- This comment will not be in the output --}} 

{{! This comment will not be in the output }}
```


#### Helper
使用 `Handlebars.registerHelper`注册模板助手

```js
Handlebars.registerHelper('fullName', function(person) {
  return person.firstName + " " + person.lastName;
});

Handlebars.registerHelper('agree_button', function() {
  var emotion = Handlebars.escapeExpression(this.emotion),
      name = Handlebars.escapeExpression(this.name);

// 如果不需要转义内容，使用Handlebars.SafeString 
  return new Handlebars.SafeString(
    "<button>I agree. I " + emotion + " " + name + "</button>"
  );
});

Handlebars.registerHelper('if', function(conditional, options) {
  if(conditional) {
    return options.fn(this);  
  } else {
    return options.inverse(this);  // 使用else里边的模板
  }
});
```



#### 模板片段Partials 
使用 `Handlebars.registerPartial`注册模板片段

```js
Handlebars.registerPartial('myPartial', '{{name}}')
```

使用{% raw %}`{{> myPartial }}`{% endraw %}调用

```html
<div class="post">
  {{> userMessage tagName="h1" }}
  <h1>Comments</h1>
</div>

<!-- 动态模板片段 -->
{{> (whichPartial) }}

<!-- 更改上下文 -->
{{> myPartial myOtherContext }}

<!-- hash 参数，模板中可直接使用该参数-->
{{> myPartial parameter=value }}

<!-- Partial Blocks  如果partial 没有注册就直接调用，就会抛出异常，使用块级helper可解决 -->
{{#> myPartial }}
  Failover content    {{! myPartial 没有注册时才渲染 }}
{{/myPartial}}

<!-- 可以通过@partial-block拿到默认内容 -->
{{#> layout }}
  My Content
{{/layout}}

Site Content
{{> @partial-block }}

<!-- 模板内声明partial，当前block和子模板中可用 -->
{{#*inline "myPartial"}}
  My Content
{{/inline}}

{{#each children}}
  {{> myPartial}}
{{/each}}

```


#### 内置Helper

`if` 如果值为 `false, undefined, null, "", 0, or []` 就会渲染else部分

```html
<div class="entry">
  {{#if author}}
    <h1>{{firstName}} {{lastName}}</h1>
  {{else}}
    <h1>Unknown Author</h1>
  {{/if}}
</div>
```

`unless` 相当于`if`的else情况

```html
{{#unless license}}
  <h3 class="warning">WARNING: This entry does not have a license!</h3>
{{/unless}}
```

`each`遍历数组，会改变当前context，执行遍历的元素

```html
<ul class="people_list">
  {{#each people}}
    <li>{{this}}</li>
    
  {{else}}
    暂无数据
  {{/each}}
</ul>
```
>{{@key}} 遍历对象，相当于变量的key;{{@index}}遍历数组，为array的index
 
`with` 会改变当前context

```html

<div class="entry">
  <h1>{{title}}</h1>

  {{#with author}}
  <h2>By {{firstName}} {{lastName}}</h2>
  {{else}}
  <p class="empty">No content</p>
  {{/with}}
</div>
``` 

`log` 渲染时打印日志

```html
{{log "Look at me!"}}

<!-- level 支持的值为 debug, info, warn, and error -->
{{log "Log!" level="error"}}
```

处理helper异常
如果调用一个不存在的helper，就会异常，`options.name`会自动注入为helper名称

```js
Handlebars.registerHelper('helperMissing', function(/* [args, ] options */) {
  var options = arguments[arguments.length - 1];
  throw new Handlebars.Exception('Unknown field: ' + options.name);
});
```


#### API

```js
// 注册 partial
Handlebars.registerPartial('foo', partial);

Handlebars.registerPartial({
  foo: partial,
  bar: partial
});
// 注销 partial
Handlebars.unregisterPartial('foo');

// 注册 helper
Handlebars.registerHelper('foo', function() {
});

Handlebars.registerHelper({
  foo: function() {
  },
  bar: function() {
  }
});
// 注销helper
Handlebars.unregisterHelper('foo');

// 创建块级作用域 child data
Handlebars.createFrame(data)

// 顶级对象
@root 

// 数组第一个元素
@first

// 数组最后一个元素
@last

// 对象的key
@key

// 数组的index
@index

```
[API更多请参考](http://handlebarsjs.com/reference.html)

