---
title: css 选择器总结
date: 2019-11-20 12:03:01
tags: 前端
category: 前端
---
# 类选择器

没什么可说的 `.class`  修改样式.

选择器.key 选中所有class属性为 key的元素. 
选择器 p.key 选中所有class属性为key的<p> 元素。

# ID选择器

没什么可说的 `#id`  修改样式.

# 伪类选择器

注意是`伪类,伪类!,伪类!`, `p,div` 元素伪类,不能是.class:after 伪类.
这种`#data-table-1 test1:first-child{color: red;}`不生效样式.
加上元素 td`td.test1:first-child{color: red;}` 可以

* :link
* :visited
* :active
* :hover
* :focus
* :first-child
* :nth-child
* :nth-last-child
* :nth-of-type
* :first-of-type
* :last-of-type
* :empty
* :target
* :checked
* :enabled
* :disabled

# 关系的选择器

<table id="relselectors">
 <caption>常见的基于关系的选择器</caption>
 <tbody>
  <tr>
   <td style="width: 10em;"><strong>选择器</strong></td>
   <td><strong>选择的元素</strong></td>
  </tr>
  <tr>
   <td><code>A E</code></td>
   <td>元素A的任一后代元素E&nbsp;(后代节点指A的子节点，子节点的子节点，以此类推)</td>
  </tr>
  <tr>
   <td><code>A &gt; E</code></td>
   <td>元素A的任一子元素E(也就是直系后代)</td>
  </tr>
  <tr>
   <td><code>E:first-child</code></td>
   <td>任一是其父母结点的第一个子节点的元素E</td>
  </tr>
  <tr>
   <td><code>B + E</code></td>
   <td>元素B的任一下一个兄弟元素E</td>
  </tr>
  <tr>
   <td><code>B ~ E</code></td>
   <td>B元素后面的拥有共同父元素的兄弟元素E</td>
  </tr>
 </tbody>
</table>

# 实例

```html
<table id="data-table-1">
		<tr>
			<td>颜色红色并且加重字体</td>
			<td>0001</td>
			<td>default</td>
		</tr>
		<tr>
			<td>222</td>
			<td>0002</td>
			<td>default2</td>
		</tr>
		<tr>
			<td>222</td>
			<td>0002</td>
			<td>default2</td>
		</tr>
</table>


#data-table-1 td:first-child{
			color: cyan;
			font-weight: bold;
}
		/*  第一行的第二列 */
#data-table-1 td:first-child + td {color: blue;}


```