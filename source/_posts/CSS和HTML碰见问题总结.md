---
title: CSS和HTML碰见问题总结
date: 2019-03-17 18:44:25
tags: 前端
category: 前端
---

# 气泡/对话框样式

https://www.cnblogs.com/dreamflower/p/5371246.html

# 文字流光渐变特效

示例

```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>Title</title>
		<style>
			.masked h4 {
				display: block;
				width: 600px;
				height: 100px;
				/*渐变背景*/
				background-image: -webkit-linear-gradient(left, #3498db, #f47920 10%, #d71345 20%, #f7acbc 30%,
					#ffd400 40%, #3498db 50%, #f47920 60%, #d71345 70%, #f7acbc 80%, #ffd400 90%, #3498db);
				color: transparent;
				/*文字填充色为透明*/
				text-fill-color: transparent;
				-webkit-background-clip: text;
				/*背景剪裁为文字，只将文字显示为背景*/
				background-size: 200% 100%;
				/*背景图片向水平方向扩大一倍，这样background-position才有移动与变化的空间*/
				/* 动画 */
				animation: masked-animation 4s infinite linear;
			}

			@keyframes masked-animation {
				0% {
					background-position: 0 0;
					/*background-position 属性设置背景图像的起始位置。*/
				}

				100% {
					background-position: -100% 0;
				}
			}
		</style>
	</head>
	<body>

		<div class="masked">
			<h4>人生如逆旅,我亦是行人.采菊东篱下,悠然见南山.</h4>
		</div>

	</body>
</html>

```

1、linear-gradient属性给文字加一个线性渐变的背景色

2、transparent 将文字设置为透明(文字变为透明,依然占位)

3、background-clip将背景裁剪为文字,屏幕就变成这样子了。

4、animation设置动画，我们这里通过背景色position的变化来达成这个效果，所以要先将background-size放大，这样子background-position才有变化的空间。

# flex:1 什么意思

flex属性是 flex盒模型的子元素属性

flex 属性用于设置或检索弹性盒模型对象的`子元素`如何`分配空间`。

flex 属性是 flex-grow、flex-shrink 和 flex-basis 属性的简写属性。


默认值：	0 1 auto


让所有弹性盒模型对象的子元素都有相同的长度，忽略它们内部的内容：

```
#main
{
	width:220px;
	height:300px;
	border:1px solid black;
	display:flex;
}

#main div
{
	flex:1;
}


<div id="main">
  <div style="background-color:coral;">红色</div>
  <div style="background-color:lightblue;">蓝色</div>  
  <div style="background-color:lightgreen;">带有更多内容的绿色 div</div>
</div>
```

flex:1 子元素项目一样大.如果有一个子元素设置flex:1.5,这个元素相对于其他元素1.5倍大.

