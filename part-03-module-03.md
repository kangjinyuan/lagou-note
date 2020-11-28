### 一. Vuex 状态管理

#### 1. 状态管理

- state 驱动应用的数据源
- view 以声明方式将 state 映射到视图
- actions 响应在 view 上的用户输入导致的状态变化

#### 2. 三种组件间通信方式

- 父组件给子组件传值

  - 子组件中通过 props 接收数据

  - 父组件中给子组件通过相应属性传值

  - 子组件

    ```
    <template>
      <div>
        <h1>Props Down Child</h1>
        <h2>{{ title }}</h2>
      </div>
    </template>
    <script>
    export default {
      props:{
        title: String
      }
    }
    </script>
    ```

  - 父组件

    ```
    <template>
      <div>
        <h1>Props Down Parent</h1>
        <child title="My journey with vue"></child>
      </div>
    </template>
    <script>
    import Child from './01-Child.vue'
    export default {
      components: {
        Child
      }
    }
    </script>
    ```

    

- 子组件给父组件传值

  - 子组件通过自定义事件进行传值， $emit(事件名，参数)

  - 父组件通过在子组件上注册子组件的自定义事件处理事件，@事件名=‘处理事件的方法’

  - 子组件

    ```
    <template>
      <div>
        <h1 :style="{ fontSize: fontSize + 'em' }">Props Down Child</h1>
        <button @click="handler">文字放大</button>
      </div>
    </template>
    <script>
    export default {
      props:{
        fontSize: String
      },
      methods:{
        handler(){
          this.$emit('enlargeText', 0.1)
        }
      }
    }
    </script>
    ```

  - 父组件

    ```
    <template>
      <div>
        <h1 :style="{ fontSize: hFontSize + 'em' }">Props Down Parent</h1>
    
        这里的文字不需要变化
    
        <child :fontSize="hFontSize" @enlargeText="enlargeText"></child>
        <child :fontSize="hFontSize" @enlargeText="enlargeText"></child>
        <child :fontSize="hFontSize" @enlargeText="hFontSize += $event"></child>
      </div>
    </template>
    <script>
    import Child from './02-Child.vue'
    export default {
      components:{
        Child
      },
      data(){
        return {
          hFontSize: 1
        }
      },
      methods:{
        enlargeText(size){
          this.hFontSize += size
        }
      }
    }
    </script>
    ```

- 不相关组件之间传值

  - 通过事件中心处理

    - eventBus

      ```
      import Vue from 'vue'
      export default new Vue()
      ```
    
  - 组件1通过自定义事件来传递参数
    
    ```
      <template>
        <div>
          <h1>Event Bus Sibling01</h1>
          <div class="number" @click="sub">-</div>
          <input type="text" style="width: 30px;text-align: center;" :value="value">
          <div class="number" @click="add">+</div>
        </div>
    </template>
      <script>
    import bus from './eventBus.js'
      export default {
      props:{
          num: Number
        },
        data(){
          return {
            value: -1
          }
        },
        methods:{
          sub(){
            if(this.value > 1){
              this.value--
              bus.$emit('num-change', this.value)
            }
          },
          add(){
            this.value++
            bus.$emit('num-change', this.value)
          }
        },
        created(){
          this.value = this.num
        }
      }
      </script>
      <style>
      .number {
        display: inline-block;
        cursor: pointer;
        width: 20px;
        text-align: center;
      }
      </style>
    ```
    
    - 组件2通过自定义事件来接收参数并处理事件过程
    
      ```
      <template>
        <div>
          <h1>Event Bus Sibling02</h1>
          <div>{{ msg }}</div>
        </div>
      </template>
      <script>
      import bus from './eventBus.js'
      export default {
        data(){
          return {
            msg: ''
          }
        },
        created(){
          bus.$on('num-change', value => {
            this.msg = `您选择了${value}件商品`
          })
        }
      }
      </script>
      ```


#### 4. 简易状态管理

- store

  ```
  export default {
      debug: true,
      state: {
          user: {
              name: 'kjy',
              age: 24,
              sex: '男'
          }
      },
      setUserNameAction(name) {
          if (this.debug) {
              console.log('setUserNameAction triggered: ', name)
          }
          this.state.user.name = name
      }
  }
  ```

- ComponentA.vue

  ```
  <template>
    <div>
      <h1>ComponentA</h1>
      user name: {{ sharedState.user.name }}
      <button @click="change">Change Info</button>
    </div>
  </template>
  <script>
  import store from './store.js'
  export default {
    data(){
      return {
        privateState:{},
        sharedState: store.state
      }
    },
    methods:{
      change(){
        store.setUserNameAction('ComponentA')
      }
    }
  }
  </script>
  ```

- ComponentB

  ```
  <template>
    <div>
      <h1>ComponentB</h1>
      user name: {{ sharedState.user.name }}
      <button @click="change">Change Info</button>
    </div>
  </template>
  <script>
  import store from './store.js'
  export default {
    data(){
      return {
        privateState:{},
        sharedState: store.state
      }
    },
    methods:{
      change(){
        store.setUserNameAction('ComponentB')
      }
    }
  }
  </script>
  ```

#### 3. Vuex

- 什么是 Vuex

  - Vuex 是专门为 Vue.js 设计的状态管理库
  - Vuex 采用集中式的方式存储需要共享的数据
  - Vuex 的作用是进行状态管理，解决复杂组件通信，数据共享
  - Vuex 集成到了 devtools 中，提供了 time-travel 时光旅行历史回滚功能

- 什么时候使用 Vuex

  - 非必要情况下不要使用 Vuex
  - 大型的单页应用
    - 多个视图依赖于同一状态
    - 来自不同视图的行为需要变更同一状态

- 核心概念

  - Store

    - Vuex 应用仓库的核心
    - 每一个应用仅有一个 Store
    - Store 是一个容器，包含应用中的大部分状态
    - 不能直接改变 Store 中的状态，需要通过提交 Mutation 的方式改变状态

  - State

    - 单一状态树

    - 应用的状态（共享数据）

    - State 中的状态是响应式的

    - 使用方式

      - 方式一

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0,
                message: 'Hello Vuex'
            },
            mutations: {},
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            Count: {{ $store.state.count }} <br/>
            Message: {{ $store.state.message }}
          </div>
        </template>
        
        <script>
        export default {
          name: 'App',
          components: {
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

      - 方式二

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0,
                message: 'Hello Vuex'
            },
            mutations: {},
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            Count: {{ count }} <br/>
            Message: {{ message }}
          </div>
        </template>
        
        <script>
        import { mapState } from 'vuex'
        export default {
          name: 'App',
          components: {
          },
          computed:{
            // count: state => state.count
            ...mapState(['count','message'])
            或
            ...mapState({ count: 'count', message: 'message' })
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

        

  - Getter

    - 类似于 computed，方便从一个属性中派生出其他的值

    - 内部可以给计算的结果进行缓存，当依赖的状态改变时，计算的值才会改变

    - 使用方式

      - 方式一

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0,
                message: 'Hello Vuex'
            },
            getters: {
                reverseMessage(state) {
                    return state.message.split('').reverse().join('')
                }
            },
            mutations: {},
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Getter</h2>
            reverseMessage: {{ $store.getters.reverseMessage }}
          </div>
        </template>
        
        <script>
        export default {
          name: 'App',
          components: {
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

      - 方式二

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0,
                message: 'Hello Vuex'
            },
            getters: {
                reverseMessage(state) {
                    return state.message.split('').reverse().join('')
                }
            },
            mutations: {},
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Getter</h2>
            reverseMessage: {{ reverseMsg }}
          </div>
        </template>
        
        <script>
        import { mapGetters } from 'vuex'
        export default {
          name: 'App',
          components: {
          },
          computed:{
            ...mapGetters(['reverseMessage'])
            或
            ...mapGetters({ reverseMsg: 'reverseMessage' })
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

        

  - Mutation

    - 状态的变化需要提交 Mutation 来完成

    - 不要再 Mutation 中加入异步的操作，否则再 devtools 中无法观测到数据的变化

    - 使用方式

      - 方式一

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0
            },
            mutations: {
                increate(state, payload) {
                    state.count += payload
                }
            },
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Mutation</h2>
            <button @click="$store.commit('increate', 2)">Mutation</button>
          </div>
        </template>
        
        <script>
        export default {
          name: 'App',
          components: {
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

      - 方式二

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0
            },
            mutations: {
                increate(state, payload) {
                    state.count += payload
                }
            },
            actions: {},
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Mutation</h2>
            <button @click="increate(2)">Mutation</button>
          </div>
        </template>
        
        <script>
        import { mapMutations } from 'vuex'
        export default {
          name: 'App',
          components: {
          },
          methods:{
            ...mapMutations(['increate'])
            或
            ...mapMutations({ myIncreate: 'increate' })
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

        

  - Action

    - 和 Mutation 类似，不同的是 Action 可以进行异步的操作

    - 内部改变状态时需要提交 Mutation

    - 使用方式

      - 方式一

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0
            },
            mutations: {
                increate(state, payload) {
                    state.count += payload
                }
            },
            actions: {
                increateAsync(context, payload) {
                    setTimeout(() => {
                        context.commit('increate', payload)
                    }, 2000)
                }
            },
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Action</h2>
            <button @click="$store.dispatch('increateAsync',2)">Action</button>
          </div>
        </template>
        
        <script>
        export default {
          name: 'App',
          components: {
          },
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

      - 方式二

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            state: {
                count: 0
            },
            mutations: {
                increate(state, payload) {
                    state.count += payload
                }
            },
            actions: {
                increateAsync(context, payload) {
                    setTimeout(() => {
                        context.commit('increate', payload)
                    }, 2000)
                }
            },
            modules: {}
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Action</h2>
            <button @click="increateAsync(2)">Action</button>
          </div>
        </template>
        
        <script>
        import { mapActions } from 'vuex'
        export default {
          name: 'App',
          components: {
          }
          methods:{
            ...mapActions(['increateAsync'])
            或
            ...mapActions({ myIncreateAsync: 'increateAsync' })
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

        

  - Module

    - 由于使用单一状态树，应用的所有状态会集中到一个大的对象上，当应用变得非常复杂时，Store 对象就有可能变得相当臃肿，为了解决此类问题，Vuex 可以利用 Module 将 Store 分割成模块，每一个 Module 可以拥有自己的 State，Getter，Mutation，Action，嵌套的子模块

    - 使用方式

      - 方式一

        ```
        const state = {
            products: [
                { id: 1, title: 'iPhone 11', price: 8000 },
                { id: 2, title: 'iPhone 12', price: 10000 }
            ]
        }
        const getters = {}
        const mutations = {
            setProducts(state, payload) {
                state.products = payload
            }
        }
        const actions = {}
        
        export default {
            state,
            getters,
            mutations,
            actions
        }
        ```

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        import products from './modules/products.js'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            modules: {
                products
            }
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            <h2>Module</h2>
            products：{{ $store.state.products.products }}
            <button @click="$store.commit('setProducts', [])">Module Mutation</button>
          </div>
        </template>
        
        <script>
        export default {
          name: 'App',
          components: {
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

      - 方式二

        ```
        const state = {
            products: [
                { id: 1, title: 'iPhone 11', price: 8000 },
                { id: 2, title: 'iPhone 12', price: 10000 }
            ]
        }
        const getters = {}
        const mutations = {
            setProducts(state, payload) {
                state.products = payload
            }
        }
        const actions = {}
        
        export default {
        	namespaced: true,
            state,
            getters,
            mutations,
            actions
        }
        ```

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'
        import products from './modules/products.js'
        
        Vue.use(Vuex)
        
        export default new Vuex.Store({
            modules: {
                products
            }
        })
        ```

        ```
        <template>
          <div id="app">
            <h1>Vuex-demo</h1>
            products：{{ products }}
            <button @click="setProducts([])">Module Mutation</button>
          </div>
        </template>
        
        <script>
        import { mapState,mapMutations } from 'vuex'
        export default {
          name: 'App',
          components: {
          },
          computed:{
            ...mapState('products', ['products'])
            或
            ...mapState('products', { myProducts: 'products' })
          },
          methods:{
            ...mapMutations('products',['setProducts'])
            或
            ...mapMutations('products', { mySetProducts: 'setProducts' })
          }
        }
        </script>
        
        <style>
        #app {
          font-family: Avenir, Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
          text-align: center;
          color: #2c3e50;
          margin-top: 60px;
        }
        </style>
        
        ```

  - Vuex 插件
  
    - Vuex 插件就是一个函数
  
    - 这个函数接收一个 store 的参数
  
      ```
      const myPlugin = store => {
      	// 当 store 初始化之后调用
      	// subscribe 的作用是用来订阅 store 中的 mutation
      	store.subscribe((mutation, state) => {
      		// 每次 mutation 之后调用
      		// mutation 的格式为 { type, payload }
      	})
      }
      ```
  
      ```
      const store = new Vuex.Store({
      	plugins: [myPlugin]
      })
      ```
  
- 简易版 Vuex

  ```
  let _Vue = null
  class Store {
    constructor (options) {
      const {
        state = {},
        getters = {},
        mutations = {},
        actions = {}
      } = options
      this.state = _Vue.observable(state)
      this.getters = Object.create(null)
      Object.keys(getters).forEach(key => {
        Object.defineProperty(this.getters, key, {
          get: () => getters[key](state)
        })
      })
      this._mutations = mutations
      this._actions = actions
    }
  
    commit (type, payload) {
      this._mutations[type](this.state, payload)
    }
  
    dispatch (type, payload) {
      this._actions[type](this, payload)
    }
  }
  
  function install (Vue) {
    if (install.installed) {
      return
    }
    install.installed = true
    _Vue = Vue
    _Vue.mixin({
      beforeCreate () {
        if (this.$options.store) {
          _Vue.prototype.$store = this.$options.store
        }
      }
    })
  }
  
  export default {
    Store,
    install
  }
  ```

### 二. 服务端渲染

#### 1. 基础

- SPA 单页面应用

  - 优点
    - 用户体验好
    - 开发效率高
    - 渲染性能好
    - 可维护性好
  - 缺点
    - 首屏渲染时间长
    - 不利于 SEO

- 同构应用

  - 通过服务端渲染首屏直出，解决 SPA 应用首屏渲染慢以及不利于 SEO 问题
  - 通过客户端渲染接管页面内容交互得到更好的用户体验
  - 这种方式通常称为现代化的服务端渲染，也叫同构渲染
  - 这种方式构建的应用称为服务端渲染应用或同构应用

- 渲染

  - 把 数据 + 模板 拼接到一起

- 传统服务端渲染

  - 过程
    - 客户端请求一个地址
    - 服务端查询页面所需要的数据
    - 数据库返回数据
    - 服务端将数据结合页面模板渲染为 HTML
    - 服务端返回 HTML 给客户端

  ```
  const express = require('express')
  const fs = require('fs')
  const template = require('art-template')
  
  const app = express()
  
  app.get('/', (req, res) => {
    // 1. 获取页面模板
    const temp = fs.readFileSync('./index.html', 'utf-8')
    // 2. 获取数据
    const data = JSON.parse(fs.readFileSync('./data.json', 'utf-8'))
    // 3. 执行渲染：数据 + 模板 = 渲染结果
    const html = template.render(temp, data)
    // 4. 把渲染结果发送给客户端
    res.send(html)
  })
  
  app.listen(3000, () => console.log('running...'))
  ```

  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>server render</title>
  </head>
  <body>
    <h1>传统的服务端渲染</h1>
    <h2>{{ title }}</h2>
    <ul>
      {{ each posts }}
      <li>{{ $value.title }}</li>
      {{ /each }}
    </ul>
  </body>
  </html>
  ```

  - 缺点
    - 网页越来越复杂的情况，存在很多不足
      - 前后端代码完全耦合在一起，不利于开发和维护
      - 前端没有足够发挥空间
      - 服务端压力大
      - 用户体验一般

- 客户端渲染

  - Ajax 使得客户端动态获取数据成为可能
  - 过程
    - 客户端请求一个地址
    - 服务端返回一个空白 HTML（只存在 js 脚本） 给客户端
    - 客户端发送 Ajax 请求数据给服务端
    - 服务端向数据库查询数据
    - 数据库返回数据给服务端
    - 服务端返回数据给客户端
    - 客户端动态渲染页面
  - 为什么客户端渲染首屏渲染慢
    - 最少要经历三次的 http 的请求周期
      - 第一次是页面的请求
      - 第二次 js 文件的请求
      - 第三次动态数据请求
  - 为什么客户端渲染不利于 SEO
    - 搜索引擎是通过获取网页内容来进行 SEO 操作的，当获取页面时，所获取到的页面内容 是一个空白 HTML（只存在 js 脚本），所以不利于 SEO

- 同构渲染

  - 同构渲染 = 后端渲染 + 前端渲染

  - 基于 React，Vue 等框架，客户端渲染和服务端渲染的结合

    - 在服务器端执行一次，用于实现服务端渲染（首屏直出）
    - 在客户端再执行一次，用于接管页面交互

  - 核心解决了 SEO 和首屏渲染慢的问题

  - 拥有传统服务端渲染的优点，也有客户端渲染的优点

  - 过程

    - 客户端向服务端请求一个地址
    - 服务端向接口服务拿到页面所需要的数据
    - 接口服务把数据返回给服务端
    - 服务端渲染页面以及生成客户端 SPA 脚本
    - 服务端返回给客户端 HTML（渲染好的页面内容 + 客户端 SPA 脚本）
    - 客户端呈现服务端返回的 HTML
    - 通过页面中的脚本激活为 SPA 应用
    - 客户端渲染页面交互

  - 实现

    - 使用 React，Vue等框架的官方解决方案

      - 优点：有助于理解原理
      - 缺点：需要搭建环境，比较麻烦

    - 使用第三方解决方案

      - React 生态的 Next.js

      - Vue 生态的 Nuxt.js

        - 安装 nuxt 包：yarn add nuxt

          ```
          <template>
            <div class="home">
              <h2>{{ title }}</h2>
              <ul>
                <li v-for="item in posts" :key="item.id">
                  {{ item.title }}
                </li>
              </ul>
            </div>
          </template>
          
          <script>
          import axios from 'axios'
          export default {
            name: 'Home',
            // Nuxt 中特殊提供的一个钩子函数，专门用于获取页面服务端渲染的数据
            async asyncData () {
              const { data } = await axios({
                method: 'GET',
                url: 'http://localhost:3000/data.json'
              })
              // 这里返回的数据会和 data () {} 中的数据合并到一起给页面使用
              return data
            }
          }
          </script>
          ```

  - 同构渲染问题

    - 开发条件所限

      - 浏览器特定的代码只能在某些生命周期钩子函数中使用
      - 一些外部扩展库可能需要特殊处理才能在服务端渲染应用中运行
      - 不能在服务端渲染期间操作 DOM
      - 某些代码操作需要区分运行环境

    - 涉及构建设置和部署的更多要求

      |      | 客户端渲染                    | 同构渲染                  |
      | ---- | ----------------------------- | ------------------------- |
      | 构建 | 仅构建客户端应用即可          | 需要构建两端              |
      | 部署 | 可以部署在任意的 WEB 服务器中 | 只能部署在 Node.js Server |

    - 更多的服务端负载

      - 在 Node 中渲染完整的应用程序，相比仅仅提供静态文件的服务器需要大量占用 cpu 资源
      - 如果应用在高流量环境下使用，需要准备相应的服务器负载
      - 需要更多的服务端渲染优化工作处理

### 三. Nuxt.js

#### 1. 基础

- Nuxt.js 是什么

  - 一个基于 Vue.js 生态的第三方开源服务端渲染应用框架
  - 可以帮助我们轻松的使用 Vue.js 技术栈构建同构应用

- Nuxt.js 的使用方式

  - 初始项目

    - 安装 nuxt

      ```
      yarn add nuxt
      ```

    - 在 package.json 中配置 script

      ```
      {
        "name": "my-nuxt",
        "version": "1.0.0",
        "description": "my-nuxt",
        "main": "index.js",
        "scripts": {
          "dev": "nuxt"
        },
        "author": "康小源",
        "license": "MIT",
        "dependencies": {
          "nuxt": "^2.14.7"
        }
      }
      ```

    - 在项目根目录新建 pages 文件夹用于存放页面

      ```
      -- pages
      	-- index.vue
      	-- about.vue
      ```

    - 编写页面文件 index.vue

      ```
      <template>
        <div>
          <h1>Hello Nuxt.js</h1>
        </div>
      </template>
      
      <script>
      export default {
        name: 'Home'
      }
      </script>
      
      <style>
      
      </style>
      ```

    - 启动项目

      ```
      yarn dev
      ```

  - 已有的 Node.js 服务端项目

    - 直接把 Nuxt 当作一个中间件集成到 Node Web Server 中

  - 现有的 Vue.js 项目

    - 非常熟悉 Nuxt.js
    - 至少 10% 的代码改动

- Nuxt 路由

  - 基础路由

  - Nuxt.js 会根据 pages 目录的组件名称来生成路由配置

    ```
    -- pages
    	-- user
    		-- index.vue
    		-- one.vue
    	-- index.vue
    ```

    ```
    import Vue from 'vue'
    import Router from 'vue-router'
    import { interopDefault } from './utils'
    import scrollBehavior from './router.scrollBehavior.js'
    
    const _f767eed2 = () => interopDefault(import('..\\pages\\about.vue' /* webpackChunkName: "pages/about" */))
    const _2ce321b1 = () => interopDefault(import('..\\pages\\user\\index.vue' /* webpackChunkName: "pages/user/index" */))
    const _970c39f6 = () => interopDefault(import('..\\pages\\user\\one.vue' /* webpackChunkName: "pages/user/one" */))
    const _b7f83948 = () => interopDefault(import('..\\pages\\index.vue' /* webpackChunkName: "pages/index" */))
    
    // TODO: remove in Nuxt 3
    const emptyFn = () => {}
    const originalPush = Router.prototype.push
    Router.prototype.push = function push (location, onComplete = emptyFn, onAbort) {
      return originalPush.call(this, location, onComplete, onAbort)
    }
    
    Vue.use(Router)
    
    export const routerOptions = {
      mode: 'history',
      base: decodeURI('/'),
      linkActiveClass: 'nuxt-link-active',
      linkExactActiveClass: 'nuxt-link-exact-active',
      scrollBehavior,
    
      routes: [{
        path: "/about",
        component: _f767eed2,
        name: "about"
      }, {
        path: "/user",
        component: _2ce321b1,
        name: "user"
      }, {
        path: "/user/one",
        component: _970c39f6,
        name: "user-one"
      }, {
        path: "/",
        component: _b7f83948,
        name: "index"
      }],
    
      fallback: false
    }
    
    export function createRouter () {
      return new Router(routerOptions)
    }
    ```

- 路由导航

  ```
  <template>
    <div>
      <h1>About Page</h1>
      <!-- a 链接，会刷新导航，走服务端渲染 -->
      <h2>a 链接</h2>
      <a href="/">首页</a>
      
      <h2>router-link</h2>
      <router-link to="/">首页</router-link>
      或
      <nuxt-link to="/">首页</nuxt-link>
  
      <h2>编程式导航</h2>
      <button @click="toIndex">首页</button>
    </div>
  </template>
  
  <script>
  export default {
    name: 'About',
    methods: {
      toIndex () {
        this.$router.push('/')
      }
    }
  }
  </script>
  
  <style>
  
  </style>
  ```

- 动态路由

  - 在 Nuxt.js 里面定义带参数的动态路由，需要创建对应的以下划线为前缀的 vue 文件或目录

    ```
    -- pages
    	-- _slug
    		-- comments.vue
    		-- index.vue
    	-- user
    		-- _id.vue
    	-- index.vue
    ```

    ```
    <template>
      <div>
        <h1>User Page</h1>
        <!-- <p>{{ $route.params.id }}</p> -->
      </div>
    </template>
    
    <script>
    export default {
      name: 'User'
    }
    </script>
    
    <style>
    
    </style>
    ```

    ```
    import Vue from 'vue'
    import Router from 'vue-router'
    import { interopDefault } from './utils'
    import scrollBehavior from './router.scrollBehavior.js'
    
    const _f767eed2 = () => interopDefault(import('..\\pages\\about.vue' /* webpackChunkName: "pages/about" */))
    const _5d7fdc59 = () => interopDefault(import('..\\pages\\user\\_id.vue' /* webpackChunkName: "pages/user/_id" */))
    const _b7f83948 = () => interopDefault(import('..\\pages\\index.vue' /* webpackChunkName: "pages/index" */))
    
    // TODO: remove in Nuxt 3
    const emptyFn = () => {}
    const originalPush = Router.prototype.push
    Router.prototype.push = function push (location, onComplete = emptyFn, onAbort) {
      return originalPush.call(this, location, onComplete, onAbort)
    }
    
    Vue.use(Router)
    
    export const routerOptions = {
      mode: 'history',
      base: decodeURI('/'),
      linkActiveClass: 'nuxt-link-active',
      linkExactActiveClass: 'nuxt-link-exact-active',
      scrollBehavior,
    
      routes: [{
        path: "/about",
        component: _f767eed2,
        name: "about"
      }, {
        path: "/user/:id?",
        component: _5d7fdc59,
        name: "user-id"
      }, {
        path: "/",
        component: _b7f83948,
        name: "index"
      }],
    
      fallback: false
    }
    
    export function createRouter () {
      return new Router(routerOptions)
    }
    ```

- 嵌套路由

  - 创建内嵌子路由，需要添加一个 Vue 文件，同时添加一个与该文件同名的目录用来存放子视图组件

    ```
    -- pages
    	-- user
    		-- _id.vue
    		-- index.vue
    	-- user.vue
    ```

    ```
    <template>
      <div>
        <h1>User Parent</h1>
        <!-- 子路由出口 -->
        <nuxt-child />
        或
        <router-view />
      </div>
    </template>
    
    <script>
    export default {
      name: 'User'
    }
    </script>
    
    <style>
    </style>
    ```

    ```
    import Vue from 'vue'
    import Router from 'vue-router'
    import { interopDefault } from './utils'
    import scrollBehavior from './router.scrollBehavior.js'
    
    const _f767eed2 = () => interopDefault(import('..\\pages\\about.vue' /* webpackChunkName: "pages/about" */))
    const _333dcc9e = () => interopDefault(import('..\\pages\\user.vue' /* webpackChunkName: "pages/user" */))
    const _2ce321b1 = () => interopDefault(import('..\\pages\\user\\index.vue' /* webpackChunkName: "pages/user/index" */))
    const _5d7fdc59 = () => interopDefault(import('..\\pages\\user\\_id.vue' /* webpackChunkName: "pages/user/_id" */))
    const _b7f83948 = () => interopDefault(import('..\\pages\\index.vue' /* webpackChunkName: "pages/index" */))
    
    // TODO: remove in Nuxt 3
    const emptyFn = () => {}
    const originalPush = Router.prototype.push
    Router.prototype.push = function push (location, onComplete = emptyFn, onAbort) {
      return originalPush.call(this, location, onComplete, onAbort)
    }
    
    Vue.use(Router)
    
    export const routerOptions = {
      mode: 'history',
      base: decodeURI('/'),
      linkActiveClass: 'nuxt-link-active',
      linkExactActiveClass: 'nuxt-link-exact-active',
      scrollBehavior,
    
      routes: [{
        path: "/about",
        component: _f767eed2,
        name: "about"
      }, {
        path: "/user",
        component: _333dcc9e,
        children: [{
          path: "",
          component: _2ce321b1,
          name: "user"
        }, {
          path: ":id",
          component: _5d7fdc59,
          name: "user-id"
        }]
      }, {
        path: "/",
        component: _b7f83948,
        name: "index"
      }],
    
      fallback: false
    }
    
    export function createRouter () {
      return new Router(routerOptions)
    }
    ```

- 自定义路由配置

  - 在项目根目录新建 nuxt.config.js 配置文件

    ```
    /**
     * Nuxt.js 配置文件
     */
    module.exports = {
      router: {
        base: '/app',
        // routes: 路由配置表
        // reslove: 解析路由组件路径
        extendRoutes (routes, resolve) {
          routes.push({
            name: 'hello',
            path: '/hello',
            component: resolve(__dirname, 'pages/about.vue')
          })
        }
      }
    }
    ```

- 视图

  - 模板

    - 如果想自定义模板，在项目根目录下创建一个 app.html 文件

      - 默认模板

        ```
        <!DOCTYPE html>
        <html {{ HTML_ATTRS }}>
          <head {{ HEAD_ATTRS }}>
            {{ HEAD }}
          </head>
          <body {{ BODY_ATTRS }}>
            {{ APP }}
          </body>
        </html>
        ```

      - 自定义模板

        ```
        <!DOCTYPE html>
        <!--[if IE 9]><html lang="en-US" class="lt-ie9 ie9" {{ HTML_ATTRS }}><![endif]-->
        <!--[if (gt IE 9)|!(IE)]><!--><html {{ HTML_ATTRS }}><!--<![endif]-->
          <head {{ HEAD_ATTRS }}>
            {{ HEAD }}
          </head>
          <body {{ BODY_ATTRS }}>
          	<h1>app.html</h1>
            {{ APP }}
          </body>
        </html>
        ```

  - 布局

    - 默认布局

      - 可通过在项目根目录下添加 layouts/default.vue 扩展默认布局

      - 默认布局

        ```
        <template>
          <nuxt />
        </template>
        ```

      - 自定义布局

        ```
        <template>
          <div>
            <h1>自定义布局</h1>
            <nuxt />
          </div>
        </template>
        ```

        ```
        <template>
          <div>
            <h1>Hello Nuxt.js</h1>
          </div>
        </template>
        
        <script>
        export default {
          name: 'Home',
          // 可以使用 layout 属性来指定布局组件
          layout: 'foo'
        }
        </script>
        
        <style>
        
        </style>
        ```

- 异步数据

  - asyncData

    - 基本用法

      - Nuxt 会将 asyncData 返回的数据融合 data 方法返回的数据一并给组件
      - 调用时机：服务端渲染期间和客户端路由更新之前

    - 注意事项

      - 只能在页面组件中使用
      - 没有 this，因为他是在组件初始化之前被调用

      ```
      <template>
        <div>
          <h1>Hello Nuxt.js</h1>
          <h2>{{ title }}</h2>
          <ul>
            <li v-for="post in posts" :key="post.id">{{ post.title }}</li>
          </ul>
        </div>
      </template>
      
      <script>
      import axios from 'axios'
      export default {
        name: 'Home',
        // 当想要动态页面内容有利于 SEO 或提升首屏渲染速度的时候，就在 asyncData 中发起请求，普通数据初始化到 data () {} 中
        async asyncData () {
          const { data } = await axios({
            method: 'GET',
            // 地址一定要写完整，如果不写完整，在服务端渲染时，默认是 80 端口，找不到 json
            url: 'http://localhost:3000/data.json'
          })
          return data
        }
      }
      </script>
      
      <style>
      
      </style>
      ```

  - 上下文对象

    ```
    <template>
      <div>
        <h1>{{ article.title }}</h1>
        <p>{{ article.body }}</p>
      </div>
    </template>
    
    <script>
    import axios from 'axios'
    export default {
      name: 'Article',
      async asyncData (context) {
        const { data } = await axios({
          method: 'GET',
          url: 'http://localhost:3000/data.json'
        })
        // context 是上下文对象，包含了服务端渲染时上下文属性
        const { id } = context.params
        return {
          article: data.posts.find(post => post.id === parseInt(id))
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

### 四. Nuxt.js 案例

- github 仓库：https://github.com/gothinkster/realworld

- 在线示例：https://demo.realworld.io/#/

- 页面模板：https://github.com/gothinkster/realworld-starter-kit/blob/master/FRONTEND_INSTRUCTIONS.md

- 接口文档：https://github.com/gothinkster/realworld/tree/master/api

- CDN 网站：https://www.jsdelivr.com
  
  - 里面是一个免费的 CDN 服务，所有的 npm 包都可以支持 CDN 链接
  
- 打包 Nuxt.js 应用

  - https://zh.nuxtjs.org/guide/commands

    | 命令          | 描述                                                         |
    | ------------- | ------------------------------------------------------------ |
    | nuxt          | 启动一个热加载的 WEB 服务器（开发模式）localhost:3000        |
    | nuxt build    | 利用 webpack 编译应用，压缩 js 和 css 资源（发布用）         |
    | nuxt start    | 以生产模式启动一个 WEB 服务器（需要先执行 nuxt build）       |
    | nuxt generate | 编译应用，并依据路由配置生成对应的 HTML 文件（用于静态站点的部署） |

- 最简单的部署方式

  - 配置 Host + Port
  - 压缩发布包
  - 把发布包传到服务器
  - 解压
  - 安装依赖
  - 启动服务

- 使用 pm2 启动服务

  - github 仓库地址：https://github.com/Unitech/pm2

  - 官方文档：https://pm2.io/

  - 安装：npm install --global pm2

  - 启动：pm2 start 脚本路径

    - pm2 start npm -- start

  - 常用命令

    | 命令        | 说明         |
    | ----------- | ------------ |
    | pm2 list    | 查看应用列表 |
    | pm2 start   | 启动应用     |
    | pm2 stop    | 停止应用     |
    | pm2 reload  | 重载应用     |
    | pm2 restart | 重启应用     |
    | pm2 delete  | 删除应用     |

- 自动化部署

  - 环境准备
    - Linux 服务器
    - 把代码提交到 Github 远程仓库
  - 配置 Github Access Token
    - 生成：点击用户选择 settings => Developer settings => Personal access tokens
    - 配置到项目的 Secrets 中：github项目仓库中 settings => secrets
  - 配置 Github Actions 执行脚本
    - 在项目根目录创建 .github/workflows 目录
    - 下载 main.yml 到 workflows 中
      - https://gist.github.com/lipengzhou/b92f80142afa37aea397da47366bd872
    - 修改配置
    - 配置 PM2 配置文件
    - 提交更新
    - 查看自动部署状态
    - 访问网站
    - 提交更新... 

