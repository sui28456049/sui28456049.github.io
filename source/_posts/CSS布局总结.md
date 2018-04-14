---
title: CSS布局总结
date: 2018-03-17 15:32:55
tags: HTML&CSS
category: HTML&CSS
---

# 对齐大全

`父元素一定是块元素,根据子元素不同分为以下几种`

## 子元素是行内元素:如:(a,span)

​                a.水平居中:在父元素上设置: text-align:center;

​		b.垂直居中:在行内子元素上设置行高与父元素相同: line-height



```html
	<style>
	   .box1 {
			width: 200px;
			height: 200px;
			background-color: #FFFF0A;
			text-align: center;  /*可以使内部行内元素水平居中*/
		}
		.box1 a {
			line-height: 200px;  /*子元素设置行高与父元素高度相同*/
		}
	</style>

<div class="box1">
     	<a href="">百度....</a>
</div>
```

## 子元素是多行内联文本

​		a.水平居中: 父元素设置: text-align:center

​		b.垂直居中: 父元素设置:  display:table-cell;vertical-align:middle;



```html
<style>
		.box2 {
			width: 200px;
			height: 200px;
			background-color: #FC0107;
			text-align: center;  /*可以使内部多行行内元素水平居中*/
			/*以下二个声明可以使多行文本垂直居中*/
			display: table-cell;  /*设置显示方式为表格单元格*/
			vertical-align: middle; /*设置该单元格内的元素垂直居中*/
		}
</style>

<div class="box2">
		<span>新浪</span><br>
		<span>wwww.sui.com</span>
</div>
```

## 子元素是块元素

a. 水平居中: 子元素设置左右自动: margin: auto;

b. 垂直居中: 与多行内联文本处理方式一致: display:table-cell;vertical-align:middle

```html
<style>
		.box3 {
			width: 200px;
			height: 200px;
			background-color: #66CCFF;
			/*以下二个声明可以使块级子元素垂直居中*/
			display: table-cell;  /*设置显示方式为表格单元格*/
			vertical-align: middle; /*设置该单元格内的元素垂直居中*/			
		}
		.box3 .child {
			width: 100px;
			height: 100px;
			background-color: #F4FF0A;
			margin: auto;  /*水平居中*/
		}
</style>

<div class="box3">
		<div class="child"></div>
</div>
```

## 子元素是不定宽的块元素:最常见分页导航

a. 水平居中: 子元素转行内元素,父元素加text-align:center

b. 垂直居中: 分页的ul加行高line-height= paren.hight

c. 底边居中: 与多行内联文本垂直处理方式一致,vertical-align:bottom;

```html
	<style>
		.box4 {
			width: 200px;
			height: 200px;
			background-color: #FD6FCF;
			text-align: center;  /*可以使行内元素水平居中*/
			/*以下二个声明可以使块级子元素垂直居中*/
			display: table-cell;  /*设置显示方式为表格单元格*/
			vertical-align:bottom; /*设置该单元格内的元素底边居中*/	
		}
		.box4 ul {
			margin: 0;
			padding: 0;
			/*line-height: 200px;*/
		}
		.box4 li {
			list-style: none;
			display: inline;
		}
	</style>

	<div class="box4">
		<ul>
			<li><a href="">1</a></li>
			<li><a href="">2</a></li>
			<li><a href="">3</a></li>
			<li><a href="">4</a></li>
			<li><a href="">5</a></li>
		</ul>				
	</div>
```



