---
title: vuex源码阅读4—helpers与util
date: 2018-09-13 18:58:14
tags:
 - vuex
 - 源码阅读
---

##### util.js

util导出了几个工具函数，其实深拷贝是比较有意思的，带有缓存功能。直接看源码注释吧。

```JS
/**
 * Get the first item that pass the test
 * by second argument function
 * 找到列表中满足过滤条件的第一个元素，indexOf的问题在于它不能传函数
 *
 * @param {Array} list
 * @param {Function} f
 * @return {*}
 */
export function find (list, f) {
  return list.filter(f)[0]
}

/**
 * Deep copy the given object considering circular structure.
 * This function caches all nested objects and its copies.
 * If it detects circular structure, use cached copy to avoid infinite loop.
 *
 * @param {*} obj
 * @param {Array<Object>} cache
 * @return {*}
 */
export function deepCopy (obj, cache = []) {
  // just return if obj is immutable value
  // 基础类型或者函数直接返回obj
  if (obj === null || typeof obj !== 'object') {
    return obj
  }

  // if obj is hit, it is in circular structure
  // 如果是对象，对象本身是可以设置属性的
  // 对缓存数组的元素，判断其original属性是否全等于obj
  // 如果是，说明有缓存，直接返回元素的copy属性
  const hit = find(cache, c => c.original === obj)
  if (hit) {
    return hit.copy
  }

  // 创建空副本
  const copy = Array.isArray(obj) ? [] : {}
  // put the copy into cache at first
  // because we want to refer it in recursive deepCopy
  // 添加缓存
  cache.push({
    original: obj,
    copy
  })

  // 遍历对象/数组
  Object.keys(obj).forEach(key => {
    // 递归调用赋值
    copy[key] = deepCopy(obj[key], cache)
  })
  //  最后返回副本
  return copy
}

/**
 * forEach for object
 * 对对象对每一个元素，调用函数
 */
export function forEachValue (obj, fn) {
  Object.keys(obj).forEach(key => fn(obj[key], key))
}

// 判断是否为对象类型，其实是对象或数组
export function isObject (obj) {
  return obj !== null && typeof obj === 'object'
}

// 判断是否为promise,其实不太准确，这个只是判断是否为thenable对象
export function isPromise (val) {
  return val && typeof val.then === 'function'
}

// 断言函数，不满足条件则抛出异常
export function assert (condition, msg) {
  if (!condition) throw new Error(`[vuex] ${msg}`)
}

```

##### helpers.js

顾名思义，这里包含了vuex的几个辅助方法，分别是：mapState、mapMutations、mapGetters、mapActions，及createNamespacedHelpers方法。

从代码看，mapState、mapMutations、mapActions的用法都是有2种的。

看官网中的示例,`countAlias: 'count',`就是值不是函数的用法。
```JS
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

又如官网中，对mutations和actions的辅助函数的用法,值传的都是字符串：
```JS
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

```JS
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```
createNamespacedHelpers方法返回的对象包含以上4个辅助方法，且是缓存了命名空间的那种。

下面是helpers.js的完整源码～

```JS
/**
 * Reduce the code which written in Vue.js for getting the state.
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} states # Object's item can be a function which accept state and getters for param, you can do something for state and getters in it.
 * @param {Object}
 */
export const mapState = normalizeNamespace((namespace, states) => {
  const res = {}
  normalizeMap(states).forEach(({ key, val }) => {
    res[key] = function mappedState () {
      // 默认state和getters取的是全局的
      let state = this.$store.state
      let getters = this.$store.getters
      // 如果有命名空间，state和getters都取当前模块的
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapState', namespace)
        if (!module) {
          return
        }
        state = module.context.state
        getters = module.context.getters
      }
      //  如果val是函数，返回该函数的调用结果，否则返回state[val]
      return typeof val === 'function'
        ? val.call(this, state, getters)
        : state[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})

/**
 * Reduce the code which written in Vue.js for committing the mutation
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} mutations # Object's item can be a function which accept `commit` function as the first param, it can accept anthor params. You can commit mutation and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 */
export const mapMutations = normalizeNamespace((namespace, mutations) => {
  const res = {}
  normalizeMap(mutations).forEach(({ key, val }) => {
    res[key] = function mappedMutation (...args) {
      // Get the commit method from store
      // commit默认是全局，如果有命名空间，取模块中的commit
      let commit = this.$store.commit
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapMutations', namespace)
        if (!module) {
          return
        }
        commit = module.context.commit
      }
      // 如果val是函数，返回val的调用结果,commit作为第一个参数，args作为第二个参数
      // 如果val不是函数，调用commit，val作为第一个参数，args作为第二个参数
      return typeof val === 'function'
        ? val.apply(this, [commit].concat(args))
        : commit.apply(this.$store, [val].concat(args))
    }
  })
  return res
})

/**
 * Reduce the code which written in Vue.js for getting the getters
 * 代码与mapState类似，用法上也都是用在computed中
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} getters
 * @return {Object}
 */
export const mapGetters = normalizeNamespace((namespace, getters) => {
  const res = {}
  normalizeMap(getters).forEach(({ key, val }) => {
    // thie namespace has been mutate by normalizeNamespace
    // val是带命名空间的
    val = namespace + val
    res[key] = function mappedGetter () {
      if (namespace && !getModuleByNamespace(this.$store, 'mapGetters', namespace)) {
        return
      }
      if (process.env.NODE_ENV !== 'production' && !(val in this.$store.getters)) {
        console.error(`[vuex] unknown getter: ${val}`)
        return
      }
      // 值都是从全局getters取的
      return this.$store.getters[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})

/**
 * Reduce the code which written in Vue.js for dispatch the action
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} actions # Object's item can be a function which accept `dispatch` function as the first param, it can accept anthor params. You can dispatch action and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 */
export const mapActions = normalizeNamespace((namespace, actions) => {
  const res = {}
  normalizeMap(actions).forEach(({ key, val }) => {
    res[key] = function mappedAction (...args) {
      // get dispatch function from store
      // 获取全局store的dispatch
      let dispatch = this.$store.dispatch
      // 如果有命名空间，就获取模块的
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapActions', namespace)
        if (!module) {
          return
        }
        dispatch = module.context.dispatch
      }
      // 如果val是方法，调用val，否则调用dispatch
      return typeof val === 'function'
        ? val.apply(this, [dispatch].concat(args))
        : dispatch.apply(this.$store, [val].concat(args))
    }
  })
  return res
})

/**
 * 返回一个对象，对象中的map函数是一个已经接收了namespace变量的函数，调用的时候namespace就不需要传了。
 * Rebinding namespace param for mapXXX function in special scoped, and return them by simple object
 * @param {String} namespace
 * @return {Object}
 */
export const createNamespacedHelpers = (namespace) => ({
  mapState: mapState.bind(null, namespace),
  mapGetters: mapGetters.bind(null, namespace),
  mapMutations: mapMutations.bind(null, namespace),
  mapActions: mapActions.bind(null, namespace)
})

/**
 * Normalize the map
 * 把数组和对象格式化成数组，数组的每一个元素是一个对象，包含key和value，
 * 如果传入的是数组，这里对象的key和value值是一样的
 * normalizeMap([1, 2, 3]) => [ { key: 1, val: 1 }, { key: 2, val: 2 }, { key: 3, val: 3 } ]
 * normalizeMap({a: 1, b: 2, c: 3}) => [ { key: 'a', val: 1 }, { key: 'b', val: 2 }, { key: 'c', val: 3 } ]
 * @param {Array|Object} map
 * @return {Object}
 */
function normalizeMap (map) {
  return Array.isArray(map)
    ? map.map(key => ({ key, val: key }))
    : Object.keys(map).map(key => ({ key, val: map[key] }))
}

/**
 * 传入一个fn函数，返回一个参数为namespace和map、调用了fn的函数
 * Return a function expect two param contains namespace and map. it will normalize the namespace and then the param's function will handle the new namespace and the map.
 * @param {Function} fn
 * @return {Function}
 */
function normalizeNamespace (fn) {
  return (namespace, map) => {
    // 如果namespace参数不是字符串，map重新赋值为namespace，namespace重新赋值为空字符串
    if (typeof namespace !== 'string') {
      map = namespace
      namespace = ''
      // 如果namespace是字符串，判断它最后一位是不是/，不是的话就加上/
    } else if (namespace.charAt(namespace.length - 1) !== '/') {
      namespace += '/'
    }
    return fn(namespace, map)
  }
}

/**
 * 根据命名空间返回模块
 * Search a special module from store by namespace. if module not exist, print error message.
 * @param {Object} store
 * @param {String} helper
 * @param {String} namespace
 * @return {Object}
 */
function getModuleByNamespace (store, helper, namespace) {
  const module = store._modulesNamespaceMap[namespace]
  if (process.env.NODE_ENV !== 'production' && !module) {
    console.error(`[vuex] module namespace not found in ${helper}(): ${namespace}`)
  }
  return module
}
```