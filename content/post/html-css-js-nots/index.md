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