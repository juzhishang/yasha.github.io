---
title: 理解vuex
date: 2018-09-04 17:12:24
tags: 
 - vuex
---

用了这么久`vuex`,面试时也被问到很多次组件通信，没有好好出一篇博客是有点说不过去～

那就写吧～～

首先，**vuex是什么？**，它是类flux的状态管理的官方实现，大型应用的多组建通信的解决方案，可以帮我们处理组件之间共享状态管理的问题。

那么，**flux是什么呢**,它是一种架构模式，核心思想是数据和逻辑都是单向的，数据从`action`到`dispatcher`，再到`store`,最终到`view`的路线是单向不可逆的。

好了，再回到`vuex`。`vuex`有**5大核心**，分别是：

+ state，存放共享数据

+ mutation，变动state的方法，需要通过`store.commit()`提交变动来触发

+ getter，有点像vue中的computed属性

+ action，提交mutation的方法，可以调用`store.commit()`，它本身要通过`store.dispatch()`派发来触发。

+ module，用来分割模块，每一个模块中都可以有`state、mutations、getters、actions`属性，模块可以嵌套，所以也有可能有`modules`属性。

![vuex](https://ysha-01.oss-cn-shanghai.aliyuncs.com/vuex.png)

上图表示了`vuex`中的数据和逻辑流向。组件派发`action`,`action`提交`mutation`,`mutation`修改`state`,`state`修改后重新渲染组件。
其中，与后端接口的交互是放在`action`中的，`action`中可以有异步操作，但`mutation`中只可以有同步操作。


`vuex`与普通的全局对象有2点不同：
+ 1、它的状态存储是响应式的
+ 2、数据流是单向的，必须通过提交`mutation`显示改变



#### 用法

`vuex`使用单一状态树，每个应用只包含一个`store`实例。
最简单的使用`state`的方式是引入`store`文件，然后在`computed`中通过`store.state.xxx`获取。

更好的方式是入口文件中使用`Vue.use(Vuex)`,然后在创建`vue`实例的时候把`store`属性带上，这样子组件就可以通过`this.$store`来访问`store`，而不需要每次引入`store`文件
```javascript
// 入口函数
import Vue from 'vue';
import App from './App';
import router from './router';
import store from './store';

Vue.config.productionTip = false;

/* eslint-disable no-new */
new Vue({
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  el: '#app',
  router,
  components: { App },
  template: '<App/>',
});
```
```javascript
// 组件中
  computed: {
    tit() {
      return this.$store.state.title;
    },
  },
```

每一个核心及它们的辅助函数，可以看官网文档的具体介绍，觉得已经很全面了，这里就不浪费笔墨了。

#### DEMO

用`vue-cli`跑了一个demo,简单添加了下`vuex`的功能。
在`src`目录下窗口`store`目录，包含以下文件
```
├── actions.js
├── getters.js
├── index.js
├── mutations.js
├── state.js
└── types.js
```

```javascript
// actions.js
import * as types from './types';

export default {
  [types.COUNT](context, value) {
    context.commit(types.COUNT, value);
  },
};
```

```javascript
// getters.js
export default {
  total(state) {
    return `${state.title} - ${state.count}`;
  },
};
```

```javascript
// index.js
import Vue from 'vue';
import Vuex from 'vuex';
import actions from './actions';
import getters from './getters';
import mutations from './mutations';
import state from './state';

Vue.use(Vuex);

export default new Vuex.Store({
  state,
  mutations,
  actions,
  getters,
});
```

```javascript
// mutations.js
import * as types from './types';

export default {
  [types.TITLE](state, value) {
    state.title = value;
  },
  [types.COUNT](state, value) {
    state.count = value;
  },
};
```

```javascript
// state.js
export default {
  title: '666',
};
```

```javascript
// type.js
export const TITLE = 'title';
export const COUNT = 'count';

```

然后在入口函数中引入`store`，将`store`注入到子组件中：
```javascript
import Vue from 'vue';
import App from './App';
import router from './router';
import store from './store';

Vue.config.productionTip = false;

/* eslint-disable no-new */
new Vue({
  store,
  el: '#app',
  router,
  components: { App },
  template: '<App/>',
});

```

组件中使用：
```javascript
import * as types from '../store/types';

export default {
  name: 'HelloWorld',
  data() {
    return {
      msg: 'Welcome to Your Vue.js App',
    };
  },
  created() {
    this.$store.commit(types.TITLE, 555);
    this.$store.dispatch(types.COUNT, 5);
  },
  computed: {
    tit() {
      return this.$store.state.title;
    },
  },
};
```

这里的例子中没有用到`module`,如果是大型应用的话，推荐使用`module`。