# vue-source-learn
Vue.js 2 source code learning

## flow

```js
flow
├── compiler.js        # 编译相关
├── component.js       # 组件数据结构
├── global-api.js      # Global API 结构
├── modules.js         # 第三方库定义
├── options.js         # 选项相关
├── ssr.js             # 服务端渲染相关
├── vnode.js           # 虚拟 node 相关
```

## 源码目录设计

src/

```js
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```

- compiler
  包含 Vue.js 所有编译相关的代码，包括把模板解析成 AST 语法树，AST 语法树优化，代码生成等功能
- core
  包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等
- platforms
  platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js
- server
  Vue.js 2.0 支持了服务端渲染，所有服务端渲染相关的逻辑都在这个目录下
- sfc
  把 .vue 文件内容解析成一个 JavaScript 的对象
- shared
  Vue.js 会定义一些工具方法，这里定义的工具方法都是会被浏览器端的 Vue.js 和服务端的 Vue.js 所共享的

## 源码构建

Vue.js 源码是基于 [Rollup](https://github.com/rollup/rollup) 构建的，它的构建相关配置都在 scripts 目录下。

从打包命令看vue源码的构建过程

`"build": "node scripts/build.js"`

### Runtime Only VS Runtime+Compiler

最终都会编译成render函数来渲染

- Runtime Only 运行时不带编译，离线编译

我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。

- Runtime+Compiler 不对代码做预编译，舍弃template模板

我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板，如下所示：

```
// 需要编译器的版本 Runtime+Compiler
new Vue({
  template: '<div>{{ hi }}</div>'
})
 
// 这种情况不需要 Runtime Only
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```

## 入口

分析运行时编译构建出来的Vue.js，它的入口（当我们的代码执行 `import Vue from 'vue'` 的时候）是 `src/platforms/web/entry-runtime-with-compiler.js`：

一路往上查询，最终发现Vue是ES5构造器，在`./instance/index`中定义，需要`new Vue` 去实例化。

- 为什么不使用 ES6 的 class 实现

  ```
  initMixin(Vue)
  stateMixin(Vue)
  eventsMixin(Vue)
  lifecycleMixin(Vue)
  renderMixin(Vue)
  ```

  Vue 在创建实例后调用了很多个 mixin 的方法，每一个 mixin 方法都往 Vue 原型上挂载了很多属性和方法，Vue 按功能把这些扩展分散到多个模块（不同文件）中去实现（便于维护和管理），用 class 很难实现这样的功能

