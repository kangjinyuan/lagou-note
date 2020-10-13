### 一. Vue-Router

------

#### 1. 使用步骤

- 创建路由相关的组件（views）

  ```
  
  <template>
    <div>
      这里是首页
    </div>
  </template>
  <script>
  export default {
    name: 'Index'
  }
  </script>
  ```

  ```
  <template>
    <div>
      这是 Bolg 页面
    </div>
  </template>
  
  <script>
  export default {
    name: 'Blog'
  }
  </script>
  ```

  ```
  <template>
    <div>
      这是 Photo 页面
    </div>
  </template>
  <script>
  export default {
    name:'Photo'
  }
  </script>
  ```

  

- 注册路由插件，定义路由规则，创建路由对象

  ```
  import Vue from 'vue'
  import VueRouter from 'vue-router'
  import Index from '../views/Index.vue'
  
  // 注册路由组件
  Vue.use(VueRouter)
  
  // 路由规则
  const routes = [{
          path: '/',
          name: 'Index',
          component: Index
      },
      {
          path: '/blog',
          name: 'Blog',
          // route level code-splitting
          // this generates a separate chunk (about.[hash].js) for this route
          // which is lazy-loaded when the route is visited.
          component: () =>
              import ( /* webpackChunkName: "blog" */ '../views/Blog.vue')
      }, {
          path: '/photo',
          name: 'Photo',
          // route level code-splitting
          // this generates a separate chunk (about.[hash].js) for this route
          // which is lazy-loaded when the route is visited.
          component: () =>
              import ( /* webpackChunkName: "photo" */ '../views/Photo.vue')
      }
  ]
  
  //创建路由对象
  const router = new VueRouter({
      routes
  })
  
  // 导出路由对象
  export default router
  ```

  

- 在vue实例中创建router对象

  ```
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  
  Vue.config.productionTip = false
  
  new Vue({
      // 注册路由对象
      router,
      render: h => h(App)
  }).$mount('#app')
  ```

  

- 通过 <router-view/> 来占位

  ```
  <!-- 路由组件的占位 -->
  <router-view/>
  ```

  

- 通过 <router-link to="路由地址"></router-link> 跳转链接

  ```
  <router-link to="/">Home</router-link> |
  <router-link to="/blog">Blog</router-link> | 
  <router-link to="/photo">Photo</router-link>
  ```

  

#### 2. 动态路由

- 通过占位符来匹配变化的链接

  ```
  {
          path: '/detail/:id',
          name: 'Detail',
          // route level code-splitting
          // this generates a separate chunk (about.[hash].js) for this route
          // which is lazy-loaded when the route is visited.
          component: () =>
              import ( /* webpackChunkName: "detail" */ '../views/Detail.vue')
      }
  ```

  

#### 3. 嵌套路由

```
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'
import Index from '../views/Index.vue'

// 注册路由组件
Vue.use(VueRouter)

// 路由规则
const routes = [{
    path: '/',
    component: Home,
    children: [{
        path: '/index',
        name: 'Index',
        component: Index
    }, {
        path: '/blog',
        name: 'Blog',
        component: import ( /* webpackChunkName: 'blog' */ '../views/Blog.vue')
    }, {
        path: '/photo',
        name: 'Photo',
        component: import ( /* webpackChunkName: 'photo' */ '../views/Photo.vue')
    }]
}]

//创建路由对象
const router = new VueRouter({
    routes
})

// 导出路由对象
export default router
```

#### 4. 编程式导航

```
this.$router.push('/') // 跳转 路径为'/'的页面
this.$router.push({name: 'Home'}) // 跳转 'Home'页面
this.$router.push({name: 'Home', params: {id: 1}}) // push方法传递路由参数
this.$router.replace('/login') // 跳转登录，replace方法不会记录历史页面
this.$router.go(-1) // go方法跳转历史页面
```

#### 5. Hash模式和History模式

- url区别

  - Hash模式：https://music.163.com/#/playlist?id=3102961863
  - History模式：https://music.163.com/playlist/3102961863

- 原理区别

  - Hash模式是基于锚点，以及onhashchange事件，通过锚点的值作为路由地址，当地址发生变化后，触发onhashchange事件，然后根据地址找到对应的组件重新渲染

  - History模式是基于HTML5中的History API

    - history.pushState() （IE10 以后才支持）通过此方法改变地址栏，注意：该方法不会向服务器发送请求，只是改变地址栏

    - 监听popState()，监听浏览器历史发生的变化，根据地址找到对应的组件重新渲染，注意：popState的触发条件是点击浏览器前进或后退按钮，或者是调用history的back或forword方法时

    - history.replaceState()

    - history模式需要服务器的支持

      - node服务器配置history

        ```
        const path = require('path')
        // 导入处理 history 模式的模块
        const history = require('connect-history-api-fallback')
        // 导入 express
        const express = require('express')
        
        const app = express()
        // 注册处理 history 模式的中间件
        app.use(history())
        // 处理静态资源的中间件，网站根目录 ../web
        app.use(express.static(path.join(__dirname, '../web')))
        
        // 开启服务器，端口是 3000
        app.listen(3000, () => {
          console.log('服务器开启，端口：3000')
        })
        ```

        

      - nginx服务器配置history

        - 修改nginx中，conf目录下的nginx.conf文件

          ```
          location / {
          	root html;
          	index index.html index.htm;
          	// 尝试请求文件
          	try_files $uri $uri/ /index.html;
          }
          ```

          

    - 单页面应用中，服务端不存在http://www.testurl.com/login 这样的地址会返回找不到该页面（404）

    - 在服务端应该除了静态资源外都返回单页面应用的index.html

#### 6. history模式VueRouter

```
let _Vue = null
export default class VueRouter {
    // Vue.use()调用install方法时，会传递两个参数，一个是Vue的构造函数，另一个是可选的选项对象
    static install(Vue) {
        // 1. 判断当前插件是否已经被安装
        if (VueRouter.install.installed) {
            return
        }
        VueRouter.install.installed = true
            // 2. 把Vue构造函数记录到全局变量
        _Vue = Vue
            // 3. 把创建Vue实例时传入的router对象注入到Vue实例上
        _Vue.mixin({
            beforeCreate() {
                if (this.$options.router) {
                    _Vue.prototype.$router = this.$options.router
                    this.$options.router.init()
                }
            }
        })
    }

    constructor(options) {
        // 记录构造函数中传入的选项 options
        this.options = options
            // 解析options中的路由规则，key为路由地址，value为路由组件，在<router-view>组件中根据当前的路由
            // 地址来routeMap中找到对应的组件将其渲染到浏览器
        this.routeMap = {}
            // 响应式的对象，利用Vue提供的observable方法将其变为响应式
        this.data = _Vue.observable({
            // 记录当前路由地址，默认是 '/'
            current: '/'
        })
    }

    init() {
        this.createRouteMap()
        this.initComponents(_Vue)
        this.initEvent()
    }

    createRouteMap() {
        // 遍历所有的路由规则，把路由规则解析成键值对的形式，存储到routeMap中
        this.options.routes.forEach(route => {
            this.routeMap[route.path] = route.component
        })
    }

    // Vue这个参数的作用，为了减少initComponents这个方法和外界的依赖
    initComponents(Vue) {
        Vue.component('router-link', {
            props: {
                to: String
            },
            render(h) {
                return h('a', {
                    attrs: {
                        href: this.to
                    },
                    on: {
                        click: this.headleClick
                    }
                }, [this.$slots.default])
            },
            methods: {
                headleClick(e) {
                    history.pushState({}, '', this.to)
                    this.$router.data.current = this.to
                    e.preventDefault()
                }
            }
        })
        const self = this
        Vue.component('router-view', {
            render(h) {
                const component = self.routeMap[self.data.current]
                return h(component)
            }
        })
    }

    initEvent() {
        window.addEventListener('popstate', () => {
            this.data.current = window.location.pathname
        })
    }
}
```

#### 7. Hash模式VueRouter

```
let _Vue = null
export default class VueRouter {
    static install(Vue) {
        if (VueRouter.install.installed) {
            return
        }
        VueRouter.install.installed = true
        _Vue = Vue
        _Vue.mixin({
            beforeCreate() {
                if (this.$options.router) {
                    _Vue.prototype.router = this.$options.router
                    this.$options.router.init()
                }
            }
        })
    }
    constructor(options) {
        this.options = options
        this.routeMap = {}
        this.data = _Vue.observable({
            current: window.location.pathname
        })
    }

    init() {
        this.createRouteMap()
        this.initComponents(_Vue)
        this.initEvent()
    }

    createRouteMap() {
        this.options.routes.forEach(route => {
            this.routeMap[route.path] = route.component
        })
    }

    initComponents(Vue) {
        Vue.component('router-link', {
            props: {
                to: String
            },
            render(h) {
                return h('a', {
                    attrs: {
                        href: '#' + this.to
                    },
                    on: {
                        click: this.handleClick
                    }
                }, [this.$slots.default])
            },
            methods: {
                handleClick(e) {
                    window.location.hash = '#' + this.to
                    this.$router.data.current = this.to
                    e.preventDefault()
                }
            }
        })
        const self = this
        Vue.component('router-view', {
            render(h) {
                const component = self.routeMap[self.data.current]
                return h(component)
            }
        })
    }

    initEvent() {
        window.addEventListener('load', this.hashChange.bind(this))
        window.addEventListener('hashchange', this.hashChange.bind(this))
    }

    hashChange() {
        if (!window.location.hash) {
            window.location.hash = '#/'
        }
        this.data.current = window.location.hash.substr(1)
    }

}
```

### 二. Vue 响应式

#### 1. 数据驱动

- Vue最独特的特性之一
- 开发过程中仅需要关注数据本身，不需要关心数据是如何渲染到视图

#### 2. 数据响应式

- 数据模型仅仅是普通的 JavaScript 对象，当我们修改数据时，视图会进行更新，避免了繁琐的DOM操作，提高开发效率

- Vue2.X数据响应式

  - 一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，Vue 将遍历此对象所有的属性，并使用  Object.defineProperty 把这些属性全部转为 getter/setter 。Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也是 Vue 不支持 IE8 以及更低版本浏览器的原因 

    ```
    // 模拟 Vue 中的 data 选项
        let data = {
            msg: 'hello',
            count: 20
        }
    
        // 模拟 Vue 的实例
        let vm = {}
    
        // 数据劫持：当访问或者设置 vm 中的成员的时候，做一些干预操作
        Object.defineProperty(vm, 'msg', {
            // 可枚举（可遍历）
            enumerable: true,
            // 可配置（可以使用 delete 删除，可以通过 defineProperty 重新定义）
            configurable: true,
            // 当获取值的时候执行
            get() {
                console.log('get: ', data.msg)
                return data.msg
            },
            // 当设置值的时候执行
            set(newValue) {
                console.log('set: ', newValue)
                if (newValue === data.msg) {
                    return
                }
                data.msg = newValue
                    // 数据更改，更新 Dom 的值
                document.getElementById('app').textContent = data.msg
            }
        })
    
        vm.msg = 'Hello World'
        console.log(vm.msg)
    ```

    

- Vue3.X数据响应式

  - 通过 Proxy 这个对象，直接监听对象，而非属性，这个对象是 ES6 新增的，IE不支持，性能由浏览器优化

    ```
    let data = {
            msg: 'hello',
            count: 0
        }
    
        let vm = new Proxy(data, {
            // 执行代理行为的函数
            // 当访问 vm 的成员会执行
            get(target, key) {
                console.log('get, key: ', key, target[key])
                return target[key]
            },
            // 当设置 vm 的成员会执行
            set(target, key, newValue) {
                console.log('set, key: ', key, newValue)
                if (target[key] === newValue) {
                    return
                }
                target[key] = newValue
                document.getElementById('app').textContent = target[key]
            }
        })
    
        vm.msg = 'Hello World'
        console.log(vm.msg)
    ```

    

#### 3. 双向数据绑定

- 数据改变，视图改变；视图改变，数据改变
- 我们可以使用 v-model 在表单元素上创建双向数据绑定

#### 4. 发布订阅模式

- 订阅者

- 发布者

- 事件中心

  ```
  // 事件触发器
      class EventEmitter {
          constructor() {
                  // 记录所有的事件及处理函数
                  // { 'click': [fn1, fn2], 'change': [fn] }
                  this.subs = Object.create(null)
              }
              // 注册事件
          $on(eventType, handler) {
              this.subs[eventType] = this.subs[eventType] || []
              this.subs[eventType].push(handler)
          }
  
          // 触发事件
          $emit(eventType) {
              if (this.subs[eventType]) {
                  this.subs[eventType].forEach(handler => {
                      handler()
                  })
              }
          }
      }
  
      let em = new EventEmitter()
  
      em.$on('click', () => {
          console.log('click1')
      })
      em.$on('click', () => {
          console.log('click2')
      })
      em.$emit('click')
  ```

#### 5. 观察者模式

- 观察者（订阅者）-- Watcher

  - update()：当事件发生时，具体要做的事情

- 目标（发布者）-- Dep

  - subs 数组：存储所有的观察者

  - addSub()：添加观察者

  - notify()：当时间发生时，调用所有观察者的 update() 方法

    ```
    // 发布者-目标
        class Dep {
            constructor() {
                this.subs = []
            }
    
            addSub(sub) {
                if (sub && sub.update) {
                    this.subs.push(sub)
                }
            }
    
            notify() {
                this.subs.forEach(sub => {
                    sub.update()
                })
            }
        }
    
        // 订阅者-观察者
        class Watcher {
            update() {
                console.log('update')
            }
        }
    
        let dep = new Dep()
    
        let watcher = new Watcher()
    
        dep.addSub(watcher)
    
        dep.notify()
    ```

    

- 没有事件中心

#### 6. 观察者模式和发布订阅模式的区别

- **观察者模式**是由具体目标调度，比如当事件触发时，Dep就会取调用观察者的方法，所以观察者模式的订阅者与发布者之间是存在依赖的
- **发布/订阅模式**由统一的调度中心调用，因此发布者和订阅者不需要知道对方的存在

#### 7. Vue 响应式原理

- 基础结构

  - Vue 把 data 对象中的成员注入到 Vue 实例中，并且把 data 中的成员转成 getter/setter

    - 负责接收初始化参数（选项）

    - 负责把 data 中的属性注入到 Vue 实例中，并转换成 getter/setter

      ```
      class Vue {
          constructor(options) {
              // 1. 通过属性保存选项中的数据
              // 传入的参数
              this.$options = options || {}
                  // 传入的 data 对象
              this.$data = options.data || {}
                  // 如果 el 是字符串，找到该dom对象，如果是dom对象，直接赋值
              this.$el = typeof options.el === 'string' ? document.querySelector(options.el) : options.el
                  // 2. 把 data 中的成员转换成 getter/setter ，注入到 Vue 实例中
              this._proxyData(this.$data)
                  // 3. 调用 Observer 对象，监听数据的变化
              new Observer(this.$data)
                  // 4. 调用 Compiler 对象，解析指令和插值表达式
              new Compiler(this)
          }
      
          _proxyData(data) {
              // 遍历 data 中的所有属性
              Object.keys(data).forEach(key => {
                  // 把 data 中属性注入到 Vue 的实例中
                  Object.defineProperty(this, key, {
                      enumerable: true,
                      configurable: true,
                      get() {
                          return data[key]
                      },
                      set(newValue) {
                          if (newValue === data[key]) {
                              return
                          }
                          data[key] = newValue
                      }
                  })
              })
          }
      }
      ```

      

  - Observer 能够对数据对象的所有属性进行监听，如有变动可拿到最新的值并通知Dep

    - 负责把 data 选项中的属性转换成响应式数据

    - data 中的某个属性也是对象，把该属性转换成响应式数据

    - 数据变化发送通知

      ```
      // 数据劫持
      class Observer {
          constructor(data) {
                  this.walk(data)
              }
              // 遍历对象的所有属性
          walk(data) {
              // 1. 判断 data 是否是空或者对象
              if (!data || typeof data !== 'object') {
                  return
              }
              // 2. 遍历 data 对象的所有属性
              Object.keys(data).forEach(key => {
                  this.defineReactive(data, key, data[key])
              })
          }
      
          // 定义响应式数据
          // 传 val 参数的作用
          // 1. 避免发送获取属性值时发生死递归
          // 2. 利用闭包扩展了 val 变量的作用域
          defineReactive(obj, key, val) {
              const that = this
              let dep = new Dep()
                  // 如果 val 是对象，把 val 内部的属性转换成响应式数据
              this.walk(val)
              Object.defineProperty(obj, key, {
                  enumerable: true,
                  configurable: true,
                  get() {
                      // 收集依赖
                      Dep.target && dep.addSub(Dep.target)
                      return val
                  },
                  set(newValue) {
                      if (newValue === val) {
                          return
                      }
                      val = newValue
                          // 如果 newValue 是一个对象，把 newValue 内部的属性转换成响应式数据
                      that.walk(newValue)
                          // 发送通知
                      dep.notify()
                  }
              })
          }
      }
      ```

      

  - Compiler 解析每个元素中的指令以及插值表达式，并替换成相应的数据

    - 负责编译模板，解析指令/插值表达式

    - 负责页面的首次渲染

    - 当数据变化后重新渲染视图

      ```
      class Compiler {
          constructor(vm) {
              this.el = vm.$el
              this.vm = vm
              this.compile(this.el)
          }
      
          // 编译模板，处理文本节点和元素节点
          compile(el) {
              let childNodes = el.childNodes
              Array.from(childNodes).forEach(node => {
                  // 处理文本节点
                  if (this.isTextNode(node)) {
                      this.compileText(node)
                  } else if (this.isElementNode(node)) { // 处理元素节点
                      this.compileElement(node)
                  }
      
                  // 判断 node 节点是否有字节点，如果有子节点，要递归调用 compile
                  if (node.childNodes && node.childNodes.length > 0) {
                      this.compile(node)
                  }
              })
          }
      
          // 编译元素节点，处理指令
          compileElement(node) {
              // 遍历所有的属性节点
              Array.from(node.attributes).forEach(attr => {
                  // 判断是否是指令
                  let attrName = attr.name
                  if (this.isDirective(attrName)) {
                      attrName = attrName.substr(2)
                      let key = attr.value
                      this.update(node, key, attrName)
                  }
              })
          }
      
          update(node, key, attrName) {
              let updateFn = this[attrName + 'Updater']
              updateFn && updateFn.call(this, node, key, this.vm[key])
          }
      
          // 处理 v-text 指令
          textUpdater(node, key, value) {
              node.textContent = value
              new Watcher(this.vm, key, newValue => {
                  node.textContent = newValue
              })
          }
      
          // 处理 v-model 指令
          modelUpdater(node, key, value) {
              node.value = value
              new Watcher(this.vm, key, newValue => {
                      node.value = newValue
                  })
                  // 双向绑定
              node.addEventListener('input', () => {
                  this.vm[key] = node.value
              })
          }
      
          // 编译文本节点，处理插值表达式
          compileText(node) {
              let reg = /\{\{(.+?)\}\}/
              let value = node.textContent
              if (reg.test(value)) {
                  let key = RegExp.$1.trim()
                  node.textContent = value.replace(reg, this.vm[key])
                      // 创建 Watcher 对象，当数据改变更新视图
                  new Watcher(this.vm, key, newValue => {
                      node.textContent = newValue
                  })
              }
          }
      
          // 判断元素属性名字是否是指令
          isDirective(attrName) {
              return attrName.startsWith('v-')
          }
      
          // 判断节点是否是文本节点
          isTextNode(node) {
              return node.nodeType === 3
          }
      
          // 判断节点是否是元素节点
          isElementNode(node) {
              return node.nodeType === 1
          }
      }
      ```

      

  - Watcher 观察者（订阅者）

    - 当数据变化触发依赖，dep 通知所有的 Watcher 实现更新视图

    - 自身实例化的时候往 dep 对象中添加自己

      ```
      class Watcher {
          constructor(vm, key, cb) {
              this.vm = vm
                  // data 中的属性名称
              this.key = key
                  // 回调函数负责更新视图
              this.cb = cb
                  // 把 Watcher 对象记录到 Dep 类的静态属性 target 中
              Dep.target = this
                  // 触发 get 方法，在 get 方法中会调用 addSub
              this.oldValue = vm[key]
                  // 防止重复添加
              Dep.target = null
          }
      
          // 当数据发生变化时更新视图
          update() {
              let newValue = this.vm[this.key]
              if (this.oldValue === newValue) {
                  return
              }
              this.cb(newValue)
          }
      }
      ```

      

  - Dep 发布者（目标）

    - 收集 data 对象中的依赖（Watcher）

    - 发布通知，触发所有的 Watcher 的 update 方法

      ```
      class Dep {
          constructor() {
              // 储存所有的观察者
              this.subs = []
          }
      
          // 添加观察者
          addSub(sub) {
              if (sub && sub.update) {
                  this.subs.push(sub)
              }
          }
      
          // 发送通知
          notify() {
              this.subs.forEach(sub => {
                  sub.update()
              })
          }
      }
      ```


### 三. Virtual Dom

#### 1. Virtual Dom 概念

- 什么是 Virtual Dom
  - 由普通 JS 对象来描述 Dom 对象，因为不是真实的 Dom 对象，所以叫 Virtual Dom
  - Virtual Dom 相对于真实 Dom ，内置属性要少很多（创建开销要小）

#### 2. 为什么要使用 Virtual Dom

- 手动操作 Dom 比较麻烦，还需要考虑浏览器兼容性问题，虽然有 jQuery 等库简化 Dom 操作，但是随着项目的复杂，Dom 操作复杂提升

- 为了简化 Dom 的复杂操作，于是出现了各种 MVVM 框架，MVVM 框架解决了视图和状态的同步问题

- 为了简化视图的操作我们可以使用模板引擎，但是模板引擎没有解决跟踪状态变化的问题，于是 Virtual Dom 出现了

- Virtual Dom 的好处是当状态改变时不需要立即更新 Dom ，只需要创建一个虚拟树来描述 Dom，Virtual Dom 内部将弄清楚如何有效（diff）的更新 Dom

  参考 github 上 Virtual Dom 的描述

  - Virtual Dom 可以维护程序上的状态，跟踪上一次的状态
  - 通过比较前后两次状态的差异更新真实 Dom

#### 3. Virtual Dom 的作用

- 维护视图和状态的关系
- 复杂视图情况下提升渲染性能
- 除了渲染 Dom 之外，还可以实现 SSR（Nuxt.js/Next.js）,原生应用（Weex/React Native），小程序（mpvue/uni-app）等

#### 4. Virtual Dom 开源库

- Snabbdom
  - Vue 2.x 内部使用的 Virtual Dom 就是改造的 Snabbdom
  - 大约 200 SL SLOC （single line of code）
  - 通过模块可扩展
  - 源码使用 TypeScript 开发
  - 最快的 Virtual Dom 之一
- virtual-dom
  - 最早的 Virtual Dom 开源库

#### 5. 导入 snabbdom

- 安装 snabbdom

  ```
  yarn add snabbdom
  ```

- 导入 snabbdom

  ```
  import { init } from 'snabbdom/init'
  import { h } from 'snabbdom/h'
  ```

  如果遇到下面的错误

  Cannot resolve dependency 'snabbdom/init’

  因为模块路径并不是 snabbdom/int，这个路径是作者在 package.json 中的 exports 字段设置的，而我们使用的打包工具不支持 exports 这个字段，webpack 4 也不支持，webpack 5 beta 支持该字段。该字段在导入 snabbdom/init 的时候会补全路径成 snabbdom/build/package/init.js。

  ```
  {
    "exports": {
      "./init": "./build/package/init.js",
      "./h": "./build/package/h.js",
      "./helpers/attachto": "./build/package/helpers/attachto.js",
      "./hooks": "./build/package/hooks.js",
      "./htmldomapi": "./build/package/htmldomapi.js",
      "./is": "./build/package/is.js",
      "./jsx": "./build/package/jsx.js",
      "./modules/attributes": "./build/package/modules/attributes.js",
      "./modules/class": "./build/package/modules/class.js",
      "./modules/dataset": "./build/package/modules/dataset.js",
      "./modules/eventlisteners": "./build/package/modules/eventlisteners.js",
      "./modules/hero": "./build/package/modules/hero.js",
      "./modules/module": "./build/package/modules/module.js",
      "./modules/props": "./build/package/modules/props.js",
      "./modules/style": "./build/package/modules/style.js",
      "./thunk": "./build/package/thunk.js",
      "./tovnode": "./build/package/tovnode.js",
      "./vnode": "./build/package/vnode.js"
    }
  }
  ```

  **解决方法一**：安装 Snabbdom@v0.7.4 版本

  ```
  import { h, thunk, init } from 'snabbdom'
  ```

  **解决方法二**：导入 init、h，以及模块只要把把路径补全即可。

  ```
  import { h } from 'snabbdom/build/package/h'
  import { init } from 'snabbdom/build/package/init'
  import { classModule } from 'snabbdom/build/package/modules/class'
  ```

- snabbdom 的核心仅提供最基本的函数，只导出了三个函数 init(), h(), thunk()

  - init() 是一个高阶函数，返回 patch()

    - patch() 是 snabbdom 的核心函数
    - 作用：对比两个虚拟 VNode 的差异，把差异更新到真实 Dom

  - h() 返回虚拟节点 VNode，这个函数我们在使用 Vue.js 的时候见过

    ```
    new Vue({
    	router,
    	store,
    	render: h => h(App)
    }).$mount('#app')
    ```

  - thunk() 是一种优化策略，可以在处理不可变数据时使用

  - 代码演示

    ```
    import { h, init } from 'snabbdom'
    
    // 1. hello world
    // patch() 是 snabbdom 的核心函数
    // 作用：对比两个虚拟 VNode 的差异，把差异更新到真实 Dom
    let patch = init([])
        // 第一个参数：标签 + 选择器
        // 第二个参数： 如果是字符串的话就是标签中的内容，如果是子元素，则为数组，在数组中接着用 h()创建 vnode
    let vnode = h('div#container.cls', 'Hello World')
    let app = document.querySelector('#app')
        // 第一个参数：VNode，也可以是真实的 Dom，内部会把 Dom 元素转换为 VNode
        // 第二个参数：VNode
        // 返回值：VNode
    let oldVnode = patch(app, vnode)
    
    vnode = h('div', 'Hello Snabbdom')
    
    patch(oldVnode, vnode)
    ```

    ```
    import { h, init } from 'snabbdom'
    
    // 2. div 中放置子元素 h1,p
    let patch = init([])
    
    let vnode = h('div#container', [
        h('h1', 'Hello Snabbdom'),
        h('p', '这是一个 p 标签')
    ])
    
    let app = document.querySelector('#app')
    
    let oldVnode = patch(app, vnode)
    
    setTimeout(() => {
        vnode = h('div#container', [
            h('h1', 'Hello World'),
            h('p', 'Hello P')
        ])
        patch(oldVnode, vnode)
    
        // 清空页面元素
        // 官网中 patch(oldVnode,null) 是错误的，会报错
        // 正确的做法
        patch(oldVnode, h('!'))
    }, 2000)
    ```


#### 6. 模块

- snabbdom的核心库并不能处理元素的属性/样式/事件等，如果需要处理的话，可以使用模块
- 常用模块
  - attributes
    - 设置 Dom 元素的属性，使用 setAttribute()
    - 处理布尔类型的属性
  - props
    - 和 attributes 模块类似，设置 Dom 元素的属性 element[attr] = value
    - 不处理布尔类型的属性
  - class
    - 切换类样式
    - 注意：给元素设置类样式是通过 sel 选择器
  - dataset
    - 设置 data-* 的自定义属性
  - eventlisteners
    - 注册和移除事件
  - style
    - 设置行内样式，支持动画
    - delayed/remove/destory
- 模块使用
  - 导入需要的模块
  - init() 中注册模块
  - 使用 h() 函数创建 Vnode 的时候，可以把第二个参数设置为对象，其他参数往后移

#### 7. snabbdom的核心

- 使用 h() 函数创建 JavaScript 对象（VNode）描述真实 Dom，主要是调用 vnode() 函数返回 VNode 节点
- init() 设置模块，创建patch()
- patch() 比较新旧两个 VNode
- 把变化的内容更新到真实的 Dom 

#### 8. patch 的整体过程

- patch(oldVnode,vnode)
- 打补丁，把新节点中变化的内容渲染到真实 Dom，最后返回新节点作为下一次处理的旧节点
- 对比新旧 VNode 是否相同节点（节点的 key 和 sel 相同）
- 如果不是相同节点，删除之前的内容，重新渲染
- 如果是相同节点，再判断新的 vnode 是否有 text，如果有并且和 oldVnode 的 text 不同，直接更新文本内容
- 如果新的 VNode 有 children，判断子节点是否有变化，判断子节点的过程使用的就是 Diff 算法
- diff 过程只进行同层级比较