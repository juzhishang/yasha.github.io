---
title: vuex源码阅读1—mixin
date: 2018-09-10 20:49:55
tags:
 - vuex
 - 源码阅读
---

忧伤的故事，去面试连续三场被问到了vuex的实现原理，只好先把源码看起来了，我也没在怕的～

看下目录结构先，代码量不多的。

```
.
├── helpers.js
├── index.esm.js
├── index.js
├── mixin.js
├── module
│   ├── module-collection.js
│   └── module.js
├── plugins
│   ├── devtool.js
│   └── logger.js
├── store.js
└── util.js
```

首先，**index是入口文件，可以看出来Store是核心代码，包含了Store构造器方法，和install方法。helpers主要是一些辅助方法。**
```javascript
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```

打开store.js
开头它引入了以下模块

```javascript
import applyMixin from './mixin'
import devtoolPlugin from './plugins/devtool'
import ModuleCollection from './module/module-collection'
import { forEachValue, isObject, isPromise, assert } from './util'
```
就来看一下mixin.js的内容好了，默认导出了一个方法，这个方法接收一个参数，就是Vue。
那么，它里面做了一些什么呢：

```javascript
export default function (Vue) {
  // 大版本
  const version = Number(Vue.version.split('.')[0])

  // vue 2.x，调用mixin全局注入，在beforeCreate钩子中调用vuexInit方法
  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    // 这个思想好像是叫面向切面？
    // vue 1.x没有beforeCreate，类似钩子是init
    // 重写了Vue原型上的_init方法，就把vuexInit方法注入到init中了
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    // 获取创建vue实例时配置的选项
    const options = this.$options
    // store injection
    if (options.store) {
      // options.store暂存到this.$store中，如果它本身是一个方法，就调用一下再赋值
      // 这样，组件中就可以通过this.$store获取store对象了
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    // options本身没有store的话，this.$store就取options.parent.$store
    // 这样大概是为了保证所有组件公用一个store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```
所以，总结一下～
**mixin.js就做了一件事情：在beforeCreated钩子中调用了vuex的初始化方法，也就是在实例上添加了$store属性。**

