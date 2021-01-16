### 一. Vue 3.0 介绍

#### 1. Vue.js 3.0 源码组织方式

- 与 Vue 2.x 的区别
  - 源码组织方式的变化
    - 源码全部采用 TypeScript 重写
    - 项目的组织方式采用 monorepo 来管理项目结构，把独立的功能模块提取到不同的包中
  - Composition API（组合 API）
    - 解决 Vue 2.x 开发大型项目时，遇到超大组件使用 Options API 不好拆分和重用的问题
  - 性能提升
    - 使用 Proxy 重写的响应式代码
    - 对编译器做了优化
    - 重写了虚拟 DOM
    - 让渲染和 update 的性能有了大幅度的提升
  - Vite
    - 官方提供的开发工具
    - 在开发阶段在测试项目的时候，不需要打包可以直接运行项目，提升开发效率

#### 2. Vue 3.0 不同构建版本

- 不再构建 UMD 模块化的方式，因为 UMD 模块化的方式会让代码有更多的冗余
- 将 cjs，ES Module，自执行函数的方式分别打包到了不同的文件中
- 构建版本
  - cjs（common js）
    - vue.cjs.js
    - vue.cjs.prod.js
  - global (全局)
    - vue.global.js
    - vue.global.prod.js
    - vue.runtime.global.js
    - vue.runtime.global.prod.js
  - browser（浏览器）
    - vue.esm-browser.js
    - vue.esm-browser.prod.js
    - vue.runtime.esm-browser.js
    - vue.runtime.esm-browser.prod.js
  - bundler（未打包，需要配合打包工具）
    - vue.esm-bundler.js
    - vue.runtime.esm-bundler.js

#### 3. Composition API

- 官方文档
  - RFC（Request For Comments）
    - https://github.com/vuejs/rfcs
  - Composition API RFC
    - https://composition-api.vuejs.org
- 设计动机
  - Option API
    - 包含一个描述组件选项（data，methods，props等）的对象
    - Option API 开发复杂组件，同一个功能逻辑的代码被拆分到不同选项
  - Compositon API
    - Vue 3.0 新增的一组 API
    - 一组基于函数的 API
    - 可以更灵活的组织组件的逻辑

#### 4. 性能提升

- 响应式系统升级
  - Vue 2.x 中响应式系统的核心 defineProperty
  - Vue 3.0 中使用 Proxy 对象重写响应式系统
    - 可以监听动态新增属性
    - 可以监听删除属性
    - 可以监听数组的索引和 length 属性
- 编译优化
  - Vue 2.x 中通过标记静态根节点，优化 diff 过程
  - Vue 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容
    - Fragments（升级 vetur 插件）
    - 静态提升
    - Patch flag
    - 缓存事件处理函数
- 源码体积的优化
  - Vue 3.0 中移除了一些不常用的 API
    - 例如：inline-template，filter 等
  - Tree-shaking
    - Vue 3.0 对 Tree-shaking 的支持更好，Tree-shaking 依赖 ES Module，通过编译阶段的静态分析，找到没有引入的模块，在打包的时候过滤掉，让打包后的体积更小

#### 5. Vite

- Vite 和 Vue Cli 的区别
  - 运行方式
    - Vite 在开发模式下不需要打包可以直接运行
    - Vue Cli 开发模式下必须对项目打包才可以运行
  - 打包方式
    - Vite 在生产环境下使用 rollup 打包
      - 基于 ES Module 的打包方式
    - Vue Cli 使用 webpack 打包
- 优点
  - 快速冷启动
  - 按需编译
  - 模块热更新
- Vite 的使用
  - Vite 创建项目
    - 使用 Vite 创建项目
      - npm init vite-app <project-name>
      - cd <project-name>
      - npm install
      - npm run dev
    - 基于模板创建项目
      - npm init vite-app --template react
      - npm init vite-app --template preact

### 二. Composition API

#### 1. 基本使用

- 新建目录，mkdir my-composition-api

- 初始化项目，npm init

- 安装 vue3.0，npm install vue@3.0

- 新建 createApp.html

  - Options API 写法

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        <p>x: {{ position.x }}</p>
        <p>y: {{ position.y }}</p>
      </div>
      <script type="module">
        import { createApp } from './node_modules/vue/dist/vue.esm-browser.js'
    
        // 创建一个 vue 对象，可以接收一个选项（组件选项，例如：data, methods, computed）作为参数
        // 返回一个 vue 对象
        const app = createApp({
          // data 不支持对象的写法，只支持函数
          data () {
            return {
              position: {
                x: 0,
                y: 0
              }
            }
          }
        })
        console.log(app)
        app.mount('#app')
      </script>
    </body>
    </html>
    ```

  - composition api 写法

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        <p>x: {{ position.x }}</p>
        <p>y: {{ position.y }}</p>
      </div>
      <script type="module">
        import { createApp, reactive } from './node_modules/vue/dist/vue.esm-browser.js'
    
        // 创建一个 vue 对象，可以接收一个选项（组件选项，例如：data, methods, computed）作为参数
        // 返回一个 vue 对象
        const app = createApp({
          // composition api 的入口函数
          // setup 有两个参数
          // 1. props：一个响应式对象，接收外部传入的参数，不能被解构
          // 2. context：一个对象，具有三个成员：attrs，emit，slots
          // setup 需要返回一个对象，这个对象可以使用在模板，methods，computed，生命周期的钩子函数中
          // setup 在 props 解析完毕，在组件实例创建之前执行，所以在 setup 内部无法通过 this 获取组件实例
          setup () {
            // reactive 函数是 vue3.0 中新增的，作用是创建响应式对象
            const position = reactive({
              x: 0,
              y: 0
            })
            return {
              position
            }
          },
          mounted () {
            this.position.x = 100
          }
        })
        console.log(app)
        app.mount('#app')
      </script>
    </body>
    </html>
    ```

#### 2. 生命周期钩子函数

| Options API           | Hook inside inside setup |
| --------------------- | ------------------------ |
| beforeCreate          | Not needed               |
| created               | Not needed               |
| beforeMount           | onBeforeMount            |
| mounted               | onMounted                |
| beforeUpdate          | onBeforeUpdate           |
| uodated               | onUpdated                |
| beforeUnmount         | onBeforeUnmount          |
| unmounted （distory） | onUnmounted              |
| errorCaptured         | onErrorCaptured          |
| renderTracked         | onRenderTracked          |
| renderTiggered        | onRenderTiggered         |

#### 3. reactive，toRefs，ref

- 这三个函数都是创建响应式数据的函数
- reactive 创建的响应式数据是不能以解构的形式来处理的，因为 reactive 函数返回一个 Proxy 对象，如果使用解构语法将属性解构出来的话，只是将属性的值复制了一份给解构变量，从而不会有响应式的效果
- toRefs 函数可以把响应式对象中的所有属性转换为响应式的
  - 原理
    - 传入的参数必须是一个代理对象
    - 如果传入的参数不是代理对象，则会报警告，提示需要传入一个代理对象
    - 内部会创建一个新的对象，遍历代理对象的所有属性，把所有属性都转换为响应式对象，最终返回新对象
- ref
  - 响应式 api
  - 作用是把数据转换为响应式对象
  - 如果传入参数为对象，则内部会调用 reactive 函数，返回一个代理对象，如果传入基本类型数据，内部会创建一个只有 value 属性的对象，这个 value 属性具有 getter 和 setter，在 getter 中收集依赖，在 setter 触发更新

#### 4. computed

- 第一种用法

  - computed(() => count.value + 1)

- 第二种用法

  ```
  const count = ref(1)
  const plusOne = computed({
  	get: () => count.value + 1,
  	set: val => {
  		count.value = val - 1
  	}
  })
  ```

#### 5. watch

- watch 函数的三个参数
  - 第一个参数：要监听的数据
  - 第二个参数：监听到数据变化后执行的函数，这个函数有两个参数，分别是新值和旧值
  - 第三个参数：选项对象，deep 和 immediate
- watch 的返回值
  - 取消监听的函数

#### 6. watchEffect

- 是 watch 的简化版本，也用来监听数据的变化
- 接收一个函数作为参数，监听函数内部响应式数据的变化
- 返回一个取消监听的函数

### 三. Vue.js 3.0 响应式系统原理

#### 1. 介绍

- Proxy 对象实现属性监听
- 多层嵌套属性，在访问属性过程中处理下一级属性
- 默认监听动态添加的属性
- 默认监听属性的删除操作
- 默认监听数组索引和 length 属性
- 可以作为单独的模块使用
- 核心方法
  - reactive/ref/toRefs/computed（响应式函数）
  - effect（watch 和 watchEffect 底层函数）
  - track（收集依赖函数）
  - trigger（触发更新函数）

#### 2. reactive

- 接收一个参数，判断这个参数是否是对象
- 创建拦截器对象 handler，设置 get/set/deleteProperty
- 返回 Proxy 对象

#### 3. 收集依赖

- 在依赖收集的过程中会创建三个集合（targetMap，depsMap，dep），
- targetMap
  - 用来记录目标对象和一个字典（depsMap）
  - 通过 new WeakMap() 创建
- depsMap
  - 用来记录目标对象的属性和属性值（dep：set 集合）
  - 通过 new Map() 创建
- dep
  - 用来记录 effect 函数
  - 通过 new Set() 创建

#### 4. 响应式源码

```
const isObject = value => value !== null && typeof value === 'object'

const convert = target => isObject(target) ? reactive(target) : target

const hasOwnProperty = Object.hasOwnProperty

const hasOwn = (target, key) => hasOwnProperty.call(target, key)

export function reactive (target) {
  if (!isObject(target)) return target
  const handler = {
    get (target, key, receiver) {
      // 收集依赖
      track(target, key)
      const result = Reflect.get(target, key, receiver)
      return convert(result)
    },
    set (target, key, value, receiver) {
      const oldValue = Reflect.get(target, value, receiver)
      let result = true
      if (oldValue !== value) {
        result = Reflect.set(target, key, value, receiver)
        // 触发更新
        trigger(target, key)
      }
      return result
    },
    deleteProperty (target, key) {
      const hasKey = hasOwn(target, key)
      const result = Reflect.deleteProperty(target, key)
      if (hasKey && result) {
        // 触发更新
        trigger(target, key)
      }
      return result
    }
  }
  return new Proxy(target, handler)
}

// 记录 callback
let activeEffect = null

export function effect (callback) {
  activeEffect = callback
  callback() // 访问响应式对象属性，去收集依赖
  activeEffect = null
}

let targetMap = new WeakMap()

export function track (target, key) {
  if (!activeEffect) return
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  dep.add(activeEffect)
}

export function trigger (target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  const dep = depsMap.get(key)
  if (dep) {
    dep.forEach(effect => {
      effect()
    })
  }
}

export function ref (raw) {
  // 判断 raw 是否为 ref 创建的对象，如果是的话直接返回
  if (isObject(raw) && raw.__v_isRef) return
  let value = convert(raw)
  const r = {
    __v_isRef: true,
    get value () {
      track(r, 'value')
      return value
    },
    set value (newValue) {
      if (newValue !== value) {
        raw = newValue
        value = convert(raw)
        trigger(r, 'value')
      }
    }
  }
  return r
}

export function toRefs (proxy) {
  const ret = proxy instanceof Array ? new Array(proxy.length) : {}
  for (const key in proxy) {
    ret[key] = toProxyRef(proxy, key)
  }
  return ret
}

function toProxyRef (proxy, key) {
  const r = {
    __v_isRef: true,
    get value () {
      return proxy[key]
    },
    set value (newValue) {
      proxy[key] = newValue
    }
  }
  return r
}

export function computed (getter) {
  const result = ref()
  effect(() => (result.value = getter()))
  return result
}
```

### 四. Vite

#### 1. 介绍

- Vite 是一个面向现代浏览器的一个更轻，更快的 Web 应用开发工具
- 它基于 ECMAScript 标准原生模块系统（ES Module）实现
- 作用
  - 解决 webpack 在开发阶段使用 webpack dev server 冷启动时间长
  - webpack HMR 热更新反应速度慢
- 特性
  - 快速冷启动
  - 模块热更新
  - 按需编译
  - 开箱即用
- Vite 项目依赖
  - Vite
  - @vue/compiler-sfc
- 基础使用
  - vite serve 用于开启一个开发服务器
  - vite build 用于打包代码
- Vite VS vue-cli
  - Vite
    - 在运行 vite serve 时，不需要打包，直接开启一个 web 服务器，当浏览器请求服务器，例如请求一个单文件组件，会在服务器端编译单文件组件，然后把编译的结果返回给浏览器
    - 注意
      - 文件编译是在服务器端
      - 模块的处理是在请求到服务器端处理
  - vue-cli
    - 当运行 vue-cli-service serve 时，内部会使用 Webpack 首先打包所有的模块，如果模块比较多，打包的速度会非常慢，把打包的结果存储到内存中，然后开启 web 服务器，浏览器请求 web 服务器，服务器把打包的结果直接返回给浏览器
- Vite HMR VS Webpack HMR
  - Vite HMR
    - 立即编译当前所修改文件
  - Webpack HMR
    - 会自动以这个文件为入口重新 build 一次，所有涉及到的依赖都会加载一次

#### 2. 原理

- 新建项目 my-vite-cli

- 初始化项目，npm init

- 安装项目依赖，npm i koa koa-send @vue/compiler-sfc

- 在项目根目录创建 index.js

  ```
  #!/usr/bin/env node
  
  const path = require('path')
  const { Readable } = require('stream')
  const Koa = require('koa')
  const send = require('koa-send')
  const compilerSFC = require('@vue/compiler-sfc')
  
  // 新建 koa 实例
  const app = new Koa()
  
  // 将文件流转换为字符串
  const streamToString = stream => new Promise((resolve, reject) => {
    const chunks = []
    stream.on('data', chunk => chunks.push(chunk))
    stream.on('end', () => resolve(Buffer.concat(chunks).toString('utf-8')))
    stream.on('error', reject)
  })
  
  const stringToStream = text => {
    const stream = new Readable()
    stream.push(text)
    stream.push(null)
    return stream
  }
  
  // 使用 Koa 开发静态 Web 服务器，默认返回根目录的 index.html
  // 创建中间件，负责处理静态文件，默认运行命令行运行目录下的 index.html
  
  // 3. 加载第三方模块
  app.use(async (ctx, next) => {
    if (ctx.path.startsWith('/@modules/')) {
      const moduleName = ctx.path.substr(10)
      const packagePath = path.join(process.cwd(), 'node_modules', moduleName, 'package.json')
      const package = require(packagePath)
      ctx.path = path.join('/node_modules', moduleName, package.module)
    }
    await next()
  })
  
  // 1. 静态 Web 服务器
  app.use(async (ctx, next) => {
    await send(ctx, ctx.path, { root: process.cwd(), index: 'index.html' })
    await next()
  })
  
  // 4. 处理单文件组件
  app.use(async (ctx, next) => {
    if (ctx.path.endsWith('.vue')) {
      const contents = await streamToString(ctx.body)
      const { descriptor } = compilerSFC.parse(contents)
      let code
      // 单文件组件的第一次请求
      if (!ctx.query.type) {
        code = descriptor.script.content
        // console.log(code)
        code = code.replace(/export\s+default\s+/g, 'const __script = ')
        code += `
          import { render as __render } from '${ctx.path}?type=template'
          __script.render = __render
          export default __script
        `
      } else if (ctx.query.type === 'template') {
        // 单文件组件的第二次请求
        const templateRender = compilerSFC.compileTemplate({ source: descriptor.template.content })
        code = templateRender.code
      }
      ctx.type = 'application/javascript'
      ctx.body = stringToStream(code)
    }
    await next()
  })
  
  // 2. 修改第三方模块的路径
  app.use(async (ctx, next) => {
    if (ctx.type === 'application/javascript') {
      const contents = await streamToString(ctx.body)
      ctx.body = contents.replace(/(from\s+['"])(?![\.\/])/g, '$1/@modules/').replace(/process\.env\.NODE_ENV/g, '"development"')
    }
  })
  
  app.listen(3000)
  
  console.log('server running @ http://localhost:3000')
  
  
  ```

  

- 在 package.json 文件中配置 bin 字段，值为 'index.js'