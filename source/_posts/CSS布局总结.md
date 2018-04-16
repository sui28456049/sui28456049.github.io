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



# 单列布局

## 头主尾等宽

```html

<style>
		.container {
			max-width: 960px; /*设置最大宽度为固定值*/
			margin: 0 auto;  /*设置内部子块元素左右居中*/
		}
		.header {
			height: 50px;  /*只需设置高度,宽度继承父元素*/
			background-color: #21FF80;
		}
		.main {
			height: 600px;  /*只需设置高度,宽度继承父元素*/
			background-color: #80007F;
		}
		.footer {
			height: 50px;  /*只需设置高度,宽度继承父元素*/
			background-color: #21FF06;
		}
	</style>

<h4>基本思路:</h4>
	<p>1.头部,主体,尾部全部放在一个容器内统一设置</p>
	<p>2.子块只须设置高度即可</p>
	<!-- DOM结构 -->
	<div class="container">
		<div class="header">头部</div>
		<div class="main">主体</div>
		<div class="footer">尾部</div>
	</div>
```

## 头尾自适应主体固宽

```html
<style>
		.container {
			max-width: 960px; /*设置最大宽度为固定值*/
			margin: 0 auto;  /*设置内部子块元素左右居中*/
		}
		.header {
			height: 50px;  /*只需设置高度,宽度继承父元素*/
			background-color: #21FF80;
		}
		.main {
			height: 600px;  /*只需设置高度,宽度继承父元素*/
			background-color: #80007F;
		}
		.footer {
			height: 50px;  /*只需设置高度,宽度继承父元素*/
			background-color: #21FF06;
		}
	</style>

<h4>基本思路:</h4>
	<p>1.头部,尾部单独放在一个容器内,仅设置height高度</p>
	<p>2.头部,尾部内容区仍与主体等宽</p>
	<p>3.主体单独放在一个容器内,并设置左右自动margin: auto</p>
	<h4>CSS样式无需修改,只需调整一下DOM结构即可</h4>
	<!-- DOM结构 -->
	
		<div class="header">
			<div class="container">头部</div>
		</div>
		<div class="main">
			<div class="container">主体</div>
		</div>
		<div class="footer">
			<div class="container">尾部</div>
		</div>
```

# 两列布局

## 左侧固定,右主体自适应

```html
<style>
	.left {
		width: 200px;
		height: 600px;
		background-color: skyblue;
		float: left;
	}
	.main {
		height: 600px;
		background-color: cyan; 
		margin-left: 200px;
	}
</style>


<h4>基本思路:</h4>
	<p>1.左侧固定宽度,并设置左浮动</p>
	<p>2.右侧主体仅设置margin-left,将区块向右拉过左侧宽度即可</p>
	<div class="left">左侧</div>
	<div class="main">主体</div>
```

##  右侧固定,左边主体自适应

```
<style>
	.right {
		width: 200px;
		height: 600px;
		background-color: skyblue;
		float: right;
	}
	.main {
		height: 600px;
		background-color: cyan; 
		margin-right: 200px;
	}
</style>

<h4>基本思路:</h4>
	<p>1.右侧固定宽度,并设置右浮动</p>
	<p>2.左侧主体仅设置margin-right,将区块向右拉过右侧宽度即可</p>
	<div class="right">右侧</div>
	<div class="main">主体</div>
```

## 左右二列宽度固定[浮动]

```html
<style>
	.container {
		width: 960px; 	
		margin: 0 auto;
		background-color: yellow;
		overflow: hidden; /*允许子元素撑开父级区块*/
	}
	.clear {
		-ms-zoom: 1;
	}
	.clear:after {
		content: '';
		display: block;
		clear:both;
	}
	.left {
		width: 200px;
		height: 600px;
		background-color: skyblue;
		float: left;  /*向左浮动*/
	}
	.right {
		width: 750px;
		height: 600px;
		background-color: cyan; 
		float:right;  /*向右浮动*/
	}
</style>

<h4>基本思路:</h4>
	<p>1.要给左右二列添加一个父级区块</p>
	<p>2.给左右二个区块设置浮动float:left/right</p>
	<p>3.给父区块添加after伪类让子块撑开父级</p>

	<div class="container">
		<div class="left">左侧</div>
		<div class="right">右侧</div>
	</div>
```

## 左右二列宽度固定[绝对定位]

```html
<style>
	.container {
		/*父区块必须设置绝对定位*/
		position: absolute;
		left:0;  /*左边起始定位点*/
		right:0; /*右边起始定位点*/
		margin: auto; /*左右边距自动挤压,即自动居中*/

		max-width: 960px; /*宽度一定要设置*/
	}
	.left {
		/*设置左边栏定位起始点*/
		position: absolute;
		top:0;
		left:0;

		width: 200px;
		height: 600px;
		background-color: skyblue;
	}
	.right {
		/*设置右边栏定位起始点*/
		position: absolute;
		top:0;
		right:0;

		width: 750px;
		height: 600px;
		background-color: cyan; 
	}
</style>

<h4>基本思路:</h4>
	<p>1.要给左右二列添加一个父级区块,并设置定位属性 <br>
	2.给左右二个区块分别使用绝对定位<br>
	3.父区块一定要设置宽度width</p>

	<div class="container">
		<div class="left">左侧</div>
		<div class="right">右侧</div>
	</div>
```

# 三列布局

## float+margin

```html
<style>
		.left {
			width: 200px; 
			height:600px;
			background-color: cyan;
			float:left;  /*左侧向左浮动对齐*/
		}
		.right {
			width: 200px; 
			height:600px;
			background-color: cyan;
			float:right; /*右侧向右浮动对齐*/
		}
		.main {
			height:600px;
			margin-left: 200px;  /*从左边挤出200px*/
			margin-right: 200px; /*从右边挤出200px*/
			background-color: #FD8008;
		}

	</style>

<h4>基本思路:<span style="color:red">float+margin</span></h4>
	<p>1.与二列布局极相似 <br>
	2.左右二列采用浮动+宽度布局<br>
	3.中间内容区使用margin挤区来<br>
	4.DOM顺序:<span style="color:red">必须先写二侧,再写主体,否则右侧会被挤下去</span><br>
	5.二列布局其实也是由它修改而来:去掉右侧.right,并去掉主体中的maring-right即可</p>

	<div class="left">左侧</div>
	<div class="right">右侧</div>
	<div class="main">内容</div>
```

## position+margin

```html
	<style>
		.container {  /*定位父级*/
			position: absolute;
			left:0;
			right:0;
		}
		.left {
			width: 200px; 
			height:600px;
			background-color: cyan;
			position: absolute;
			top:0;
			left:0;
		}
		.right {
			width: 200px; 
			height:600px;
			background-color: cyan;
			position: absolute;
			top:0;
			right:0;
		}
		.main {
			height:600px;
			margin-left: 200px;  /*从左边挤出200px*/
			margin-right: 200px; /*从右边挤出200px*/
			background-color: #FD8008;
		}

	</style>

<h4>基本思路:<span style="color:red">postion+margin</span></h4>
	<p>1.与二列定位布局极相似 <br>
	2.左右二列绝对定位+宽度布局<br>
	3.中间内容区使用margin挤区来<br>
	4.二列布局其实也是由它修改而来:去掉右侧.right,并去掉主体中的maring-right即可<br>
	5.推荐添加一个定位父级</p>


	<div class="container">
		<div class="left">左侧</div>
		<div class="right">右侧</div>
		<div class="main">内容</div>
	</div>
```





