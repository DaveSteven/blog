---
title: vue-music学习总结(一)
date: 2018-06-25 10:41:34
categories:
- Front-End
tags:
- Vue.js
---

最近在慕课上学习了vue.js开发音乐app的课程，学习到里很多东西，在这里进行总结一下。感兴趣的同学可以去看一下该课程，贴上[课程地址](https://coding.imooc.com/class/107.html)

总结分为两方面，一个是在开发项目时不知道的一些技巧，一个是开发的思想。

# 一、vue的一些使用技巧

在学习的过程中get到了以前不知道的使用技巧，也看出了自己对vue的了解甚是浅薄。

## 1. 在父组件中可以调用子组件的方法

为子组件添加 `ref` 引用，可以通过this.$refs.chilren.fn()调用该组件的方法来实现业务需求或者开发基础组件。

### 引申：Vue中的 ref
> `ref` 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上。如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例。-- Vue.js API [#ref](https://cn.vuejs.org/v2/api/#ref)

**1) 指向DOM元素**
可以使用DOM提供的原生方法，设置样式、获取宽高等等。
```Html
<template>
  <div>
    <div ref="box1"></div>
    <div ref="box2"></div>
  </div>
</template>

// 获取实际宽度：this.$refs.box1.clientWidth
// 添加样式：this.$refs.box2.style.bottom = '10px'
```

**2) 指向组件实例**
可以调用该组件实例中methods里的方法，用来开发基础组件或者实现一些业务需求。
比如开发一个最基础的弹窗组件，来控制显示隐藏。
```Html
<template>
  <div class="dialog" v-show="showFlag">
    这是一个弹窗组件
  </div>
<template>
<script>
  export default {
    data() {
      return {
        showFlag: false
      }
    },
    methods: {
      show() {
        this.showFlag = true;
      },
      hide() {
        this.showFlag = false;
      }
    }
  }
</script>
```
在父组件中调用：
```Html
<template>
  <div>
    <dialog ref="dialog" />
  </div>
</template>
<script>
  import Dialog from './dialog';

  export default {
    components: {
      Dialog
    }
  }
</script>

// 使用 this.$refs.dialog.show()，可以让弹窗显示
```

## 2.若使用的属性（变量）不需要Vue追踪依赖对其响应渲染，则不需要在data中添加此属性。
在开发过程中，有许多属性是不需要放在DOM中渲染展示的，只是在Vue实例中使用。

举个例子：
我只想在各个方法中使用这个属性（变量）。
```Javascript
export default {
  methods: {
    moveTouchStart(e) {
      this.touch.initiated = true;
      const touch = e.touches[0];
      this.touch.startX = touch.pageX;
      this.touch.startY = touch.pageY;
    },
    middleTouchMove(e) {
      if (!this.touch.initiated) {
        return;
      }
      const touch = e.touches[0];
      const deltaX = touch.pageX - this.touch.startX;
      const deltaY = touch.pageY - this.touch.startY;
      ....
    }
  }
}
```
看到上面的代码应该就很清楚了，this.touch是在方法中随时调用和改写的属性，只需要在this（Vue实例）来添加这个属性，这个属性就会存在于此实例中，可以随时调用或者更改其值。

#### 引申：Vue对data中的属性做了什么？
> 当一个 Vue 实例被创建时，它向 Vue 的响应式系统中加入了其 data 对象中能找到的所有的属性。当这些属性的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。

**如何实现响应的？**
当把一个普通的JavaScript对象传给Vue实例的 data 选项，Vue会遍历此对象所有的属性，并使用 [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 把这些属性全部转为getter/setter。

示例：
```Javascript
let obj = {};

Object.defineProperty(obj, 'name', {
  enumerable: true,
  configurable: true,
  get: function() {
    console.log('get方法被调用了')
  },
  set: function(newVal) {
    console.log(`set方法被调用了，值为：${newVal}`)
  }
})

obj.name  // get方法被调用了
obj.name = 'Dave'  //set方法被调用了，值为：Dave
```
get 和 set 方法内部的this都指向obj，意味着 get 和 set 函数可以操作对象内部的值。

这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue追踪依赖，在属性被访问和修改时通知变化。

> 每个组件实例都有相应的 watcher 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 setter 被调用时，会通知 watcher 重新计算，从而致使它关联的组件得以更新。-- Vue.js API [深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)

![](/images/reactivity.png)
图片来源：[https://cn.vuejs.org/v2/guide/reactivity.html](https://cn.vuejs.org/v2/guide/reactivity.html)