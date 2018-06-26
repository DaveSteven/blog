---
title: vue-music学习总结(二)
date: 2018-06-26 11:18:13
categories:
- Front-End
tags:
- Vue.js
---
## 3. mixins
在这个项目的开发中，使用了Vue的 `mixins`，将多个业务模块的重复代码全都提取出来进行了复用，让我感受到了mixin的魅力。

#### 解释
> `mixins` 选项接受一个混入对象的数组。这些混入实例对象可以像正常的实例对象一样包含选项，他们将在 `Vue.extend()` 里最终选择使用相同的选项合并逻辑合并。举例：如果你的混入包含一个钩子而创建组件本身也有一个，两个函数将被调用。
Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。
-- Vue.js API [#mixins](https://cn.vuejs.org/v2/api/#mixins)

#### 示例
```Javascript
var mixin = {
  created() {
    console.log(1);
  }
  var vm = new Vue({
    created() {
      console.log(2);
    },
    mixins: [mixin]
  })
}

// => 1
// => 2
```

#### 使用方法
在开发过程中，不少业务组件共用相同的代码，都可以提取到mixin中。

**示例：搜索业务**
```Javascript
export const searchMixin = {
  data() {
    return {
      query: ''  // 多个业务组件调用一个公共组件，需要传一样的参数
    };
  },
  computed: {
    ...mapGetters([
      'searchHistory' // 多个业务组件需要使用该state属性
    ])
  },
  // 公共方法
  methods: {
    addQuery(query) {
      this.$refs.searchBox.setQuery(query);
    },
    onQueryChange(query) {
      this.query = query;
    },
    blurInput() {
      this.$refs.searchBox.blur();
    },
    saveSearch() {
      this.saveSearchHistory(this.query);
    },
    ...mapActions([
      'saveSearchHistory',
      'deleteSearchHistory'
    ])
  }
};
```
添加到要使用的组件中：
```Html
<template>
  <div>
    <search-list
      @select="addQuery"
      @delete="deleteSearchHistory" // 使用了mixin中的mapActions的方法
      :searches="searchHistory"
    >
    </search-list>
  </div>
</template>
<script>
import { searchMixin } from 'common/js/mixin';

export default {
  mixins: [
    searchMixin
  ],
  methods: {
    onQueryChange(query) {
      this.query = query;  // 使用了mixin中的data对象的属性
    }
  }
}
</script>
```
简而言之，在mixin中添加的data、computed、methods、watch及hook等都会合并到该组件实例中，可以进行调用。
这样大大减少了代码冗余，提高了灵活性，各个业务组件虽然共用相同的属性及方法，却是独立互不干扰的，为以后的业务扩展提供了极大的方便。

## 4. keep-alive

在vue-music项目中，有4个导航的切换。
<img src="/images/keep_alive.gif" style="display: inline-block !important">
导航每次切换时，组件都会再次请求数据并重新渲染，这样的体验很不好，并且增加了网络请求的次数。

使用 `keep-alive` 可解决此问题。

#### 使用方法
```Html
<keep-alive>
  <component></component>
</keep-alive>
```
将需要缓存的组件嵌套在 `keep-alive` 标签中，即可缓存该组件。
在vue-music项目中四个组件注册在了路由中，并且每个组件都有网络请求，所以为了把所有组件都缓存下来，我将 `router-view` 嵌套在了 `keep-alive` 中。
```Html
<template>
  <div id="app">
    ...
    <keep-alive>
      <router-view/>
    </keep-alive>
    ...
  </div>
</template>
```

**使用之后**
<img src="/images/keep_alive2.gif" style="display: inline-block !important">