---
title: vuex源码阅读3—module
date: 2018-09-11 07:36:04
tags:
 - vuex
 - 源码阅读
---

今天来看下模块。在vuex源码中，module目录只包含了module.js与module-collection.js这2个文件。

首先来看module.js，因为它是被module-collection.js依赖的～

代码片段的高亮有问题，将就着看吧，有空换个模板。。。

##### module

导出的是一个Module类，本身代码是简单的，实例中会包含如下实例属性

+ runtime， 记录模块是否在运行时
+ _children，用来存储子模块
+ _rawModule，存储原始模块
+ state,原始模块的state

以及原型中包含

+ namespaced，访问器属性，表示是否有命名空间
+ 子模块的添加、移除、和获取方法
+ 更新原始模块的方法
+ 对原始模块中_children、getters、mutations、actions的遍历方法

```js
import { forEachValue } from '../util'

// Base data struct for store's module, package with some attribute and method
// store的Module类
export default class Module {
  // 接收2个参数，分别是原始模块和运行时
  constructor (rawModule, runtime) {
    // 设置实例的运行时属性
    this.runtime = runtime
    // Store some children item
    // 设置实例的_children属性，默认是空对象（没有原型的）
    this._children = Object.create(null)
    // Store the origin module object which passed by programmer
    // 设置实例的_rawModule属性
    this._rawModule = rawModule
    // rawState来源是原始模块的state属性
    const rawState = rawModule.state

    // Store the origin module's state
    // 设置实例的state属性，值来源是rawState方法的执行结果
    // 或者rawState本身，若不存在rawState，则是空对象
    this.state = (typeof rawState === 'function' ? rawState() : rawState) || {}
  }

  // 返回原始模块是否存在namespaced属性
  // 访问器属性
  get namespaced () {
    return !!this._rawModule.namespaced
  }

  // 在_children上添加子模块
  addChild (key, module) {
    this._children[key] = module
  }

  // 移除子模块
  removeChild (key) {
    delete this._children[key]
  }

  // 获取子模块
  getChild (key) {
    return this._children[key]
  }

  // 更新原始模块的namespaced、actions、mutations和getters
  update (rawModule) {
    this._rawModule.namespaced = rawModule.namespaced
    if (rawModule.actions) {
      this._rawModule.actions = rawModule.actions
    }
    if (rawModule.mutations) {
      this._rawModule.mutations = rawModule.mutations
    }
    if (rawModule.getters) {
      this._rawModule.getters = rawModule.getters
    }
  }

  // 遍历子模块，调用fn方法
  forEachChild (fn) {
    forEachValue(this._children, fn)
  }

  // 遍历原始模块的getters，调用fn
  forEachGetter (fn) {
    if (this._rawModule.getters) {
      forEachValue(this._rawModule.getters, fn)
    }
  }

  // 遍历原始模块的actions，调用fn
  forEachAction (fn) {
    if (this._rawModule.actions) {
      forEachValue(this._rawModule.actions, fn)
    }
  }

  // 遍历原始模块的mutations,调用fn
  forEachMutation (fn) {
    if (this._rawModule.mutations) {
      forEachValue(this._rawModule.mutations, fn)
    }
  }
}

```

##### module-collection

这个也是导出一个类：ModuleCollection。它的作用就是根据传入的对象，创建模块树。构造器方法就是调用了原型方法register，进行模块注册。

原型上包含如下方法：

+ 根据路径获取模块
+ 根据路径获取命名空间
+ 更新原始模块
+ 模块的注册及取消注册

```js
import Module from './module'
// assert是个断言函数，如果条件不满足会抛出错误
import { assert, forEachValue } from '../util'

// 模块集合构造器
export default class ModuleCollection {
  constructor (rawRootModule) {
    // register root module (Vuex.Store options)
    // 注册根模块
    this.register([], rawRootModule, false)
  }

  // 根据路径获取子模块
  get (path) {
    return path.reduce((module, key) => {
      return module.getChild(key)
    }, this.root)
  }
  // 根据路径获取子模块的命名空间
  getNamespace (path) {
    let module = this.root
    return path.reduce((namespace, key) => {
      module = module.getChild(key)
      return namespace + (module.namespaced ? key + '/' : '')
    }, '')
  }

  // 更新原始根模块
  update (rawRootModule) {
    // 更新this.root属性
    update([], this.root, rawRootModule)
  }
  // 注册模块
  register (path, rawModule, runtime = true) {
    // 非生产环境，断言原始模块
    if (process.env.NODE_ENV !== 'production') {
      assertRawModule(path, rawModule)
    }

    // 创建模块实例
    const newModule = new Module(rawModule, runtime)
    // path是空数组，直接赋值
    if (path.length === 0) {
      this.root = newModule
    } else {
      // path.slice(0, -1)返回的是一个删除最后一个元素的新数组
      // 为什么要这个调用this.get呢，
      // 因此此时最后一个元素的模块还不存在，它的要在调用addChild添加才有的
      // 获取父模块
      const parent = this.get(path.slice(0, -1))
      // 在父模块中插入子模块
      parent.addChild(path[path.length - 1], newModule)
    }

    // register nested modules
    // 如果原始模块存在modules对象，说明是嵌套的，需要递归调用来注册
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
  }

  // 取消注册
  unregister (path) {
    // 获取父对象
    const parent = this.get(path.slice(0, -1))
    // 获取路径数组的最后一个元素
    const key = path[path.length - 1]
    // 判断当前子模块是否处于运行时，如果否，可以直接移除子模块
    if (!parent.getChild(key).runtime) return

    parent.removeChild(key)
  }
}

// 更新模块
function update (path, targetModule, newModule) {
  // 非生产环境，断言原始模块
  if (process.env.NODE_ENV !== 'production') {
    assertRawModule(path, newModule)
  }

  // update target module
  // 更新模块，update方法在module.js中已经看到过了，
  // 它是被添加在Module的原型上，每一个module实例都有该方法
  // 这里只涉及模块中namespaced、actions、mutations和getters的更新
  targetModule.update(newModule)

  // update nested modules
  // 更新嵌套模块
  if (newModule.modules) {
    // 遍历子模块
    for (const key in newModule.modules) {
      // 如果目标模块上没有对应的key,在非生产环境打印警告日志，生产环境则直接返回
      if (!targetModule.getChild(key)) {
        if (process.env.NODE_ENV !== 'production') {
          console.warn(
            `[vuex] trying to add a new module '${key}' on hot reloading, ` +
            'manual reload is needed'
          )
        }
        return
      }
      // 递归调用更新方法
      update(
        path.concat(key),
        targetModule.getChild(key),
        newModule.modules[key]
      )
    }
  }
}

/**
 * 下面2个断言配置对象，都包含assert和expected属性
 * assert表示条件判断
 * expected表示期待的类型
 */
const functionAssert = {
  assert: value => typeof value === 'function',
  expected: 'function'
}

// 函数或者带handler方法的对象
const objectAssert = {
  assert: value => typeof value === 'function' ||
    (typeof value === 'object' && typeof value.handler === 'function'),
  expected: 'function or object with "handler" function'
}

// 断言的类型，getters和mutations对应“函数断言”，actions对应“对象断言”
const assertTypes = {
  getters: functionAssert,
  mutations: functionAssert,
  actions: objectAssert
}

// 断言原始模块
function assertRawModule (path, rawModule) {
  // 遍历['getters'，'mutations'，'actions']
  // 如果原始对象不包含该属性，就返回
  Object.keys(assertTypes).forEach(key => {
    if (!rawModule[key]) return

    // 断言配置
    const assertOptions = assertTypes[key]

    // 遍历原始模块的getters|mutations|actions
    // 对getters|mutations|actions中的每一个属性做断言
    // 判断类型是否符合，如果不符合，抛出错误信息
    forEachValue(rawModule[key], (value, type) => {
      assert(
        assertOptions.assert(value),
        makeAssertionMessage(path, key, type, value, assertOptions.expected)
      )
    })
  })
}

// 详细的错误信息
function makeAssertionMessage (path, key, type, value, expected) {
  let buf = `${key} should be ${expected} but "${key}.${type}"`
  if (path.length > 0) {
    buf += ` in module "${path.join('.')}"`
  }
  buf += ` is ${JSON.stringify(value)}.`
  return buf
}

```