---
title: vuex源码阅读5—store
date: 2018-09-14 13:08:21
tags:
 - vuex
 - 源码阅读
---

最后一块内容，终于读到vuex的核心代码。

首先是引入依赖,现在再看这些依赖，我们已经很熟悉它们的功能了。
```javascript
import applyMixin from './mixin'
import devtoolPlugin from './plugins/devtool'
import ModuleCollection from './module/module-collection'
import { forEachValue, isObject, isPromise, assert } from './util'
```

代码中定义了一个变量`let Vue`,它是用来标志vuex插件是否已经安装。

store模块导出了2个东西，一是Store类，二是install方法，但我们需要先看一下模块中都一些内部方法。

**`unifyObjectStyle`**

这只是一个格式化方法，vuex.Store的实力方法commit和dispatch，都有两种写法。因此需要`unifyObjectStyle`进行格式化，确保输入参数统一。

```javascript
commit(type: string, payload?: any, options?: Object)
commit(mutation: Object, options?: Object)
```
```javascript
dispatch(type: string, payload?: any, options?: Object)
dispatch(action: Object, options?: Object)
```

```javascript
// 统一对象样式，调整参数
function unifyObjectStyle (type, payload, options) {
  // 如果是对象或数组，且存在type属性
  if (isObject(type) && type.type) {
    // 调整参数
    options = payload
    payload = type
    type = type.type
  }

  // 非开发环境，断言type类型
  if (process.env.NODE_ENV !== 'production') {
    assert(typeof type === 'string', `expects string as the type, but found ${typeof type}.`)
  }
  // 返回包含这三个参数的对象
  return { type, payload, options }
}
```

**`enableStrictMode`**

启用严格模式，store._vm其实是一个vue实例，因此可以调用它的$watch，开启深度监听。

```javascript
// 启用严格模式
function enableStrictMode (store) {
  // 深度、异步监听state
  // 利用vue的$watch实现
  store._vm.$watch(function () { return this._data.$$state }, () => {
    // 如果是非生产环境，断言store._committing
    // 不是通过commit提交变动，会提示
    if (process.env.NODE_ENV !== 'production') {
      assert(store._committing, `do not mutate vuex store state outside mutation handlers.`)
    }
  }, { deep: true, sync: true })
}
```
**`getNestedState`**

获取嵌套对象中对应路径的state

```javascript
// 获取对应路径State
function getNestedState (state, path) {
  // path如果是空数组，直接返回原state
  // 否则，返回嵌套的State
  return path.length
    ? path.reduce((state, key) => state[key], state)
    : state
}
```

**`registerMutation registerAction registerGetter`**


```javascript
// 注册mutation
function registerMutation (store, type, handler, local) {
  // 获取变动类型对应的数组，如果不存在则创建，这就是个执行队列数组
  const entry = store._mutations[type] || (store._mutations[type] = [])
  // 将事件处理函数包装一层，推到数组中
  //  这样当commit提交这个变动的时候就会触发执行队列中的函数
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload)
  })
}

// 注册action
function registerAction (store, type, handler, local) {
  // 同上面mutation一样，也是个执行队列
  const entry = store._actions[type] || (store._actions[type] = [])
  // 也是将包装过的函数推到执行队列中
  entry.push(function wrappedActionHandler (payload, cb) {
    // res为处理函数的调用结果
    let res = handler.call(store, {
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb)
    // 如果res不是promise,转为promise
    if (!isPromise(res)) {
      res = Promise.resolve(res)
    }
    // 如果有安装开发者工具,返回带catch的promise
    if (store._devtoolHook) {
      return res.catch(err => {
        // devtool触发vuex的error
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      // 没有直接返回res
      return res
    }
  })
}

// 注册getter
function registerGetter (store, type, rawGetter, local) {
  // 如果getter已经存在，非生产环境提示重复的getter
  if (store._wrappedGetters[type]) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate getter key: ${type}`)
    }
    return
  }
  // 将getter方法挂载在store._wrappedGetters[type]
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  }
}
```

**`makeLocalGetters`**

根据命名空间，从store中获取对应的getter

```javascript
// 获取模块内的getter
function makeLocalGetters (store, namespace) {
  const gettersProxy = {}
  // 暂存命名空间的长度，方便遍历时对getters的每一个key截取
  const splitPos = namespace.length
  // 遍历store.getters
  Object.keys(store.getters).forEach(type => {
    // skip if the target getter is not match this namespace
    // 如果截取的字符串与命名空间不匹配则跳过
    if (type.slice(0, splitPos) !== namespace) return

    // extract local getter type
    // 剩余部分就是模块内的getter的key名
    const localType = type.slice(splitPos)

    // Add a port to the getters proxy.
    // Define as getter property because
    // we do not want to evaluate the getters in this time.
    // 在“代理”上定义本地模块的getters属性
    // 实际上它的值还是从全局store中读取的
    Object.defineProperty(gettersProxy, localType, {
      get: () => store.getters[type],
      enumerable: true
    })
  })
  //  返回代理
  return gettersProxy
}
```

**`makeLocalContext`**

获取本地上下文，它调用了上面的makeLocalGetters方法，因为本地上下文中也包含getters/state/dispatch/commit，而getters是要从根store中去获取的。

```javascript
/**
 * 获取本地模块上下文
 * make localized dispatch, commit, getters and state
 * if there is no namespace, just use root ones
 */
function makeLocalContext (store, namespace, path) {
  // 若namespace为‘’，noNamespace为true
  const noNamespace = namespace === ''
  // local对象，包含dispatch和commit方法
  const local = {
    // 无命名空间，方法是store.dispatch，否则方法是一个箭头函数
    dispatch: noNamespace ? store.dispatch : (_type, _payload, _options) => {
      // 整理参数
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args
      // 如果options不存在，或者options.root为假
      if (!options || !options.root) {
        // 拼接action类型
        type = namespace + type
        // 非生产环境且action类型不存在则报错
        if (process.env.NODE_ENV !== 'production' && !store._actions[type]) {
          console.error(`[vuex] unknown local action type: ${args.type}, global type: ${type}`)
          return
        }
      }
      // 内部调用store.dispatch方法
      return store.dispatch(type, payload)
    },
    // 无命名空间，方法是store.commit，
    // 否则，方法是一个箭头函数
    commit: noNamespace ? store.commit : (_type, _payload, _options) => {
      // 获取整理后的参数
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args
      // options选项不存在或者options.root为假
      if (!options || !options.root) {
        // 拼接变动类型
        type = namespace + type
        // 非生产环境且变动类型不存在则报错
        if (process.env.NODE_ENV !== 'production' && !store._mutations[type]) {
          console.error(`[vuex] unknown local mutation type: ${args.type}, global type: ${type}`)
          return
        }
      }
      //  箭头函数内部其实也调用了store.commit方法
      store.commit(type, payload, options)
    }
  }

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      // 根据是否有命名空间返回相应的getters
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    state: {
      get: () => getNestedState(store.state, path)
    }
  })
  // 最后返回local对象
  // 也就是上下文中包含了getters/state/dispatch/commit
  return local
}
```

**`installModule`**

安装模块，包括设置命名空间，注册mutation、action、getter和子模块。这个方法挺重要的，在调用Store类的构造器方法、注册模块方法和重置store方法都要用到。

```javascript
// 安装模块，设置命名空间，注册mutation、action、getter和子模块
function installModule (store, rootState, path, module, hot) {
  // 根据path数组的长度判断是否是根state
  const isRoot = !path.length
  // 根据路径获取子模块的命名空间
  const namespace = store._modules.getNamespace(path)

  // register in namespace map
  // 如果模块有命名空间，添加到store的模块命名空间映射表中
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }

  // set state
  // 如果不是根state且不是热更新
  if (!isRoot && !hot) {
    // 获取父级state
    const parentState = getNestedState(rootState, path.slice(0, -1))
    // 获取模块名
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      // 这样模块的state就是响应式
      Vue.set(parentState, moduleName, module.state)
    })
  }

  // 设置并暂存module.context（本地上下文）
  const local = module.context = makeLocalContext(store, namespace, path)

  // 遍历注册mutation
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  // 遍历注册action
  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  // 遍历注册getter
  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  // 遍历安装子模块
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

**`resetStore resetStoreVM`**
两个重置方法

```javascript
// 重置整个store
// 此方法在热更新和注销模块的时候会用到
function resetStore (store, hot) {
  // 将传入的store实例的_actions，_mutations，
  // _wrappedGetters，_modulesNamespaceMap置为null
  store._actions = Object.create(null)
  store._mutations = Object.create(null)
  store._wrappedGetters = Object.create(null)
  store._modulesNamespaceMap = Object.create(null)
  const state = store.state
  // 重新安装模块和重置VM
  // init all modules
  installModule(store, state, [], store._modules.root, true)
  // reset vm
  resetStoreVM(store, state, hot)
}

// 重新设置Store的vm,在vuex中就是store._vm属性，它存的实际上是一个vue实例
// 只不过这个实例比较特殊，它没有template,只有state和computed
function resetStoreVM (store, state, hot) {
  // 存储原有store._vm
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  // store._wrappedGetters缓存了当前store中所有的getter
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  // 遍历,获取每个getter的key及对应的方法
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism
    // 在computed对象中设置getter的同名属性
    // 对应的值是一个监听函数，内部调用了fn(即getter方法)
    computed[key] = () => fn(store)
    // 代理getters,获取getter时，实际上是获取store._vm（一个vue实例）上的计算属性
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  // 如果silent为true，表示取消警告日志
  const silent = Vue.config.silent
  // 这里暂时设置为slient
  Vue.config.silent = true
  // 创建一个vue实例，赋值给store._vm
  // 实例只有data和computed属性
  // data中只有一个$$state，存储的就是state
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  // 上面代码执行完后又重新设置为原来的模式
  Vue.config.silent = silent

  // enable strict mode for new vm
  // 如果store.strict为true,开启严格模式
  if (store.strict) {
    enableStrictMode(store)
  }

  // 如果存在旧vm且是热更新
  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        // 原有vm的state设为null,这样强制getter都计算一遍
        oldVm._data.$$state = null
      })
    }
    // 销毁旧的实例
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```

**`genericSubscribe`**

```javascript
// 传入参数为fn函数，和subs数组
// 返回一个从subs中删除fn的箭头函数
function genericSubscribe (fn, subs) {
  // 如果数组中找不到该方法，就添加到数组
  if (subs.indexOf(fn) < 0) {
    subs.push(fn)
  }
  return () => {
    // 找到数组中函数的下标,然后删除该元素
    const i = subs.indexOf(fn)
    if (i > -1) {
      subs.splice(i, 1)
    }
  }
}
```

好了，现在只剩下install和Store了。
先看install,既然vue在使用vuex是通过use方法，那vuex必须要有install方法了。install方法中会保证vuex只被安装一次，但它的核心代码很简单，就是`applyMixin(Vue)`,调用mixin方法，这个在vue挂载前就能执行vuex的初始化方法。

**`install`**
```javascript
// 导出install方法
export function install (_Vue) {
  // 如果Vue已经存在，非生产环境提示vuex已安装
  // 保证install里面的代码只执行一次
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  // 不存在则为Vue赋值
  Vue = _Vue
  // 调用mixin方法，把vuexInit方法注入到beforeCreated/init中
  applyMixin(Vue)
}
```

**`Store`**

```javascript
// 导出Store类
export class Store {
  constructor (options = {}) {
    // Auto install if it is not done yet and `window` has `Vue`.
    // To allow users to avoid auto-installation in some cases,
    // this code should be placed here. See #731
    // 浏览器环境下vuex还没安装，调用install方法自动安装
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.Vue)
    }

    // 非生产环境的一些断言,保证vuex已安装，保证支持promise，保证Store类是通过new方式调用
    if (process.env.NODE_ENV !== 'production') {
      assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
      assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
      assert(this instanceof Store, `store must be called with the new operator.`)
    }

    const {
      plugins = [],
      strict = false
    } = options

    // store internal state
    // 初始化一些内部值
    // _committing表示是否在提交状态，默认为假，表示可以修改state。
    // 作用是保证对Vuex中state的修改只能在mutation的回调函数中，而不能在外部随意修改state
    this._committing = false
    this._actions = Object.create(null)
    this._actionSubscribers = []
    this._mutations = Object.create(null)
    // 存放所有getters
    this._wrappedGetters = Object.create(null)
    this._modules = new ModuleCollection(options)
    // 存放module和其namespace的对应关系
    this._modulesNamespaceMap = Object.create(null)
    // 存储所有对mutation变化的订阅者
    this._subscribers = []
    // Vue实例，主要是利用Vue实例方法$watch来观测变化的
    this._watcherVM = new Vue()

    // bind commit and dispatch to self
    // 绑定Store类的dispatch和commit方法到当前实例上
    const store = this
    const { dispatch, commit } = this
    // 绑定dispatch
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    // 绑定commit
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    // 是否开启严格模式，在严格模式下，任何 mutation 处理函数以外修改 Vuex state 都会抛出错误
    // 但开启会影响性能
    this.strict = strict

    // 根state
    const state = this._modules.root.state

    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    // 安装模块，这也同时递归注册了所有子模块，收集所有模块的getter到_wrappedGetters中去，
    // this._modules.root代表根模块才独有保存的Module对象
    installModule(this, state, [], this._modules.root)

    // initialize the store vm, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    // 重设vm,本质vm就是vue的一个实例
    resetStoreVM(this, state)

    // apply plugins
    // 调用插件
    plugins.forEach(plugin => plugin(this))

    // 调用devtool插件
    if (Vue.config.devtools) {
      devtoolPlugin(this)
    }
  }

  // 获取state对象
  get state () {
    return this._vm._data.$$state
  }
  // 设置state对象，已经不能用了，在非生产环境会提示使用store.replaceState()替代
  set state (v) {
    if (process.env.NODE_ENV !== 'production') {
      assert(false, `use store.replaceState() to explicit replace store state.`)
    }
  }

  commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    // 根据变动类型获取对应的回调函数集合
    const entry = this._mutations[type]
    // 如果未传，非生产环境报错
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    // 执行mutation中的所有方法
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    // 通知所有订阅者
    // Store给外部提供了一个subscribe方法，用以注册一个订阅函数，会push到Store实例的_subscribers中，
    // 同时返回一个从_subscribers中注销该订阅者的方法。
    this._subscribers.forEach(sub => sub(mutation, this.state))

    // 非生产环境，发现options.silent选项，提示Silent已废弃
    if (
      process.env.NODE_ENV !== 'production' &&
      options && options.silent
    ) {
      console.warn(
        `[vuex] mutation type: ${type}. Silent option has been removed. ` +
        'Use the filter functionality in the vue-devtools'
      )
    }
  }

  dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }
    // 遍历通知订阅者
    this._actionSubscribers.forEach(sub => sub(action, this.state))

    // 判断entry的长度，如果大于1，调用Promise.all执行异步函数数组，否则直接调用entry的第一个元素
    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }

  subscribe (fn) {
    return genericSubscribe(fn, this._subscribers)
  }

  subscribeAction (fn) {
    return genericSubscribe(fn, this._actionSubscribers)
  }

  watch (getter, cb, options) {
    if (process.env.NODE_ENV !== 'production') {
      assert(typeof getter === 'function', `store.watch only accepts a function.`)
    }
    // 返回的是vue实例的$watch方法
    return this._watcherVM.$watch(() => getter(this.state, this.getters), cb, options)
  }

  // 替换state
  replaceState (state) {
    this._withCommit(() => {
      this._vm._data.$$state = state
    })
  }

  // 注册模块
  registerModule (path, rawModule, options = {}) {
    // path转成数组
    if (typeof path === 'string') path = [path]
    //  非生产环境，断言path是否非空数组
    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
      assert(path.length > 0, 'cannot register the root module by using registerModule.')
    }
    // 调用模块的register方法
    this._modules.register(path, rawModule)
    // 安装模块
    installModule(this, this.state, path, this._modules.get(path), options.preserveState)
    // reset store to update getters...
    // 重置vm
    resetStoreVM(this, this.state)
  }

  // 解除注册
  unregisterModule (path) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
    }

    this._modules.unregister(path)
    this._withCommit(() => {
      // 获取父state
      const parentState = getNestedState(this.state, path.slice(0, -1))
      // 删除子模块,利用vue.delete方法，确保模块在被删除的时候，视图能监听到变化
      Vue.delete(parentState, path[path.length - 1])
    })
    resetStore(this)
  }

  // 热更新,Vuex 支持在开发过程中热重载 mutation、modules、actions、和getters
  hotUpdate (newOptions) {
    this._modules.update(newOptions)
    resetStore(this, true)
  }
  // _withCommit是一个代理方法，所有触发mutation的进行state修改的
  // 操作都经过它，由此来统一管理监控state状态的修改
  // 保证在同步修改state的过程中，this._committing 的值始终为true
  _withCommit (fn) {
    // 暂存this._committing
    const committing = this._committing
    // this._committing设置为true
    this._committing = true
    // 调用fn后，this._committing设置为原来的状态
    fn()
    this._committing = committing
  }
}
```

Store中包含构造器方法、存取器state以及下面原型方法：

+ commit
+ dispatch
+ subscribe
+ subscribeAction
+ watch
+ replaceState
+ registerModule
+ unregisterModule
+ hotUpdate
+ _withCommit (内部方法)

