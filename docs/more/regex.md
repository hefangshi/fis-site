---
layout: post
title: 不完全正则指南- F.I.S
category: more
---

# 不完全正则指南

## 写在前面

本文只会描述FIS使用中的正则使用技巧，不会全面覆盖正则的使用技巧，建议希望完整的正则学习的同学可以参考[正则表达式30分钟入门教程](http://www.cnblogs.com/deerchao/archive/2006/08/24/zhengzhe30fengzhongjiaocheng.html)

## 什么是正则

> 在编写处理字符串的程序或网页时，经常会有查找符合某些复杂规则的字符串的需要。正则表达式就是用于描述这些规则的工具。换句话说，正则表达式就是记录文本规则的代码。

在FIS中，我们使用正则表达式主要用于在目录规范配置 `roadmap.path` 中处理路径匹配，只有将文件或目录路径匹配正确，我们才能为其设置目录规范。

在Javascript中，我们声明正则表达式有两种方式

```javascript
//直接书写表达式
var regex = /123/;

//通过字符串生成表达式
var regex = new RegExp('123');
```

我们一般使用第一种方式编写正则表达式，通过两个 `/` 号包裹的就是我们的正则表达式。 `/123/` 将会匹配所有包含 `123` 的文本。

## 测试正则

由于正则表达式较为复杂，因为一个好的调试工具是必须的，个人经常使用的有以下几个

* [debuggex](https://www.debuggex.com/)，图形化的正则表达式展示，适合用于解析复杂的正则表达式
* [regex101](http://regex101.com/#javascript)，详细的匹配内容展示，非常适合调试正则表达式
* [regexr](http://www.regexr.com/)，与regex101功能相似，可以按照个人喜好使用

也建议在下面的学习中多使用上面的工具自己尝试匹配一下

## 基本语法

### 元字符

在正则表达式中，除了一些特殊符号由于另作他用无法直接使用外，类似a-Z, 0-9等字符都是可以直接使用用于匹配的

```javascript
//匹配所有包含hello world的字符串
var regex = /hello world/;
```

当然如果我们全部都直接按照原字符编写的话就无法达到通配的效果了，因此正则表达式还提供了一些 **元字符** 用于处理通用匹配需求，此处列举一些常用的元字符

字符  | 说明
--- | -------------
.   | 匹配除换行符以外的任意字符
\w  | 匹配字母或数字或下划线或汉字 `[a-zA-Z0-9_]`
\s  | 匹配任意的空白符 `[\r\n\t\f]`
\d  | 匹配数字
\b  | 匹配单词的开始或结束
^  | 匹配字符串的开始
$  | 匹配字符串的结束

### 重复限定符

只有用于处理通配效果的元字符很明显是不够的，因此正则表达式还拥有一系列的限定符用于描述字符的重复情况

字符  | 说明
--- | -------------
 *	| 重复零次或更多次
 +	| 重复一次或更多次
 ?	| 重复零次或一次
{n}	| 重复n次
{n,}	| 重复n次或更多次
{n,m}	| 重复n到m次

通过元字符与重复限定符的结合使用，我们就可以匹配各种形式的字符串了

```javascript
//匹配除换行符外的任意字符任意次
var regex = /.*/;

//匹配[a-zA-Z0-9_]至少一次
regex = /\w+/;

//匹配数字字符2次
regex = /\d{2}/;

//匹配数字字符2-4次
regex = /\d{2,4}/;

//匹配所有js文件
regex = /.*\.js$/;
```

### 字符转义

如果你想查找元字符或限定符本身的话，比如你查找 `.` 或者 `\*` ，就出现了问题：你没办法指定它们，因为它们会被解释成别的意思。这时你就得使用 `\` 来取消这些字符的特殊意义。因此，你应该使用 `\.` 和 `\*` 。当然，要查找\本身，你也得用 `\\`。

这在路径匹配中十分常见，在**FIS中**，无论在windows还是unix下，我们均可以用 `\/` 来匹配目录分隔符

```javascript
//匹配 /modules/moduleA
var regex = /\/modules\/moduleA/;
```

### 分支条件

如果我们希望一部分的正则表达式可以按两种模式匹配，那么我们可以利用分支条件来指定多种规则，具体的方法是使用 `|` 分开不同的规则

```javascript
//匹配js文件
var regex = /.*\.js$/;

//匹配css文件
regex = /.*\.css$/;

//匹配js或css文件
regex = /.*\.js$|.*\.css$/;

//匹配js或css文件
regex = /.*\.(js|css)$/;
```

### 反义

有时候需要匹配除了某种字符以外的所有字符，这种时候我们就可以利用反义功能来定义字符

字符  | 说明
--- | -------------
\W  | 匹配任意不是字母，数字，下划线，汉字的字符
\S  | 匹配任意不是空白符的字符
\D  | 匹配任意非数字的字符
\B  | 匹配不是单词开头或结束的位置
[^x]  | 匹配除了x以外的任意字符
[^aeiou]  | 匹配除了aeiou这几个字母以外的任意字符

举一个FIS中经常用到的例子，如果我们只要匹配某个目录下一级目录的内容，那么很自然我们需要将目录分隔符 `\/` 排除在匹配内容之外，这时我们就可以使用 `[^\/]`来满足需求

``` javascript
//匹配 /modules 一级目录下的所有js
var regex = /\/modules\/[^\/]+\.js/;

// /moudles/a.js ok
// /moudles/a.css ok
// /moudles/moduleA/a.js fail
```
### 分组与捕获

分组功能可以将指定的正则表达式声明为子表达式，那么我们很多操作就可以针对子表达式进行，比如重复、分支

``` javascript
// 对子表达式进行重复设定，匹配abab
var regex = /(ab){2}/;

// 限定分支，公用其余匹配条件，仅在js和css处使用分支，匹配所有js与css文件
regex = /.*\/(js|css)/;
```

分组除了子表达式声明功能外，还可以将分组保存入捕获组，捕获组的概念可以理解为变量，子表达式匹配到的内容就会存放到这个变量中。

``` javascript
//匹配 /modules 目录下的所有js文件，同时捕获 /modules与.js之间的所有内容
var regex = /^\/modules\/(.*)\.js$/;

// /moudles/moudleA/a.js 分组内容为 moudleA/a
```

在FIS中，如果在 `reg` 设置了分组，那么在规范设定时， `release` 属性和 `id` 属性均可以通过变量进行引用

``` javascript
fis.config.set('roadmap.path', [
    {
        reg : /^\/modules\/(.*)\.js$/,
        release : '/static/$&',
        id : '$1'
    }
]);
```

其中 `$1` 就是对捕获组的引用，如果正则表达式中有多个分组，则会按照他们的出现顺序，按照$1, $2, $3的形式进行引用，而 `$&` 则表示的是整个正则表达式匹配的内容。

因此以 `/moudles/moudleA/a.js` 为例，在上述规则中，release的值会设置为 `/static/moudles/moudleA/a.js` ，id的值会设置为 `moduleA/a` 。

###  处理选项

Javascript中我们拥有3种处理选项

字符  | 说明
--- | -------------
i  | 忽略大小写
g  | 多次匹配，适用于希望正则表达式在字符串中进行多次匹配而不是匹配到第一个内容就返回
m  | 多行匹配模式，^与$将会在每行独立计算而不是整体计算

FIS中由于我们的正则表达式均是用来匹配路径，因此我们一般只会使用i模式用于忽略路径中的大小写，使用方法是在正则表达式的最后加上处理选项字符

``` javascript
fis.config.set('roadmap.path', [
    {
        reg : /^\/modules\/(.*)\.js$/i,
        release : '/static/$&',
        id : '$1'
    }
]);
```

这样我们的规则就可以同时处理/MOUDLES/MODULEA/a.js与/moudles/moduleA/a.js了。

## 最后

学好正则不仅仅会帮助你使用FIS，本文仅着重介绍了在FIS配置时会用到的一些正则知识，强烈建议大家在有时间的时候对正则进行一次全面的学习。