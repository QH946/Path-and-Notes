# 第三章 Vue&书城项目第一阶段

##  学习目标

* 了解什么是框架
* 了解什么是Vue
* 掌握Vue的基本语法
* 了解Vue的生命周期
* 实现登录表单校验
* 实现注册表单校验

### 1. Vue

#### 1.1 什么是框架

任何编程语言在最初的时候都是没有框架的，后来随着在实际开发过程中不断总结<span style="color:blue;font-weight:bold;">『经验』</span>，积累<span style="color:blue;font-weight:bold;">『最佳实践』</span>，慢慢的人们发现很多<span style="color:blue;font-weight:bold;">『特定场景』</span>下的<span style="color:blue;font-weight:bold;">『特定问题』</span>总是可以<span style="color:blue;font-weight:bold;">『套用固定解决方案』</span>。

于是有人把成熟的<span style="color:blue;font-weight:bold;">『固定解决方案』</span>收集起来，整合在一起，就成了<span style="color:blue;font-weight:bold;">『框架』</span>。

在使用框架的过程中，我们往往只需要告诉框架<span style="color:blue;font-weight:bold;">『做什么（声明）』</span>，而不需要关心框架<span style="color:blue;font-weight:bold;">『怎么做（编程）』</span>。

对于Java程序来说，我们使用框架就是导入那些封装了<span style="color:blue;font-weight:bold;">『固定解决方案』</span>的jar包，然后通过<span style="color:blue;font-weight:bold;">『配置文件』</span>告诉框架做什么，就能够大大简化编码，提高开发效率。我们使用过的junit其实就是一款单元测试框架。

而对于JavaScript程序来说，我们使用框架就是导入那些封装了<span style="color:blue;font-weight:bold;">『固定解决方案』</span>的<span style="color:blue;font-weight:bold;">『js文件』</span>，然后在框架的基础上编码。

> 用洗衣服来类比框架：
>
> 典型应用场景：洗衣服
>
> 输入数据：衣服、洗衣液、水
>
> 不使用框架：手洗
>
> 使用框架：使用洗衣机，对人来说，只需要按键，具体操作是洗衣机完成的。人只是告诉洗衣机做什么，具体的操作是洗衣机完成的。

实际开发中使用框架时，我们也主要是告诉框架要做什么，具体操作是框架完成的。

#### 1.2 Vue的简介

##### 1.2.1 Vue的作者介绍

在为AngularJS工作之后，Vue的作者<span style="color:blue;font-weight:bold;">尤雨溪</span>开Vue.js。他声称自己的思路是提取Angular中自己喜欢的部分，构建出一款相当轻量的框架。

Vue最早发布于2014年2月。作者在Hacker News、Echo JS与 Reddit的JavaScript版块发布了最早的版本。一天之内，Vue 就登上了这三个网站的首页。

Vue是Github上最受欢迎的开源项目之一。同时，在JavaScript框架/函数库中， Vue所获得的星标数已超过React，并高于Backbone.js、Angular 2、jQuery等项目。

##### 1.2.2 Vue的官网介绍

Vue (读音 /vjuː/，类似于**view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

**官网地址:**https://v3.cn.vuejs.org/

#### 1.4 HelloWorld

1. 创建工程,导入vue.js文件放入工程的js文件夹中

   ① Vue框架的js文件获取

   ​	官网提供的下载地址：https://cdn.jsdelivr.net/npm/vue/dist/vue.js

   ② 创建空vue.js文件,将官网提供的vue.js文件的内容复制粘贴到本地vue.js文件中

2. 创建demo01.html(引入vuejs,定义div,创建vue实例)

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Vue的入门</title>
        <script src="../js/vue.js"></script>
    </head>
    <body>
        <div id="app">
            <div>{{message}}</div>
        </div>
        <!--
使用vue获取div中的内容，以及设置div中的内容
使用vue的具体步骤:
1. 指定一个区域，在该区域中使用vue
2. 在该区域下方编写一个script标签，在script标签中创建Vue对象
3. 以json格式给Vue对象传入需要传入的参数
1. "el" 表示哪块区域可以使用vue
2. "data" 表示你需要定义的数据模型
4. 将message这个数据模型绑定给div的标签体: 使用插值表达式进行绑定
-->
        <script>
            var vue = new Vue({
                "el":"#app",
                "data":{
                    "message":"hello vue" //定义了一个数据模型，该数据模型的名字叫"message"
                }
            });
        </script>
    </body>
</html>
```

#### 1.5 声明式渲染

##### 1.5.1 概念

###### 1.5.1.1 声明式

<span style="color:blue;font-weight:bold;">『声明式』</span>是相对于<span style="color:blue;font-weight:bold;">『编程式』</span>而言的。

- 声明式：告诉框架做什么，具体操作由框架完成
- 编程式：自己编写代码完成具体操作

###### 1.5.1.2 渲染

![images](images/img003.png)

上图含义解释：

- 蓝色方框：HTML标签
- 红色圆形：动态、尚未确定的数据
- 蓝色圆形：经过程序运算以后，计算得到的具体的，可以直接在页面上显示的数据、
- 渲染：程序计算动态数据得到具体数据的过程

##### 1.5.2 案例讲解

**HTML代码**

```html
		<!-- 使用{{}}格式，指定要被渲染的数据 -->
		<div id="app">{{message}}</div>
```

**vue代码**

```javascript
// 1.创建一个JSON对象，作为new Vue时要使用的参数
var argumentJson = {
	
	// el用于指定Vue对象要关联的HTML元素。el就是element的缩写
	// 通过id属性值指定HTML元素时，语法格式是：#id
	"el":"#app",
	
	// data属性设置了Vue对象中保存的数据
	"data":{
		"message":"Hello Vue!"
	}
};

// 2.创建Vue对象，传入上面准备好的参数
var app = new Vue(argumentJson);
```

<img src="images/img004.png" alt="images" style="zoom:50%;" />

通过验证Vue对象的『响应式』效果，我们看到Vue对象和页面上的HTML标签确实是始终保持着关联的关系，同时看到Vue在背后确实是做了大量的工作。

#### 1.6 绑定元素属性

##### 1.6.1 基本语法

v-bind:HTML标签的原始属性名

##### 1.6.2 案例一代码

**HTML代码**

```html
<div id="app">
	<!-- v-bind:value表示将value属性交给Vue来进行管理，也就是绑定到Vue对象 -->
	<!-- vueValue是一个用来渲染属性值的表达式，相当于标签体中加{{}}的表达式 -->
	<input type="text" v-bind:value="vueValue" />
	
	<!-- 同样的表达式，在标签体内通过{{}}告诉Vue这里需要渲染； -->
	<!-- 在HTML标签的属性中，通过v-bind:属性名="表达式"的方式告诉Vue这里要渲染 -->
	<p>{{vueValue}}</p>
</div>

```

**Vue代码**

```javascript
// 创建Vue对象，挂载#app这个div标签
var app = new Vue({
	"el":"#app",
	"data":{
		"vueValue":"太阳当空照"
	}
});
```

##### 1.6.3 案例二代码

**HTML代码**

```html
<div id="app">
        <p v-bind:style="fontCss">Vue and CSS</p>
    </div>

```

**Vue代码**

```javascript
new Vue({
        "el":"#app",
        "data":{
            "fontCss":{
                "color":"red",
                "font-size":"30px",
            },
        },
    });
```

**扩展:**

v-bind:属性名="属性值"可以简写成 :属性名="属性值"

#### 1.7 双向数据绑定

##### 1.7.1 提出问题

<img src="images/img006.png" alt="images" style="zoom:50%;" />

而使用了双向绑定后，就可以实现：页面上数据被修改后，Vue对象中的数据属性也跟着被修改。

##### 1.7.2 案例代码

**HTML代码**

```html
<div id="app">
	<!-- v-bind:属性名 效果是从Vue对象渲染到页面 -->
	<!-- v-model:属性名 效果不仅是从Vue对象渲染到页面，而且能够在页面上数据修改后反向修改Vue对象中的数据属性 -->
	<input type="text" v-model:value="vueValue" />
	
	<p>{{vueValue}}</p>
</div>
```

**Vue代码**

```javascript
// 创建Vue对象，挂载#app这个div标签
var app = new Vue({
	"el":"#app",
	"data":{
		"vueValue":"太阳当空照"
	}
});
```

**页面效果**

p标签内的数据能够和文本框中的数据实现同步修改：

![images](images/img007.png)

**扩展:**

1. v-model:value="值" 可以简写成 v-model="值"

2. trim修饰符

实际开发中，要考虑到用户在输入数据时，有可能会包含前后空格。而这些前后的空格对我们程序运行来说都是干扰因素，要去掉。在v-model后面加上.trim修饰符即可实现。

```html
<input type="text" v-model.trim="vueValue" />
```

Vue会帮助我们在文本框失去焦点时自动去除前后空格。

#### 1.8 条件渲染

根据Vue对象中，数据属性的值来判断是否对HTML页面内容进行渲染。

##### 1.8.1 v-if

**HTML代码**

```html
<div id="app">
	<h3>if</h3>
	<img v-if="flag" src="/pro03-vue/images/one.jpg" />
	<img v-if="!flag" src="/pro03-vue/images/two.jpg" />
</div>
```

**Vue代码**

```javascript
var app = new Vue({
    "el":"#app",
    "data":{
        "flag":true
    }
});
```

##### 1.8.2 v-if和v-else

**HTML代码**

```html
<div id="app02">
	<h3>if/else</h3>
	<img v-if="flag" src="/pro03-vue/images/one.jpg" />
	<img v-else="flag" src="/pro03-vue/images/two.jpg" />
</div>
```

**Vue代码**

```javascript
var app02 = new Vue({
    "el":"#app02",
    "data":{
        "flag":true
    }
});
```

##### 1.8.3 v-show

**HTML代码**

```html
<div id="app03">
	<h3>v-show</h3>
	<img v-show="flag" src="/pro03-vue/images/mi.jpg" />
</div>
```

**Vue代码**

```javascript
var app03 = new Vue({
    "el":"#app03",
    "data":{
        "flag":true
    }
});
```

#### 1.9 列表渲染

##### 1.9.1 迭代一个简单的数组

**HTML代码**

```html
<div id="app01">
	<ul>
		<!-- 使用v-for语法遍历数组 -->
		<!-- v-for的值是语法格式是：引用数组元素的变量名 in Vue对象中的数组属性名 -->
		<!-- 在文本标签体中使用{{引用数组元素的变量名}}渲染每一个数组元素 -->
		<li v-for="fruit in fruitList">{{fruit}}</li>
	</ul>
</div>
```

**Vue代码**

```javascript
var app01 = new Vue({
	"el":"#app01",
	"data":{
		"fruitList": [
			"apple",
			"banana",
			"orange",
			"grape",
			"dragonfruit"
		]
	}
});
```

##### 1.9.2 迭代一个对象数组

**HTML代码**

```html
<div id="app">
	<table>
		<tr>
			<th>编号</th>
			<th>姓名</th>
			<th>年龄</th>
			<th>专业</th>
		</tr>
		<tr v-for="(employee,index) in employeeList">
			<td>{{employee.empId}}</td>
			<td>{{employee.empName}}</td>
			<td>{{employee.empAge}}</td>
			<td>{{employee.empSubject}}</td>
		</tr>
	</table>
</div>
```

**Vue代码**

```javascript
var app = new Vue({
	"el":"#app",
	"data":{
		"employeeList":[
			{
				"empId":11,
				"empName":"tom",
				"empAge":111,
				"empSubject":"java"
			},
			{
				"empId":22,
				"empName":"jerry",
				"empAge":222,
				"empSubject":"php"
			},
			{
				"empId":33,
				"empName":"bob",
				"empAge":333,
				"empSubject":"python"
			}
		]
	}
});
```

#### 1.10 事件驱动

##### 1.10.1 案例一: 点击事件

**HTML代码**

```html
<div id="app">
     <input type="button" value="clickTest" v-on:click="clickTest"/>
</div>
```

**Vue代码**

```javascript
new Vue({
    "el":"#app",
    "methods":{
        "clickTest":function (ev) {
            console.log("你点我了");
        }
    }
});
```

##### 1.10.2 案例二:change事件

**HTML代码**

```html
<div id="app">
        <input type="text" v-model="username" v-on:change="changeTest"/>
    </div>
```

**Vue代码**

```javascript
new Vue({
    "el":"#app",
    "data":{
        "username":"",
    },
    "methods":{
        "changeTest":function (ev) {
            console.log("改变后的值："+this.username);
        }
    }
});
```

**扩展:**

v-on:事件名="函数"可以简写成@事件名="函数"

##### 1.10.3 取消控件的默认行为

###### ① 控件默认行为

- 点超链接会跳转页面
- 点表单提交按钮会提交表单

本来控件的默认行为是天经地义就该如此的，但是如果我们希望点击之后根据我们判断的结果再看是否要跳转，此时默认行为无脑跳转的做法就不符合我们的预期了。

###### ② 取消方式

调用**事件对象**的**preventDefault()**方法。

**超链接举例**

HTML代码：

```javascript
<a id="anchor" href="http://www.baidu.com">超链接</a>
```

JavaScript代码：

```javascript
document.getElementById("anchor").onclick = function(ev) {
    console.log("我点击了一个超链接");
    ev.preventDefault();
}
```

**表单提交按钮举例**

HTML代码：

```html
<form action="http://www.baidu.com" method="post">
    <button id="submitBtn" type="submit">提交表单</button>
</form>
```

JavaScript代码：

```javascript
document.getElementById("submitBtn").onclick = function(ev) {
    console.log("我点击了一个表单提交按钮");
    ev.preventDefault();
}
```

#####  1.10.4 阻止事件冒泡

<img src="images/img014.png" alt="images" style="zoom:50%;" />

图中的两个div，他们的HTML标签是：

```html
<div id="outterDiv">
	<div id="innerDiv"></div>
</div>
```

点击里面的div同时也等于点击了外层的div，此时如果两个div上都绑定了单击响应函数那么就都会被触发：

```javascript
document.getElementById("outterDiv").onclick = function() {
	console.log("外层div的事件触发了");
}

document.getElementById("innerDiv").onclick = function() {
	console.log("内层div的事件触发了");
}

```

所以事件冒泡就是一个事件会不断向父元素传递，直到window对象。

如果这不是我们想要的效果那么可以使用<span style="color:blue;font-weight:bold;">事件对象</span>的<span style="color:blue;font-weight:bold;">stopPropagation()</span>函数阻止。

```javascript
document.getElementById("innerDiv").onclick = function(ev) {
	console.log("内层div的事件触发了");
	
	ev.stopPropagation();
}
```

##### 1.10.5 Vue事件修饰符

对于事件修饰符，Vue官网的描述是：

> 在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：<span style="color:blue;font-weight:bold;">方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节</span>。

###### ① 取消控件的默认行为

控件的默认行为指的是：

- 点击超链接跳转页面
- 点击表单提交按钮提交表单

实现这个需求使用的Vue事件修饰符是：<span style="color:blue;font-weight:bold;">.prevent</span>

```html
<a href="http://www.baidu.com" @click.prevent="clickAnchor">超链接</a>

<form action="http://www.baidu.com" method="post">
	<button type="submit" @click.prevent="clickSubmitBtn">提交表单</button>
</form>
```

###### ② 取消事件冒泡

实现这个需求使用的Vue事件修饰符是：<span style="color:blue;font-weight:bold;">.stop</span>

```html
<div id="outterDiv" @click="clickOutterDiv">
	<div id="innerDiv" @click.stop="clickInnerDiv"></div>
</div>
```

#### 1.11 侦听属性

##### 1.11.1 提出需求

```html
<div id="app">
	<p>尊姓：{{firstName}}</p>
	<p>大名：{{lastName}}</p>
	尊姓：<input type="text" v-model="firstName" /><br/>
	大名：<input type="text" v-model="lastName" /><br/>
	<p>全名：{{fullName}}</p>
</div>
```

在上面代码的基础上，我们希望firstName或lastName属性发生变化时，修改fullName属性。此时需要对firstName或lastName属性进行<span style="color:blue;font-weight:bold;">『侦听』</span>。

具体来说，所谓<span style="color:blue;font-weight:bold;">『侦听』</span>就是对message属性进行监控，当firstName或lastName属性的值发生变化时，调用我们准备好的函数。

##### 1.11.2 Vue代码

在watch属性中声明对firstName和lastName属性进行<span style="color:blue;font-weight:bold;">『侦听』</span>的函数：

```javascript
var app = new Vue({
	"el":"#app",
	"data":{
		"firstName":"jim",
		"lastName":"green",
		"fullName":"jim green"
	},
	"watch":{
		"firstName":function(inputValue){
			this.fullName = inputValue + " " + this.lastName;
		},
		"lastName":function(inputValue){
			this.fullName = this.firstName + " " + inputValue;
		}
	}
});
```

#### 1.12 Vue的生命周期

##### 1.12.1 概念

在我们各种语言的编程领域中，<span style="color:blue;font-weight:bold;">『生命周期』</span>都是一个非常常见的概念。一个对象从创建、初始化、工作再到释放、清理和销毁，会经历很多环节的演变。比如我们在JavaSE阶段学习过线程的生命周期，今天学习Vue对象的生命周期，将来还要学习Servlet、Filter等Web组件的生命周期。

##### 1.12.2 Vue对象的生命周期

<img src="images/img008.png" alt="images" style="zoom:50%;" />

##### 1.12.3 生命周期钩子函数

Vue允许我们在特定的生命周期环节中通过钩子函数来加入我们的代码。

```html
<div id="app">
	<p id="content">{{message}}</p>
	<button @click="changeValue">点我</button>
</div>
```

```javascript
new Vue({
	"el":"#app",
	"data":{
		"message":"hello"
	},
	"methods":{
		"changeValue":function(){
			this.message = "new hello";
		}
	},
	
	// 1.实例创建之前
	"beforeCreate":function(){
		console.log("beforeCreate:"+this.message);
	},
	
	// 2.实例创建完成
	"created":function(){
		console.log("created:"+this.message);
	},
	
	// 3.数据挂载前
	"beforeMount":function(){
		console.log("beforeMount:"+document.getElementById("content").innerText);
	},
	
	// 4.数据已经挂载
	"mounted":function(){
		console.log("mounted:"+document.getElementById("content").innerText);
	},
	
	
});
```

## 2. 书城项目第一阶段

#### 2.1 准备工作

创建目录后，把一整套现成的前端页面复制到新建的目录下，然后把vue.js文件复制到script目录下。

#### 2.2 登录页面的表单验证

##### 2.2.1 规则设定

- 用户名非空
- 密码非空

##### 2.2.2 在login.html页面中加入Vue的环境

```html
<script src="../../static/script/vue.js"></script>
```

##### 2.2.3 案例思路

<img src="images/img015.png" style="zoom:50%;" />

##### 2.2.4 代码实现

**HTML代码**

```html
 <div id="app">
    <div id="login_header">
      <a href="../../index.html">
        <img class="logo_img" alt="" src="../../static/img/logo.gif" />
      </a>
    </div>

    <div class="login_banner">
      <div id="l_content">
        <span class="login_word">欢迎登录</span>
      </div>

      <div id="content">
        <div class="login_form">
          <div class="login_box">
            <div class="tit">
              <h1>尚硅谷会员</h1>
            </div>
            <div class="msg_cont">
              <b></b>
              <span class="errorMsg">{{errorMsg}}</span>
            </div>
            <div class="form">
              <form action="login_success.html" @submit="login">
                <label>用户名称：</label>
                <input
                  class="itxt"
                  type="text"
                  placeholder="请输入用户名"
                  autocomplete="off"
                  tabindex="1"
                  name="username"
                  id="username"
                  v-model="username"
                />
                <br />
                <br />
                <label>用户密码：</label>
                <input
                  class="itxt"
                  type="password"
                  placeholder="请输入密码"
                  autocomplete="off"
                  tabindex="1"
                  name="password"
                  id="password"
                  v-model="password"
                />
                <br />
                <br />
                <input type="submit" value="登录" id="sub_btn" />
              </form>
              <div class="tit">
                <a href="regist.html">立即注册</a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div id="bottom">
      <span>
        尚硅谷书城.Copyright &copy;2015
      </span>
    </div>
  </div>
```

**Vue代码**

```javascript
<script>
    new Vue({
      "el":"#app",
      "data":{
        "username":"",
        "password":"",
        "errorMsg":"请输入用户名和密码"
      },
      "methods":{
        login(){
          //1. 验证用户名不能为空
          if(this.username==""){
            this.errorMsg="用户名不能为空";
            event.preventDefault();//取消表单提交
            return;//终止验证
          }
          //2. 验证密码不能为空
          if(this.password==""){
            this.errorMsg="密码不能为空";
            event.preventDefault();//取消表单提交
          }
        }
      }
    });
  </script>
```

#### 2.3 注册页面的表单验证

##### 2.3.1 在注册页面(register.html)中引入vue

```html
<!--在注册页面引入vue-->
<script src="../../static/script/vue.js"></script>
```

**HTML代码**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>尚硅谷会员注册页面</title>
    <link type="text/css" rel="stylesheet" href="../../static/css/style.css" />
    <link rel="stylesheet" href="../../static/css/register.css" />
    <style type="text/css">
      .login_form {
        height: 420px;
        margin-top: 25px;
      }
    </style>
  </head>
  <body>
  <div id="app">
    <div id="login_header">
      <a href="../../index.html">
        <img class="logo_img" alt="" src="../../static/img/logo.gif" />
      </a>
    </div>

    <div class="login_banner">
      <div class="register_form">
        <h1>注册尚硅谷会员</h1>
        <form action="regist_success.html" @submit="checkAll">
          <div class="form-item">
            <div>
              <label>用户名称:</label>
              <input type="text" placeholder="请输入用户名" v-model="username" @blur="checkUsername"/>
            </div>
            <span class="errMess" :style="usernameCss">{{usernameErrMsg}}</span>
          </div>
          <div class="form-item">
            <div>
              <label>用户密码:</label>
              <input type="password" placeholder="请输入密码" v-model="password" @blur="checkPassword"/>
            </div>
            <span class="errMess" :style="passwordCss">{{passwordErrMsg}}</span>
          </div>
          <div class="form-item">
            <div>
              <label>确认密码:</label>
              <input type="password" placeholder="请输入确认密码" v-model="confirmPassword" @blur="checkConfirmPassword"/>
            </div>
            <span class="errMess" :style="confirmPasswordCss">{{confirmPasswordErrMsg}}</span>
          </div>
          <div class="form-item">
            <div>
              <label>用户邮箱:</label>
              <input type="text" placeholder="请输入邮箱" v-model="email" @blur="checkEmail"/>
            </div>
            <span class="errMess" :style="emailCss">{{emailErrMsg}}</span>
          </div>
          <div class="form-item">
            <div>
              <label>验证码:</label>
              <div class="verify">
                <input type="text" placeholder="" />
                <img src="../../static/img/code.bmp" alt="" />
              </div>
            </div>
            <span class="errMess"></span>
          </div>
          <button class="btn">注册</button>
        </form>
      </div>
    </div>
    <div id="bottom">
      <span>
        尚硅谷书城.Copyright &copy;2015
      </span>
    </div>
  </div>
  </body>
</html>

```

**Vue代码**

```javascript
<script>
    new Vue({
      "el":"#app",
      "data":{
        "username":"",
        "usernameErrMsg":"用户名应为6~16位数组和字母组成",
        "usernameCss":{"visibility":"hidden"},
        "password":"",
        "passwordErrMsg":"密码的长度至少为8位",
        "passwordCss":{"visibility":"hidden"},
        "confirmPassword":"",
        "confirmPasswordErrMsg":"密码两次输入不一致",
        "confirmPasswordCss":{"visibility":"hidden"},
        "email":"",
        "emailErrMsg":"请输入正确的邮箱格式",
        "emailCss":{"visibility":"hidden"},
      },
      "methods":{
        checkUsername(){
          this.usernameCss={"visibility":"visible"}
          var reg=/^[a-zA-Z0-9]{6,16}$/;
          if(reg.test(this.username)){
            this.usernameErrMsg="√";
            return true;
          }else{
            this.usernameErrMsg="用户名应为6~16位数组和字母组成"
            return false;
          }
        },
        checkPassword(){
          this.passwordCss={"visibility":"visible"};
          var reg=/^[a-zA-Z0-9]{8,}$/;
          if(reg.test(this.password)){
            this.passwordErrMsg="√";
            return true;
          }else{
            this.passwordErrMsg="密码的长度至少为8位";
            return false;
          }
        },
        checkConfirmPassword(){
          this.confirmPasswordCss={"visibility":"visible"};
          if(this.confirmPassword==""){
            this.confirmPasswordErrMsg="密码两次输入不一致";
            return false;
          }
          if(this.password==this.confirmPassword){
            this.confirmPasswordErrMsg="√";
            return true;
          }else{
            this.confirmPasswordErrMsg="密码两次输入不一致";
            return false;
          }
        },
        checkEmail(){
          this.emailCss={"visibility":"visible"};
          var reg=/^[a-zA-Z0-9_\.-]+@([a-zA-Z0-9-]+[\.]{1})+[a-zA-Z]+$/;
          if(reg.test(this.email)){
            this.emailErrMsg="√"
            return true;
          }else{
            this.emailErrMsg="请输入正确的邮箱格式"
            return false;
          }
        },
        checkAll(){
          if(!(this.checkUsername()&this.checkPassword()&this.checkConfirmPassword()&this.checkEmail())){
            event.preventDefault()
          }
        }
      }
    })
  </script>
```

