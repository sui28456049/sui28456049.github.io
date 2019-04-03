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

# vw／vh

vw = view width
vh = view height

这两个单位是CSS3引入的，以上称为视口单位允许我们更接近浏览器窗口定义大小。

在桌面端，视口指的是在桌面端，指的是浏览器的可视区域；
在移动端，它涉及3个视口：Layout Viewport（布局视口），Visual Viewport（视觉视口），Ideal Viewport（理想视口）。

视口单位中的“视口”，桌面端指的是浏览器的可视区域；移动端指的就是Viewport中的Layout Viewport。


单位	解释
vw	1vw = 视口宽度的1%
vh	1vh = 视口高度的1%
vmin	选取vw和vh中最小的那个
vmax	选取vw和vh中最大的那个

`
比如：浏览器视口尺寸为370px,那么 1vw = 370px * 1% = 6.5px(浏览器会四舍五入向下取7)
`
## 运用

根据设计稿的尺寸转换为vw单位(SASS函数编译)

```
//iPhone 6尺寸作为设计稿基准
$vm_base: 375; 
@function vm($px) {
    @return ($px / 375) * 100vw;
}
```

## 最优做法——搭配vw和rem

使用vm作为css单位代码量确实减少很多，但是你会发现它是利用视口单位实现，依赖于视口大小而自动缩放，失去了最大最小宽度的限制。

所以，我们需要结合rem单位来实现布局，而rem正好可以动态改变根元素大小，做法是：

给根元素大小设置vw–动态改变大小。
限制根元素font-size的最大最小值，配合bosy加上最大最小宽度。

```
// rem 单位换算：定为 75px 只是方便运算，750px-75px、640-64px、1080px-108px，如此类推
$vm_fontsize: 75; // iPhone 6尺寸的根元素大小基准值
@function rem($px) {
     @return ($px / $vm_fontsize ) * 1rem;
}
// 根元素大小使用 vw 单位
$vm_design: 750;
html {
    font-size: ($vm_fontsize / ($vm_design / 2)) * 100vw; 
    // 同时，通过Media Queries 限制根元素最大最小值
    @media screen and (max-width: 320px) {
        font-size: 64px;
    }
    @media screen and (min-width: 540px) {
        font-size: 108px;
    }
}
// body 也增加最大最小宽度限制，避免默认100%宽度的 block 元素跟随 body 而过大过小
body {
    max-width: 540px;
    min-width: 320px;
}
```







