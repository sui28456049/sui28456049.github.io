---
title: Vue 基础实例
date: 2018-07-22 07:46:40
tags: Vue
category: Vue
---

## 基本操作

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>

<body>
	<div id="app">{{ message }}</div>
</body>
<script>	
	var app = new Vue({
	  el: '#app',
	  data: {
	    message: 'Hello Vue!'
	  }
	})
	setTimeout(function(){
		app.$data.message = 'suisuisuisuis';
	},2000);
</script>
</html>
```

## Todo List 实现

### Vue版

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>

<body>
	<div id="app">
		<input type="text" v-model="inputVal">
		<button v-on:click="handleBtn">提交</button>
		<ul>
			<li v-for="item in list">{{item}}</li>
		</ul>
	</div>
</body>
<script>
	var app = new Vue({
	  el: '#app',
	  data: {
	    list: ['随某人','随万三','小李飞刀'],
	    inputVal: ''
	  },
	  methods: {
	  		handleBtn: function() {
	  			this.list.push(this.inputVal);
	  			this.inputVal = '';
	  		}
	  }
	});
</script>
</html>
```

### jQuery版

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
</head>

<body>
	<div>
		<input type="text" id="input">
		<button id="btn">提交</button>
		<ul id="list"></ul>
	</div>
</body>
<script>
	function Page() {

	}

	$.extend(Page.prototype, {
		init: function() {
			this.bindEvents();
		},
		bindEvents: function() {	
			var btn = $('#btn');
			btn.on('click',$.proxy(this.handleBtnClick,this));
		},
		handleBtnClick: function() {
			var inputElm = $("#input");
			var inputValue = inputElm.val();
			var ulElem = $("#list");
			ulElem.append('<li>' + inputValue + '</li>');
			inputElm.val('');
		}
	});
	
	var page = new Page();
	page.init();
</script>
</html>
```

### 组件化todo list

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

</head>

<body>
	<div id="root">
			<input type="text" v-model="todoValue">
			<button @click="handleBtnClick">提交</button>
		<ul>
			<todo-item  v-bind:content="item" v-for="item in list">

			</todo-item>
		</ul>
	</div>

</body>
<script>
		Vue.component("TodoItem",{
			props: ['content'],
			template: "<li>{{content}}</li>",
		});
		var app = new Vue({
			el: "#root",
			data: {
				todoValue: "",
				list: ['sui']
			},
			methods: {
				handleBtnClick: function() {

					this.list.push(this.todoValue);
	  				this.todoValue = '';
				}
			}
		});
</script>
</html>
```

### 子组件向父组件传值

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

</head>

<body>
	<div id="root">
			<input type="text" v-model="todoValue">
			<button @click="handleBtnClick">提交</button>
		<ul>
			<todo-item  v-bind:content="item" v-for="(item, index) in list"
						v-bind:index="index"
						@delete="handleItemDelete"
						>

			</todo-item>
		</ul>
	</div>

</body>
<script>

	var TodoItem = {
		props: ['content','index'],
		template: "<li @click='handleItemClick'>{{content}}</li>",
		methods: {
			handleItemClick: function() {
				//向父组件传递一个删除事件
				this.$emit("delete", this.index);
			}
		}
	};
		
		var app = new Vue({
			el: "#root",
			components: {
				TodoItem: TodoItem
			},
			data: {
				todoValue: "",
				list: ['sui']
			},

			methods: {
				handleBtnClick: function() {

					this.list.push(this.todoValue);
	  				this.todoValue = '';
				},
				handleItemDelete: function(index) {
					this.list.splice(index, 1);
				}
			}
		});
</script>
</html>
```

v-bind:content 可以简写为:content

## 生命周期函数

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

</head>

<body>
<!--这个div区域就是MVVM中的 View-->
    <div id="app">
    {{msg}}
    </div>
</body>
<script>
		var vm =  new Vue({
					  	  el: '#app',
					  	  data: {
						            msg: 'hello vuejs'
						        },
						  	 // 注意,所有的生命周期函数没在methods 里面
						  
						    // 这是第1个生命周期函数，表示实例完全被创建出来之前，会执行它
					        beforeCreate: function () {
					            console.log('01 beforeCreate', this.msg);
					            //注意：在 beforeCreate 生命周期函数执行的时候，data 和 methods 中的 数据都还没有没初始化
					        },

					         // 这是第2个生命周期函数
					        created: function () {
					            console.log('02 created', this.msg);
					            //注意：如果要调用 methods 中的方法，或者操作 data 中的数据，最早，只能在 created 中操作
					        },

						    // 这是第3个生命周期函数，表示 模板已经在内存中编辑完成了，但是尚未把模板渲染到页面中
					        beforeMount: function () {
					            console.log('03 beforeMount', this.msg);
					            // 在 beforeMount 执行的时候，页面中的元素，还没有被真正替换过来，只是之前写的一些模板字符串
					        },
					        // 这是第4个生命周期函数，表示，内存中的模板，已经真实的挂载到了页面中，用户已经可以看到渲染好的页面了
					        mounted: function () {
					            console.log('04 mounted', this.msg);
					            // 注意： mounted 是 实例创建期间的最后一个生命周期函数，当执行完 mounted 就表示，实例已经被完全创建好了
					            // 此时，如果没有其它操作的话，这个实例，就静静的 躺在我们的内存中，一动不动
					        },

					        beforeUpdate: function () {
					        	console.log('beforeUpdate',this.msg);
					        },

					        updated: function () {
					        	 console.log('updated', this.msg);
					        }


					  });

</script>
</html>
```

## 计算器与侦听器


## Class 与 Style 绑定

### 对象语法

```html

<style>
	.active {
		color: red;
	}
</style>


<body>

<div id="app" @click="btnClick"
			  :class="{ active: isActive}">
	{{name}}
</div>

</body>

<script>
var vm = new Vue({
		  el: '#app',
		  data: {
		  	name: '流水落花春去也',
		    isActive: false
		  },
		  methods: {
		  	btnClick: function() {
		  		this.isActive = !this.isActive
		  	}
		  }
})
</script>
```

### 数组语法

```html
<style>
	.activated {
		color: red;
	}
</style>

<div id="app" @click="btnClick"
			  :class="[activated]">
	{{name}}
</div>

<script>
	
	var vm = new Vue({
		  el: '#app',
		  data: {
		  	  name: '小桥流水人家',
			  activated: ""
		  },
		  methods: {
		  	btnClick: function() {
		  		this.activated = this.activated === "activated" ? '' :"activated";
		  	}
		  }
})

</script>
```

### 内联写法

```html

<div id="app" @click="btnClick"
			  :style="styleObj">
	{{name}}
</div>

<script>
	
	var vm = new Vue({
		  el: '#app',
		  data: {
		  	 name: '小李飞刀',
		  	 styleObj:{
		  	 	color: ""
		  	 }
		  },
		  methods: {
		  	btnClick: function() {
		  		this.styleObj.color = this.styleObj.color === "red" ? 'black' :"red";
		  	}
		  }
})
</script>
```

## 条件渲染

```html
<body>

<div id="app">
	<!-- v-if 和v-else 连着写 -->
	<div  v-if="word==='a'">这个单词是a</div>
	<div  v-else-if="word==='b'">这个单词是b</div>
	<div  v-else="word==='c'">这个单词是c</div>

    <!-- v-show 和 v-if -->
	<div  v-if="show">{{name}}</div>
	<div  v-show="show">{{name}}</div>
</div>

</body>
<script>	
	var vm = new Vue({
		  el: '#app',
		  data: {
		  	 word: "a",
		  	 name: '小李飞刀',
		  	 show: false,
		  },
		  methods: {
		  	
		  }
})
</script>
```







