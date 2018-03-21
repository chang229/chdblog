---
title: Vue项目开发过程中遇到的一些问题总结
date: 2018-03-21 14:09:30
tags:
    - vue
category:
    - vue
---

#### 一、使用vue-cli搭建项目
1.安装vue-cli:安装好node，直接全局安装vue-cli:
```
npm install -g vue-cli
```
安装完成后，使用`vue-V`查看是否安装成功
![查看vue版本号](https://images2015.cnblogs.com/blog/1059788/201701/1059788-20170106125052316-34797974.png)
如果提示“无法识别 'vue' ” ，有可能是 npm 版本过低，可以使用 npm install -g npm 来更新版本。
2.生成项目：首先进入到项目目录中，执行命令：
```
vue init webpack Vue-Project
//其中webpack是模板名称，Vue-Project是自定义的项目名称
```
命令执行之后，会在当前目录生成一个以该名称命名的文件夹：
![vue-cli创建项目](https://images2015.cnblogs.com/blog/1059788/201701/1059788-20170106133950378-145408144.png)
配置完成之后，通过命令：`cd Vue-Project`进入项目目录，使用npm安装相关依赖
```
npm install
```
启动项目：
```
npm run dev
```
如果浏览器打开之后，没有加载出页面，有可能是本地的 8080 端口被占用，需要修改一下配置文件 config>index.js
![vue配置文件修改](https://images2015.cnblogs.com/blog/1059788/201701/1059788-20170106135204409-1735535107.png)
修改完成之后，项目就算已经搭建成功，可以开始编写自己的代码啦。

#### 二、vue.js程序启动运行时，遇到Unexpected tab character问题
在vuejs程序启动时，会遇到一种常见问题，在终端报错：Unexpected tab character。如下图：
![vue项目启动报错](http://img.blog.csdn.net/20170614091540917?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHZrZWxseQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
解决方法：
在eslint的配置文件中（.eslintrc）rules项中添加一行："no-tabs":"off"，如下图：
![vue报错解决方法](http://img.blog.csdn.net/20170614092226463?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHZrZWxseQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 三、v-for 列表数据渲染完后如何重新渲染

```
<select class="dept_select select-change" data-placeholder="请选择" style="width: 180px;" name="academys" id="academys" data-rule-required="true" v-model="Academyselect">
        <option v-for="item in academyList" v-bind:value="item.id">
            {{ item.academyName }}
        </option>
 </select>
 ```
 当option列表渲染完成之后，如果数据改变需要重新渲染列表，解决方案为：
 > 把academyList替换为新数组。官网文档以下两种数据变化vue无法检测:1.通过索引修改值;2.改变数组长度。赋值新数组不属于以上两种，有数据改变，就会有更新，记得在对应vue实例中定义academyList:[]这个数组，然后方法中赋值this.academyList就可以。

 #### 四、vue中src实现数据绑定遇到的问题
 在vue2中，src实现数据绑定稍不留神就不成功。假定value.src是绑定的数据。
常见错误写法1：
```
<img src="value.src">
```
错误之处在于：
1.属性值数据绑定应该用v-bind，应该写成v-bind:src=""
2.直接用引号"value.src"会报错，取不到值。
常见错误写法2：

```
<img src="{{value.src}}">
```

常见错误写法3：

```
<img src="{value.src}">
```

以上均会报错。正确写法：

```
<img :src="value.src">
```

#### 五、vue页面中的定时器或者滚动事件报错
当vue项目中的组件被移除后，组件中的定时器和页面滚动事件并不会跟随组件一起移除掉，因此会导致出现一些错误，或影响其它组件的正常运行。
因此离开该页面需要移除这个监听的事件,`destroyed`在组件移除后执行。
```
destroyed () {
  window.removeEventListener('scroll', this.handleScroll)
}
```

#### 六、vue中使用定时器时this指向问题
> 箭头函数中的this指向是固定不变（定义函数时的指向），在vue中指向vue;
> 普通函数中的this指向是变化的（使用函数时的指向），谁调用的指向谁。

```
created () {
    setInterval(() => { console.log(this) }, 1000) // 箭头函数中this指向vue

    setInteval(function () { console.log(this) }, 1000) // 普通函数中this指向window,因为setInterval()函数是window对象的函数
}
```
在vue的定时器函数中如果要使用普通函数，要使用了一个变量来当中间值：

```
ready: function(){
    let self = this;
    setInvertal(function(){
        for(let k in self.goods_list){
            .......
        }
    }, 1000)
}
```

#### 七、vue-router 页面切换后保持在页面顶部而不是保持原先的滚动位置的办法
vue-router有提供一个方法scrollBehavior，它可以使切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。
这个功能只在 HTML5 history 模式下可用。
```
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
    if (savedPosition) {
        return savedPosition
    } else {
        return { x: 0, y: 0 }
    }
  }
})
```

#### 八、vue关于父组件调用子组件的方法
> 父组件： 在子组件中加上ref即可通过this.$refs.ref.method调用
```
<template>
  <div @click="parentMethod">
    <children ref="c1"></children>
  </div>
</template>

<script>
  import children from 'components/children/children.vue'
  export default {
    data(){
      return {
      }
    },
    computed: {
    },
    components: {
      'children': children
    },
    methods:{
      parentMethod() {
        console.log(this.$refs.c1) //返回的是一个vue对象，所以可以直接调用其方法
        this.$refs.c1.childMethod();
      }
    },
    created(){
    }
  }
</script>
```

#### 九、关于Vue背景图打包之后访问路径错误问题
通过vue-cli脚手架创建的vue项目，注意相对路径标识’./’,我们知道img为html标签，他的路径是由index.html开始访问的，他走./static/img/'图片名'是能正确访问到图片的，所以img的路径没问题，然后app.css访问./static/img/'图片名'是访问错误的，因为在css目录下并没有static目录。这样就造成了路径访问失败的问题。
解决办法：
> 1.检查config文件中的assetsPublicPath是否设置为’/’而不是’./’ ,注意区分’/’为绝对路径，绝对路径从网站静态服务器根目录查询/static/img/图片，这样的路径就是正确的，如果设置为’./’，路径就变成了相对路径，相对路径会根据相对的文件目录来确定文件资源，会造成上面分析的问题
> 2.vue-cli创建的vue项目，会自带一个static目录，将出错图片放在该目录下面即可（正确解决办法） 查看vue-cli创建项目的webpack配置文件可以找到一个将static目录拷贝到dist目录中。根据对资源的规划不同，将需要打包的资源放在src/assets目录中，不需要打包的资源放入static目录中。

#### 十、vue2绑定内联样式background的一些坑
有一个需求是，给一个盒子添加一个背景图片，这个背景图片是动态请求回来的，那么应该怎么做？
正常情况下的vue内联样式如下写法：
```
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
data: {
  activeColor: 'red',
  fontSize: 30
}
```
此时的style绑定的是一个JavaScript对象，在JavaScript中不允许出现 "-" ，那么绑定一个背景图片应该这么写：
```
<div :style="{background: 'url('+ img +')'，backgroundSize:cover }"></div>
data:{
  img:'xxx.png'
}
```
将图片改为动态请求回来的：
```
<div :style="{background: 'url('+ img +')'，backgroundSize:cover }"></div>
data:{
    img:'xxx.png'
},
methods:{
// 伪代码 请求数据
      getImg(){
        this.$http.get().then(function (e) {
          this.img = e.data //将数据赋值给img
        }.bind(this))
      }
}，
created(){
// 调用函数
    this.getImg()
}

```
因为在生命周期 mounted 之前都是虚拟dom 也就是说 当页面已经渲染完，但是vue还没执行，所有数据丢失，此时我们加上
```
<div v-if='img ' :style="{background: 'url('+ img +')'，backgroundSize:cover }"></div>
```
表示有img属性的时候我们选择这个元素,至此就成功绑定背景图片了。
