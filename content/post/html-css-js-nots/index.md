---
description: ""
title: "前端学习笔记"
draft: true
date: "2026-07-05T11:59:38+08:00"
slug: "html-css-js-nots"
categories:
 - notes
tags:
 - html
 - css
 - js
image: ""
---

## Html5笔记

### 一、基础框架
1. 文件以<DOCTYPE html开头>
基本结构为
```
<!DOCTYPE html> 
<html> 
<head> 
<meta charset="utf-8" /> 
<title></title> 
</head> 
<body> 
</body> 
</html> 
```
2. 标签是最基本单位，由于<>组成，标签分为但标签和双标签，即，是否有结束标记，<title></title>
<hr/>单标记
3. 常用标签有
<html>定义html文档
<head>定义文档头
<a>链接标签
<div>html层
<form>表单
<body>文档体
<tittle>标题
<img>图片
<table>表格
<td>表格列
<tr>表格行
<input>输入
3. 标签属性
属性是标签一部分，用于包含额外信心，属性和属性值成对出现
```
<img src=“../image/a.png” width=“100” height=“100”/> 
<!-- 结构是 属性名=”属性值” -->
```
4. <head>用于标识网页元数据，描述网页基本信息，主要包括<title><meta>
```
<meta charset="UTF-8"> 
```
用于设置字符编码

name属性，将信息写给搜索引擎看，常用属性值有，author，keywords，description，
```<meta name="viewport" ...>```
用于适配手机，否则手机看到字很小

5. <link>标签 加载一个图片作为网页图标
re属性：声明被链接文件和当前文件关系，这里选icon
type属性：声明类型
href属性：标识图片地址
```
<link rel="icon" type="image/x-icon" href="img/icon.jpg" />
```
### 二、常用块标签
1. 
<h1></h1>...<h6></h6>标题
<hr/>水平线
<p></p>段落标签
<br/>换行
<pre></pre>预览格式标签，按原格式输出
<ul><li></li></ul>无序列表标签
<ol><li></li></ol>有序列表标签 
<div></div>分区标签 ，将文档分割为独立的、不同的部分

## 三、表单
表单用于用户填写数据源并交给服务器出来，像qq登录页就是表单。

**表单由<form></form>包裹
1. 常见属性有
action用于指定给谁提交数据，即发送服务器的地址
method用于像服务器提交数据的方法，get或post

2.input输入框属性
type 默认是text
name 标识输入框名称，一般必填，传递数据时使用```name=value```的形式传递
value 输入框默认值
placeholder 提示信息，当有值时就会消失，浅灰色的提示文字
checked="cheked" 默认选中

**其他type**
-  text 文本框，默认宽度为20个字符
- password 密码输入框
- radio 单选按钮
   ```
<form action="form.php" method="post"> 
<input type="radio" name="sex" value="男" checked="checked" />男 
<input type="radio" name="sex" value="女" />女 
</form>
```
- checkbox复选框

-  file 上传文件按钮
- submit提交按钮，当没有value值时，submit 按钮中默认显示的文字为“提交”
- reset 重置表单为初始状态
- button 单机按钮
    ```
      <form action="form.php" method="post"> 
      <input type="button" value="点我！"  onclick="alert('这是一个按钮！')" /> 
      </form>
     ```
3. select 下拉选择
```
<form action="form.php" method="post"> 
 <select name="week"> 
  <option value="1">1</option> 
  <option value="2">2</option> 
  <option value="3">3</option> 
  <option value="4">4</option> 
  <option value="5">5</option> 
  <option value="6">6</option> 
  <option value="7">7</option> 
 </select> 
</form> 
```
4. textarea 文本域
readonly="readonly" 只读

5. button按钮


### 二、CSS基础
css由选择器及一条或多条声明，基本语法如下：
```
选择器{ 
属性:属性值;     
[属性:属性值; …] 
} 
```
CSS添加注释的方式为/*……*/

行内样式表，即将css代码写在HTML代码内部，作为标签的属性
```
<a href="#" style="color:red;font-size:10px;">日用百货</a>
```
内嵌式则是将css代码内嵌到HTML代码中，通常放在head标签内部
```
<head> 
<!--charset="UTF-8"表示当前文档采用字符集中utf-8,支持中文--> 
<meta charset="UTF-8"> 
<title>内部样式表</title> 
<!--内部样式表 代码要放置在style标签内--> 
<style type="text/css"> 
div{ 
color:red; 
} 
</style> 
</head>
```

外联式也是最推荐的，方便维护，各部分代码分离
```
<head> 
<meta charset="UTF-8"> 
<title>外部样式表</title> 
<link rel="stylesheet" type="text/css" href="css/ch05.css" />  
</head>
```
### 三、Javascript




















