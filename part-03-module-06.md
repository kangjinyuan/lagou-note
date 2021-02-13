### 一. 搭建项目架构

#### 1. 使用 Vue Cli 创建项目

- 安装 Vue Cli
  - npm i @vue/cli -g
- 创建 vue 项目
  - vue create <project name>
  - 选择项目所需依赖
- 启动开发服务
  - 进入项目目录
  - 启动开发服务，npm run serve

#### 2. 加入 Git 版本管理

- 在 github 上创建远程仓库
- 将本地仓库推送到线上
  - 如果没有本地仓库
    - 创建本地仓库，git init
    - 将文件添加到暂存区，git add 文件
    - 提交历史记录，git commit -m '提交日志'
    - 添加远程仓库地址，git remote add origin 远程仓库地址
    - 提交推送，git push -u origin master
  - 如果有本地仓库
    - git remote add origin 远程仓库地址
    - git push -u origin master

#### 3. 初始目录结构说明

```
|-- node_modules 第三方包存储目录
|-- public 静态资源目录，任何放置在 public 文件夹的静态资源都会被简单的复制，而不经过 Webpack
|---- favicon.ico 网站图标
|---- index.html 网站入口页面
|-- src
|---- assets 公共资源目录，放置图片等资源
|---- components 公共组件目录
|---- router 路由相关模块
|---- store 容器相关模块
|---- view 路由页面组件存放目录
|---- App.vue 根组件，最终被替换渲染到 index.html 页面中 #app 入口节点
|---- main.ts 整个项目的启动入口模块
|---- shims-tsx.d.ts 支持以 .tsc 结尾的文件，在 Vue 项目中编写 jsx 代码
|---- shims-vue.d.ts 让 TypeScript 识别 .vue 模块
|-- .browserlistrc 指定了项目的目标浏览器的范围，这个值会被 @babel/preset-env 和 Autoprefixer 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀
|-- .editorconfig EditorConfig 帮助开发人员定义和维护跨编辑器（或 IDE）的统一代码风格
|-- .eslintrc.js ESLint 的配置文件
|-- .gitignore Git 的忽略配置文件，告诉 Git 项目中要忽略的文件或文件夹
|-- README.md 说明文档
|-- babel.config.js Babel 配置文件
|-- package-lock.js 记录安装时的包的版本号，以保证自己或其他人在 npm install 时大家的依赖能保持一致
|-- package.json 包说明文件，记录了项目中使用到的第三方包依赖信息等内容
|-- tsconfig.json TypeScript 配置文件
```

#### 4. 调整初始目录结构

- 删除初始化的默认文件

- 新增项目需要的目录结构

  - App.vue

    ```
    <template>
      <div id="app">
        <!-- 根路由出口 -->
        <router-view/>
      </div>
    </template>
    
    <style lang="scss" scoped></style>
    
    ```

  - router/index.js

    ```
    import Vue from 'vue'
    import VueRouter, { RouteConfig } from 'vue-router'
    
    Vue.use(VueRouter)
    
    // 路由配置规则
    const routes: Array<RouteConfig> = []
    
    const router = new VueRouter({
      routes
    })
    
    export default router
    
    ```

  - 删除 views 目录中默认文件

  - 删除 components 目录中默认文件

  - 删除 assets 目录中默认文件

  - 在 src 目录新建 utils 目录，存放工具模块

  - 在 src 目录新建 styles 目录，存放全局样式模块

  - 在 src 目录新建 api 目录，存放接口模块

#### 5. 使用 TS 开发 Vue 项目

- 环境说明
  - 方式一
    - 全新的项目，使用 Vue Cli 脚手架工具创建 Vue 项目，在创建项目时勾选 TypeScript
  - 方式二
    - 已有项目，添加 Vue 官方配置的 TypeScript 适配插件
    - 使用 @vue/cli 安装 TypeScript 插件，vue add @vue/typescript
  
- 编译器的选择
  - 要使用 TypeScript 开发 Vue 应用程序，强烈建议使用 Visual Studio Code，它为 TypeScript 提供了极好的开箱即用的支持，如果是单文件组件（SFC），可以安装提供 SFC 支持以及其他更多实用功能的 Vetur 插件
  - WebStorm 同样为 TypeScript 和 Vue 提供了开箱即用的支持

- 相关配置说明

  - TypeScript 相关依赖项

    - dependencies 依赖

      | 依赖项                 | 说明                                    |
      | ---------------------- | --------------------------------------- |
      | vue-class-component    | 提供使用 Class 语法写 Vue 组件          |
      | vue-property-decorator | 在 Class 语法基础上提供了一些辅助装饰器 |

    - devDependencies 依赖

      | 依赖项                           | 说明                                                         |
      | -------------------------------- | ------------------------------------------------------------ |
      | @typescript-eslint/eslint-plugin | 使用 ESLint 校验 TypeScript 代码                             |
      | @typescript-eslint/parser        | 将 Typescript 转为 AST 供 ESLint 校验使用                    |
      | @vue/cli-plugin-typescript       | 使用 TypeScript + ts-loader + fork-ts-checker-webpack-plugin 进行更快的类型检查 |
      | @vue/eslint-config-typescript    | 兼容 ESLint 的 TypeScript 校验规则                           |
      | typescript                       | TypeScript 编译器，提供类型校验和转换 JavaScript 功能        |

  - TypeScript 配置文件 tsconfig.json

    ```
    {
      "compilerOptions": {
        "target": "esnext",
        "module": "esnext",
        "strict": true,
        "jsx": "preserve",
        "importHelpers": true,
        "moduleResolution": "node",
        "experimentalDecorators": true,
        "skipLibCheck": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "sourceMap": true,
        "baseUrl": ".",
        "types": [
          "webpack-env"
        ],
        "paths": {
          "@/*": [
            "src/*"
          ]
        },
        "lib": [
          "esnext",
          "dom",
          "dom.iterable",
          "scripthost"
        ]
      },
      "include": [
        "src/**/*.ts",
        "src/**/*.tsx",
        "src/**/*.vue",
        "tests/**/*.ts",
        "tests/**/*.tsx"
      ],
      "exclude": [
        "node_modules"
      ]
    }
    
    ```

  - shims-vue.d.ts

    ```
    // 主要用于 TypeScript 识别 .vue 文件模块
    // TypeScript 默认不支持导入 .vue 文件模块，这个文件告诉 TypeScript 导入 .vue 文件模块都按 VueConstructor<vue> 类型识别处理 
    declare module '*.vue' {
      import Vue from 'vue'
      export default Vue
    }
    
    ```

  - shims-tsx.d.ts

    ```
    // 为 jsx 组件模板补充类型声明
    import Vue, { VNode } from 'vue'
    
    declare global {
      namespace JSX {
        // tslint:disable no-empty-interface
        interface Element extends VNode {}
        // tslint:disable no-empty-interface
        interface ElementClass extends Vue {}
        interface IntrinsicElements {
          [elem: string]: any;
        }
      }
    }
    
    ```

  - TypeScript 模块都使用 .ts 后缀

- 定义组件方式

  - 使用 Options API

    - 组件仍然可以使用以前的定义方式（导出组件选项对象或者使用 Vue.extend()）

    - 但是当我们导出一个普通的对象，此时 TypeScript 无法推断出对应的类型

    - 至于 VS Code 可以推断出类型成员的原因是因为使用 Vetur 插件

    - 这个插件明确知道导出的是一个 Vue 对象

    - 所以必须使用 Vue.extend() 方法确保 TypeScript 能够有正常的类型推断

    - 基本使用

      ```
      // lang="ts" 告诉编辑器本身和编译的时候以 ts 来识别处理
      <script lang="ts">
      import Vue from 'vue'
      
      export default Vue.extend({
        data () {
          return {
            a: 1,
            b: '2',
            c: [],
            d: {
              a: 1,
              b: 2
            }
          }
        }
      })
      ```

  - 使用 Class API

    - 在 TypeScript 下，Vue 的组件可以使用一个继承自 Vue 类型的子类表示，这种类型需要使用 Component 装饰器去修饰

    - 装饰器函数接收的参数就是以前的组件选项对象（data，props，methods）

      ```
      // lang="ts" 告诉编辑器本身和编译的时候以 ts 来识别处理
      <script lang="ts">
      import Vue from 'vue'
      import Component from 'vue-class-component'
      
      @Component({
        name: 'App'
      })
      
      export default class Button extends Vue {
        private a = 1
        private b = '1'
        private c = []
        private d = {
          a: 1,
          b: '2'
        }
        
        test () {
          console.log(this.a)
        }
      }
      </script>
      ```

- 关于装饰器语法

  - 装饰器就是一个函数，当装饰器定义在一个类上时，会立即执行装饰器函数，该类会作为参数传给装饰器函数

  - 装饰器是 ES 草案中的一个新特性，不过这个草案最近有可能发生重大调整，不建议在生产环境中使用

  - 基本用法

    ```
    function testable (target) {
    	target.isTestable  = true
    }
    
    @testable
    class MyTestableClass {
    	// ...
    }
    
    console.log(MyTestableClass.isTestable) // true
    ```

- 使用 Class Api + vue-property-decorator

  - 基本用法

    ```
    import { Vue, Component, Prop } from 'vue-property-decorator'
    
    @component
    export default class Button extends Vue {
    	private count: number = 1
    	private text: string = 'Click me'
    	@Prop() readonly size?: string
    	
    	get content () {
    		return `${this.text} ${this.count}`
    	}
    	
    	increment () {
    		this.count++
    	}
    	
    	mounted () {
    		console.log('button is mounted')
    	}
    }
    ```

  - 这种方式继续放大了 Class 这种组件定义方法

#### 6. 代码格式规范

- 良好代码格式优点

  - 更好的多人协作
  - 更好的阅读
  - 更好的维护
  - 降低代码出错的机率
  - ...

- 约束代码规范的工具

  - JSLint
  - JSHint
  - ESLint
  - ...

- 项目中代码规范说明

  - .eslintrc.js

    ```
    module.exports = {
      root: true,
      env: {
        node: true
      },
      // 插件：扩展了校验规则
      extends: [
        'plugin:vue/essential', // eslint-plugin-vue
        '@vue/standard', // @vue/eslint-config-standard
        '@vue/typescript/recommended' // @vue/eslint-config-typescript
      ],
      parserOptions: {
        ecmaVersion: 2020
      },
      // 自定义校验规则
      rules: {
        'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
        'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
      }
    }
    
    ```

  - eslint-plugin-vue

    - Github 仓库：https://github.com/vuejs/eslint-plugin-vue
    - 官方文档：https://eslint.vuejs.org/
    - 该插件使我们可以使用 ESLint 检查 .vue 文件的 <template> 和 <script>
    - 查找语法错误
    - 查找对 Vue.js 指令的错误使用
    - 查找违反 Vue.js 样式指南的行为

  - @vue/eslint-config-standard

    - starndard  的编码规则

  - @vue/eslint-config-typescript

    - TypeScript 的编码规则

  - 自定义代码格式校验规则

    - 基本使用

      ```
      rules: {
          'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
          'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
        }
      ```

    - ESLint 附带有大量的规则，可以使用注释或者配置文件修改项目中要使用的规则，要改变一个规则配置，必须将规则 ID 设置成下列值之一

      - "off" 或 0，关闭规则
      - "warn" 或 1，开启规则，使用警告级别的错误，warn 不会导致程序退出
      - "error" 或  2，开启规则，使用错误级别的错误，error 当被触发时，程序会退出

    - 在文件中配置规则，使用以下格式的注释

      - 方式一

        ```
        /* eslint eqeqeq: "off", curly: "error" */
        
        // eqeqeq 规则关闭
        // curly 规则打开，定义为错误级别
        ```

      - 方式二

        ```
        /* eslint eqeqeq: 0, curly: 2 */
        
        // eqeqeq 规则关闭
        // curly 规则打开，定义为错误级别
        ```


#### 7. 导入 Element UI 组件库

- Element UI 介绍

  - 一套为开发者，设计师，产品经理准备的基于 Vue 2.0 的桌面端组件库
  - 官网：https://element.eleme.cn/
  - Github 仓库：https://github.com/ElemeFE/element

- 安装

  - npm install element-ui -S

- 在 main.ts 导入配置

  ```
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  import store from './store'
  import ElementUI from 'element-ui'
  import 'element-ui/lib/theme-chalk/index.css'
  
  Vue.use(ElementUI)
  
  Vue.config.productionTip = false
  
  new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount('#app')
  
  ```

#### 8. 样式处理

- 样式文件结构

  ```
  |- src/styles
  |-- index.scss // 全局样式（在入口模块被加载生效）
  |-- mixin.scss // 公共的 mixin 混入（可以把重复的样式封装为 mixin 混入到复用的地方）
  |-- reset.scss // 重置基础样式
  |-- variables.scss // 公共样式变量
  ```

- variables.scss

  ```
  $primary-color: #4Q586F;
  $success-color: #51CF66;
  $warning-color: #FCC419;
  $danger-color: #FF6B6B;
  $info-color: #868e96;
  
  $body-bg: #E9EEF3;
  
  $sidebar-bg: #F8F9FB;
  $navbar-bg: #F8F9Fb;
  
  $font-family: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  
  ```

- index.scss

  ```
  @import './variables.scss';
  
  // globals
  html {
    font-family: $font-family;
    -webkit-text-size-adjust: 100%;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    // better Font Rendering
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  
  body {
    margin: 0;
    background-color: $body-bg;
  }
  
  // custom element theme
  $--color-primary: $primary-color;
  $--color-success: $success-color;
  $--color-warning: $warning-color;
  $--color-danger: $danger-color;
  $--color-info: $info-color;
  // change font path, required
  $--font-path: '~element-ui/lib/theme-chalk/fonts';
  // import element default theme
  @import '~element-ui/packages/theme-chalk/src/index';
  // node_modules/element-ui/packages/theme-chalk/src/common/var.scss
  
  // overrides
  
  // .el-menu-item, .el-submenu__title {
  //   height: 50px;
  //   line-height: 50px;
  // }
  
  .el-pagination {
    color: #868e96;
  }
  
  // components
  
  .status {
    display: inline-block;
    cursor: pointer;
    width: .875rem;
    height: .875rem;
    vertical-align: middle;
    border-radius: 50%;
  
    &-primary {
      background: $--color-primary;
    }
  
    &-success {
      background: $--color-success;
    }
  
    &-warning {
      background: $--color-warning;
    }
  
    &-danger {
      background: $--color-danger;
    }
  
    &-info {
      background: $--color-info;
    }
  }
  
  ```


- 在 main.ts 中导入 index.scss

  ```
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  import store from './store'
  import ElementUI from 'element-ui'
  // 在 index.scss 中已经加载了 element-ui/lib/theme-chalk/index.css，所以可以不用单独加载
  // import 'element-ui/lib/theme-chalk/index.css'
  
  // 加载全局样式
  import './styles/index.scss'
  
  Vue.use(ElementUI)
  
  Vue.config.productionTip = false
  
  new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount('#app')
  
  ```

- 共享全局样式


  - 在 项目根目录新建 vue.config.js 配置文件

    ```
    // vue.config.js
    module.exports = {
      css: {
        loaderOptions: {
          // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
          // 因为 `scss` 语法在内部也是由 sass-loader 处理的
          // 但是在配置 `prependData` 选项的时候
          // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
          // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
          scss: {
            prependData: `@import "~@/styles/variables.scss";`
          }
        }
      }
    }
    
    ```

#### 9. 接口处理

- 配置接口代理

  - 后台提供接口

    - http://113.31.105.128/boss/doc.html#/home
    - http://113.31.105.128/front/doc.html#/home

  - 这两个接口都没有提供 CORS 跨域请求，所以需要在客户端配置服务端代理处理跨域

  - 配置客户端层面的服务器代理跨于参考文档

    - https://cli.vuejs.org/zh/config/#devserver-proxy
    - https://github.com/chimurai/http-proxy-middleware

  - 具体配置

    - vue.config.js

      ```
      // vue.config.js
      module.exports = {
        css: {
          loaderOptions: {
            // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
            // 因为 `scss` 语法在内部也是由 sass-loader 处理的
            // 但是在配置 `prependData` 选项的时候
            // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
            // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
            scss: {
              prependData: `@import "~@/styles/variables.scss";`
            }
          }
        },
      
        devServer: {
          proxy: {
            '/boss': {
              target: 'http://eduboss.lagou.com',
              changeOrigin: true // 把请求头中的 host 配置为 target
            },
            '/front': {
              target: 'http://edufront.lagou.com',
              changeOrigin: true
            }
          }
        }
      }
      
      ```

- 封装请求模块

  - 安装 axios

    - npm install axios

  - 创建 src/utils/request.ts

    ```
    import axios, { AxiosRequestConfig } from 'axios'
    import store from '@/store'
    
    const request = axios.create({
      // 配置选项
    })
    
    // 请求拦截器
    request.interceptors.request.use((config: AxiosRequestConfig) => {
      const { user } = store.state
    if (user) {
        config.headers.Authorization = user.access_token
    }
      return config
    }, (err: Error) => {
      return Promise.reject(err)
    })
    
    // 响应拦截器
    
    export default request
    
    ```
    

#### 10. 布局

- 初始化路由相关的页面组件

  | 路径         | 模块路径                    | 说明       |
  | ------------ | --------------------------- | ---------- |
  | /            | views/home/index.vue        | 首页       |
  | /login       | views/login/index.vue       | 用户登录   |
  | /role        | views/role/index.vue        | 角色管理   |
  | /menu        | views/menu/index.vue        | 菜单管理   |
  | /resource    | views/resource/index.vue    | 资源管理   |
  | /course      | views/course/index.vue      | 课程管理   |
  | /user        | views/user/index.vue        | 用户管理   |
  | /advert      | views/advert/index.vue      | 广告管理   |
  | /advertSpace | views/advertSpace/index.vue | 广告位管理 |
| /404         | views/error/index.vue       | 404页面    |
  
- 在 src 目录新建布局组件

  - 布局容器，layout/index.vue

    ```
    <template>
      <div class="layot-container">
        <h1>头部</h1>
        <h2>侧边栏</h2>
    
        <!-- 子路由出口 -->
        <router-view />
      </div>
    </template>
    
    <script lang="ts">
    import Vue from 'vue'
    
    export default Vue.extend({
      name: 'Layout'
    })
    </script>
    
    <style lang="scss" scoped>
    
    </style>
    
    ```

  - 侧边栏菜单，layout/components/aside/index.vue

    ```
    <template>
      <div class="app-aside-container">
        <el-menu
          default-active="2"
          @open="handleOpen"
          @close="handleClose"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          :router="true"
        >
          <el-submenu index="1">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>权限管理</span>
            </template>
            <template>
              <el-menu-item index="/role">
                <i class="el-icon-location"></i>
                <span>角色管理</span>
              </el-menu-item>
              <el-menu-item index="/menu">
                <i class="el-icon-location"></i>
                <span>菜单管理</span>
              </el-menu-item>
              <el-menu-item index="/resource">
                <i class="el-icon-location"></i>
                <span>资源管理</span>
              </el-menu-item>
            </template>
          </el-submenu>
          <el-menu-item index="/course">
            <i class="el-icon-menu"></i>
            <span slot="title">课程管理</span>
          </el-menu-item>
          <el-menu-item index="/user">
            <i class="el-icon-document"></i>
            <span slot="title">用户管理</span>
          </el-menu-item>
          <el-submenu index="4">
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span>广告管理</span>
            </template>
            <template>
              <el-menu-item index="/advert">
                <i class="el-icon-location"></i>
                <span>广告列表</span>
              </el-menu-item>
              <el-menu-item index="/advertSpace">
                <i class="el-icon-location"></i>
                <span>广告位列表</span>
              </el-menu-item>
            </template>
          </el-submenu>
        </el-menu>
      </div>
    </template>
    
    <script lang="ts">
    import Vue from 'vue'
    
    export default Vue.extend({
      name: 'AppAside',
      methods: {
        handleOpen (key: string, keyPath: string[]): void {
          console.log(key, keyPath)
        },
        handleClose (key: string, keyPath: string[]): void {
          console.log(key, keyPath)
        }
      }
    })
    </script>
    
    <style lang="scss" scoped>
      .app-aside-container {
        .el-menu {
          min-height: 100vh;
        }
      }
    </style>
    
    ```

  - 头部，layout/heander/index.vue

    ```
    <template>
      <div class="app-header-container">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>活动管理</el-breadcrumb-item>
          <el-breadcrumb-item>活动列表</el-breadcrumb-item>
          <el-breadcrumb-item>活动详情</el-breadcrumb-item>
        </el-breadcrumb>
        <el-dropdown @command="logout">
          <span class="el-dropdown-link">
            <el-avatar shape="circle" :size="40" :src="userInfo.portrait || defaultAvatar"></el-avatar>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>{{ userInfo.userName }}</el-dropdown-item>
            <el-dropdown-item divided command="logout">退出</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </template>
    
    <script lang="ts">
    import Vue from 'vue'
    import { getUserInfo } from '@/api/user'
    
    export default Vue.extend({
      name: 'AppHeader',
      data () {
        return {
          userInfo: {},
          defaultAvatar: require('@/assets/common/default-avatar.png')
        }
      },
      methods: {
        async getUserInfo () {
          const { data } = await getUserInfo()
          if (data.state !== 1) {
            this.$message.error(data.message)
          } else {
            this.userInfo = data.content
          }
        },
        logout (command: string) {
          if (command === 'logout') {
            this.$confirm('确认要退出吗？', '退出提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.$message.success('退出成功')
              this.$store.commit('setUser', null)
              this.$router.push({
                name: 'login',
                query: {
                  redirect: this.$route.path
                }
              })
            }).catch(() => {
              this.$message.info('已取消退出')
            })
          }
        }
      },
      mounted () {
        this.getUserInfo()
      }
    })
    </script>
    
    <style lang="scss" scoped>
    .app-header-container {
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    </style>
    
    ```

- 重写 vue-router 路由表

  ```
  import Vue from 'vue'
  import VueRouter, { Route, RouteConfig } from 'vue-router'
  import store from '@/store'
  import Layout from '@/layout/index.vue'
  
  Vue.use(VueRouter)
  
  // 路由配置规则
  const routes: Array<RouteConfig> = [
    {
      name: 'login',
      path: '/login',
      component: () => import(/* webpackChunkName: 'login' */ '@/views/login/index.vue')
    },
    {
      path: '/',
      component: Layout,
      meta: {
        requiresAuth: true
      },
      children: [
        {
          name: 'home',
          path: '', // 默认子路由
          component: () => import(/* webpackChunkName: 'home' */ '@/views/home/index.vue')
        },
        {
          name: 'advert',
          path: '/advert',
          component: () => import(/* webpackChunkName: 'advert' */ '@/views/advert/index.vue')
        },
        {
          name: 'advertSpace',
          path: '/advertSpace',
          component: () => import(/* webpackChunkName: 'advertSpace' */ '@/views/advertSpace/index.vue')
        },
        {
          name: 'course',
          path: '/course',
          component: () => import(/* webpackChunkName: 'course' */ '@/views/course/index.vue')
        },
        {
          name: 'menu',
          path: '/menu',
          component: () => import(/* webpackChunkName: 'menu' */ '@/views/menu/index.vue')
        },
        {
          name: 'resource',
          path: '/resource',
          component: () => import(/* webpackChunkName: 'resource' */ '@/views/resource/index.vue')
        },
        {
          name: 'role',
          path: '/role',
          component: () => import(/* webpackChunkName: 'role' */ '@/views/role/index.vue')
        },
        {
          name: 'user',
          path: '/user',
          component: () => import(/* webpackChunkName: 'user' */ '@/views/user/index.vue')
        }
      ]
    },
    {
      name: '404',
      path: '*',
      component: () => import(/* webpackChunkName: '404' */ '@/views/error/404.vue')
    }
  ]
  
  const router = new VueRouter({
    routes
  })
  
  // 全局前置守卫
  // to: 要去的路由信息
  // from: 从哪里来的路由信息
  // next: 通行的标志
  router.beforeEach((to: Route, from: Route, next: Function) => {
    if (to.matched.some(record => record.meta.requiresAuth)) {
      if (store.state.user) {
        next()
      } else {
        next({
          name: 'login',
          query: {
            redirect: to.path
          }
        })
      }
    } else {
      next()
    }
  })
  
  export default router
  
  ```

#### 11. Vuex 容器

- src/store/index.ts

  ```
  import Vue from 'vue'
  import Vuex from 'vuex'
  
  Vue.use(Vuex)
  
  export default new Vuex.Store({
    state: {
      user: JSON.parse(window.localStorage.getItem('user') || 'null')
      // user: null // 当前登录用户状态
    },
    mutations: {
      setUser (state, user) {
        state.user = JSON.parse(user)
        window.localStorage.setItem('user', user)
      }
    },
    actions: {
    },
    modules: {
    }
  })
  
  ```

  