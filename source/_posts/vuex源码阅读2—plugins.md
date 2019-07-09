---
title: vuex源码阅读2—plugins
date: 2018-09-10 22:43:48
tags: 
 - vuex
 - 源码阅读
---

这次来看plugins目录，因为store中第二个引入的模块是devtool.js，然后顺带把logger.js也看了，反正就只有2个插件。两个插件中有调用store对象的一些方法，这个在store.js中再去读好了，这边也能猜到它是干什么的。

##### devtool

这个模块导出的是一个devtoolPlugin方法，接收store作为参数。首先模块检测vue开发者工具是否安装，如果没有的话，调用devtoolPlugin方法是什么也不会干的。如果已经安装，就把devtoolHook对象存储到store._devtoolHook属性上。
发射vuex的初始化事件，让vuex拿到store实例，然后订阅mutation变动。

```javascript
// 检测vue的开发者工具是否安装
// 如果安装了，window就存在__VUE_DEVTOOLS_GLOBAL_HOOK__属性
// 是一个包含下面属性的对象 {Vue: ƒ, on: ƒ, once: ƒ, off: ƒ, emit: ƒ}
const devtoolHook =
  typeof window !== 'undefined' &&
  window.__VUE_DEVTOOLS_GLOBAL_HOOK__

export default function devtoolPlugin (store) {
  // 没有安装devtool就直接返回
  if (!devtoolHook) return

  // 把devtoolHook对象存储到store._devtoolHook属性上
  store._devtoolHook = devtoolHook

  // 发射vuex的初始化事件
  // 这样开发者工具就能拿到store实例
  devtoolHook.emit('vuex:init', store)

  // 监听vuex的vuex:travel-to-state事件
  devtoolHook.on('vuex:travel-to-state', targetState => {
    // 替换状态为目标状态
    store.replaceState(targetState)
  })

  // mutation提交改变时触发回调
  store.subscribe((mutation, state) => {
    // devtool发射vuex的mutation事件
    devtoolHook.emit('vuex:mutation', mutation, state)
  })
}
```

##### logger

这个插件是用来在matation提交变动时，控制台打印日志用的。模块也是默认导出一个方法，叫createLogger。
createLogger方法本身接收5个参数，都有默认值，然后它的返回值是一个打印分组格式化日志的方法。打印信息包含变动类型mutationType，变动事件，变动前后的状态等等。

```javascript
// Credits: borrowed code from fcomb/redux-logger
// 深拷贝，都懂，实现在util在看吧
import { deepCopy } from '../util'

// 接收5个参数，返回的是一个函数
export default function createLogger ({
  // 自动展开记录的mutation
  collapsed = true,
  // filter方法默认返回true,表示mutation需要被记录
  filter = (mutation, stateBefore, stateAfter) => true,
  // 在开始记录之前转换状态
  transformer = state => state,
  // mutation格式化方法
  mutationTransformer = mut => mut,
  // 自定义console实现，默认是console
  logger = console
} = {}) {
  return store => {
    // 深拷贝当前状态，方便和后一个状态做对比，状态改变后也可记录为上一条状态
    let prevState = deepCopy(store.state)

    // 订阅store的mutation变化，如果触发执行下面的回调
    store.subscribe((mutation, state) => {
      // 没有日志函数就什么都不做～
      if (typeof logger === 'undefined') {
        return
      }
      // 深拷贝下一个状态
      const nextState = deepCopy(state)

      // 根据传入的参数：filter()方法判断是否要记录mutation
      if (filter(mutation, prevState, nextState)) {
        const time = new Date()
        const formattedTime = ` @ ${pad(time.getHours(), 2)}:${pad(time.getMinutes(), 2)}:${pad(time.getSeconds(), 2)}.${pad(time.getMilliseconds(), 3)}`
        const formattedMutation = mutationTransformer(mutation)
        const message = `mutation ${mutation.type}${formattedTime}`
        const startMessage = collapsed
          ? logger.groupCollapsed
          : logger.group

        // render
        // 打印messagege：mutation类型+ 事件
        try {
          startMessage.call(logger, message)
        } catch (e) {
          console.log(message)
        }

        // 打印具体内容：前一个状态、变动、后一个状态
        logger.log('%c prev state', 'color: #9E9E9E; font-weight: bold', transformer(prevState))
        logger.log('%c mutation', 'color: #03A9F4; font-weight: bold', formattedMutation)
        logger.log('%c next state', 'color: #4CAF50; font-weight: bold', transformer(nextState))

        // 结束打印
        try {
          logger.groupEnd()
        } catch (e) {
          logger.log('—— log end ——')
        }
      }
      // 把变动后的状态存储为之前的状态
      prevState = nextState
    })
  }
}

// 生成重复次数字符串
function repeat (str, times) {
  return (new Array(times + 1)).join(str)
}
// 把数字转成固定位数的字符串，不足位的用0补充，maxLength表示数字的位数
// 这个挺有意思的，以前在日期格式化时，通常是用substr去做,es6+是用padStart
function pad (num, maxLength) {
  return repeat('0', maxLength - num.toString().length) + num
}

```