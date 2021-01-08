### 一. 搭建自己的 SSR

#### 1. 渲染一个 Vue 实例

- 创建一个目录，my-vue-ssr

- 初始化项目，npm init

- 安装基础依赖包，npm install vue vue-server-renderer

- 在根目录创建一个模块 server.js

  ```
  const Vue = require('vue')
  
  // 创建渲染器
  const renderer = require('vue-server-renderer').createRenderer()
  
  const app = new Vue({
    template: `
      <div id="app">
        <h1>{{ message }}</h1>
      </div>
    `,
    data: {
      message: 'Hello World'
    }
  })
  
  renderer.renderToString(app, (err, html) => {
    if (err) throw err
    console.log(html) 
    // <div id="app" data-server-rendered="true"><h1>Hello World</h1></div>
    // data-server-rendered="true" 是客户端渲染时所用到的标识
  })
  ```

- 渲染页面

  ```
  const Vue = require('vue')
  const express = require('express')
  
  // 创建渲染器
  const renderer = require('vue-server-renderer').createRenderer()
  
  const server = express()
  
  server.get('/', (req, res) => {
    const app = new Vue({
      template: `
        <div id="app">
          <h1>{{ message }}</h1>
        </div>
      `,
      data: {
        message: '我叫康小源'
      }
    })
    
    renderer.renderToString(app, (err, html) => {
      if (err) {
        return res.status(500).send('Internal Server Error')
      }
      // 解决乱码
      // 方式一 给响应头添加 Content-Type
      res.setHeader('Content-Type', 'text/html; charset=utf8')
      // 方式二 在发送内容中声明一个 meta 标签
      res.end(`
        <!DOCTYPE html>
        <html lang="en">
          <head>
            <meta charset="utf-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <title>Document</title>
          </head>
          <body>
            ${html}
          </body>
        </html>
      `)
    })
  })
  
  server.listen(3000, () => {
    console.log('Server running at prot 3000')
  })
  ```

- 通过 HTML 模板的方式渲染页面

  - 创建 index.template.html 模板

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <!-- 标记渲染内容，渲染的内容会替换此注释 -->
      <!--vue-ssr-outlet-->
    </body>
    </html>
    ```

  - 在 server.js 解析

    ```
    const Vue = require('vue')
    const express = require('express')
    const fs = require('fs')
    
    // 创建渲染器
    const renderer = require('vue-server-renderer').createRenderer({
      // 该属性会将 模板和 renderToString 解析出来的 html 结合起来
      template: fs.readFileSync('./index.template.html', 'utf-8')
    })
    
    const server = express()
    
    server.get('/', (req, res) => {
      const app = new Vue({
        template: `
          <div id="app">
            <h1>{{ message }}</h1>
          </div>
        `,
        data: {
          message: '我叫康小源'
        }
      })
      
      renderer.renderToString(app, (err, html) => {
        if (err) {
          return res.status(500).send('Internal Server Error')
        }
        res.end(html)
      })
    })
    
    server.listen(3000, () => {
      console.log('Server running at prot 3000')
    })
    ```

- 在模板中使用外部数据

  - index.template.html

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <!-- 使用三个 {} 来绑定标签 -->
      {{{ meta }}}
      <title>{{ title }}</title>
    </head>
    <body>
      <!-- 标记渲染内容，渲染的内容会替换此注释 -->
      <!--vue-ssr-outlet-->
    </body>
    </html>
    ```

  - server.js

    ```
    const Vue = require('vue')
    const express = require('express')
    const fs = require('fs')
    
    // 创建渲染器
    const renderer = require('vue-server-renderer').createRenderer({
      // 该属性会将 renderToString 解析出来的 html 结合起来
      template: fs.readFileSync('./index.template.html', 'utf-8')
    })
    
    const server = express()
    
    server.get('/', (req, res) => {
      const app = new Vue({
        template: `
          <div id="app">
            <h1>{{ message }}</h1>
          </div>
        `,
        data: {
          message: '我叫康小源'
        }
      })
      
      renderer.renderToString(app, {
        // 以数据的方式渲染
        title: 'my-ssr',
        // 以 html 标签的方式渲染
        meta: `
          <meta name="description" content="ssr" />
        `
      }, (err, html) => {
        if (err) {
          return res.status(500).send('Internal Server Error')
        }
        res.end(html)
      })
    })
    
    server.listen(3000, () => {
      console.log('Server running at prot 3000')
    })
    ```


#### 2. 构建配置

- 基本思路

  - 通过 webpack 打包构建生成对应的 server-bundle 和 client-bundle，server-bundle 用于渲染生成的页面，client-bundle 用于驱动页面的动态交互

  - 具体构建流程

    ![vue-ssr-build-](https://cloud.githubusercontent.com/assets/499550/17607895/786a415a-5fee-11e6-9c11-45a2cfdf085c.png)

- 源码结构

  ```
  - src
  	- components // 组件
  		- Foo.vue
  		- Bar.vue
  		- Baz.vue
  	- App.vue
  	- app.js // 通用入口文件
  	- entry-client.js // 客户端入口文件
  	- entry-server.js // 服务端入口文件
  ```

  - App.vue

    ```
    <template>
      <div id="app">
        <h1>{{ message }}</h1>
        <h2>客户端动态交互</h2>
        <div>
          <input type="text" v-model="message">
        </div>
        <div>
          <button @click="onClick">测试</button>
        </div>
      </div>
    </template>
    
    <script>
    export default {
      name: 'App',
      data () {
        return {
          message: '我叫康小源'
        }
      },
      methods: {
        onClick () {
          console.log('Hello World')
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

  - app.js

    ```
    import Vue from 'vue'
    import App from './App.vue'
    
    export function createApp () {
      const app = new Vue({
        render: h => h(App)
      })
      return { app }
    }
    ```

  - entry-client.js

    ```
    import { createApp } from './app'
    
    const { app } = createApp()
    
    app.$mount('#app')
    ```

  - entry-server.js

    ```
    import { createApp } from './app'
    
    export default context => {
      const { app } = createApp()
      return app
    }
    ```

- 安装依赖

  - 生产依赖

    | 包                  | 说明                                |
    | ------------------- | ----------------------------------- |
    | vue                 | Vue.js 核心库                       |
    | vue-server-renderer | Vue 服务端渲染工具                  |
    | express             | 基于 Node 的 Web 服务框架           |
    | cross-env           | 通过 npm scripts 设置跨平台环境变量 |

  - 开发依赖

    | 包                                                           | 说明                                   |
    | ------------------------------------------------------------ | -------------------------------------- |
    | webpack                                                      | webpack 核心包                         |
    | webpack-cli                                                  | webpack 命令行工具                     |
    | webpack-merge                                                | webpack 信息合并工具                   |
    | webpack-node-externals                                       | 排除 webpack 中的 Node 模块            |
    | rimraf                                                       | 基于 Node 封装的一个跨平台 rm -rf 工具 |
    | friendly-errors-webpack-plugin                               | 友好的 webpack 错误提示                |
    | @babel/core <br/>@babel/plugin-transform-runtime<br/>@babel/preset-env<br/>@babel-loader | Babel 相关工具                         |
    | vue-loader<br/>vue-template-compiler                         | 处理 .vue 资源                         |
    | file-loader                                                  | 处理字体资源                           |
    | css-loader                                                   | 处理 css 资源                          |
    | url-loader                                                   | 处理图片资源                           |
  
- webpack 配置文件

  ```
  - build
  	- wbepack.base.config.js
  	- webpack.client.config.js
  	- webpack.server.config.js
  ```

  - webpack.base.config.js

    ```
    /**
     * 公共配置
     */
    
    const vueLoaderPlugin = require('vue-loader/lib/plugin')
    const path = require('path')
    const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin')
    const resolve = file => path.resolve(__dirname, file)
    
    const isProd = process.env.NODE_ENV === 'production'
    
    module.exports = {
      mode: isProd ? 'production' : 'development',
      output: {
        filename: '[name].[chunkhash].js',
        path: resolve('../dist'),
        publicPath: 'dist'
      },
      resolve: {
        alias: {
          '@': resolve('./src')
        },
        extensions: ['.js', '.vue', '.json']
      },
      devtool: isProd ? 'source-map' : 'cheap-module-eval-source-map',
      module: {
        rules: [
          {
            test: /\.(png|jpg|gif)$/i,
            use:[
              {
                loader: 'url-loader',
                options: {
                  limit: 8192
                }
              }
            ]
          },
          {
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            use: ['file-loader']
          },
          {
            test: /\.vue$/,
            loader: 'vue-loader'
          },
          {
            test: /\.css$/,
            use: [
              'vue-style-loader',
              'css-loader'
            ]
          }
        ]
      },
      plugins: [
        new vueLoaderPlugin(),
        new FriendlyErrorsWebpackPlugin()
      ]
    }
    ```

  - webpack.client.config.js

    ```
    /**
     * 客户端打包配置
     */
    const { merge } = require('webpack-merge')
    const baseConfig = require('./webpack.base.config')
    const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')
    
    module.exports = merge(baseConfig, {
      entry: {
        app: './src/entry-client.js'
      },
      optimization: {
        splitChunks: {
          name: 'manifest',
          minChunks: Infinity
        }
      },
      module: {
        rules: [
          {
            test: /\.m?js$/,
            exclude: /(node_modules|bower_components)/,
            use: {
              loader: 'babel-loader',
              options: {
                presets: ['@babel/preset-env'],
                cacheDirectory: true,
                plugins: ['@babel/plugin-transform-runtime']
              }
            }
          }
        ]
      },
      plugins: [
        new VueSSRClientPlugin()
      ]
    })
    ```

  - webpack.server.config.js

    ```
    /**
     * 服务端打包配置
     */
    const { merge } = require('webpack-merge')
    const nodeExternals = require('webpack-node-externals')
    const baseConfig = require('./webpack.base.config')
    const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')
    
    module.exports = merge(baseConfig, {
      entry: './src/entry-server.js',
      target: 'node',
      output: {
        filename: 'server-bundle.js',
        libraryTarget: 'commonjs2'
      },
      externals: [nodeExternals({
        allowlist: [/\.css$/]
      })],
      plugins: [
        new VueSSRServerPlugin()
      ]
    })
    ```

- 构建命令

  - package.json

    ```
    {
      "name": "my-ssr2",
      "version": "1.0.0",
      "description": "my-ssr2",
      "main": "index.js",
      "scripts": {
        "build:client": "cross-env NODE_ENV=production webpack --config ./build/webpack.client.config.js",
        "build:server": "cross-env NODE_ENV=production webpack --config ./build/webpack.server.config.js",
        "build": "rimraf dist && npm run build:client && npm run build:server"
      },
      "author": "康小源",
      "license": "ISC",
      "dependencies": {
        "cross-env": "^7.0.3",
        "express": "^4.17.1",
        "vue": "^2.6.12",
        "vue-server-renderer": "^2.6.12"
      },
      "devDependencies": {
        "@babel/core": "^7.12.9",
        "@babel/plugin-transform-runtime": "^7.12.1",
        "@babel/preset-env": "^7.12.7",
        "babel-loader": "^8.2.2",
        "css-loader": "^5.0.1",
        "file-loader": "^6.2.0",
        "friendly-errors-webpack-plugin": "^1.7.0",
        "rimraf": "^3.0.2",
        "url-loader": "^4.1.1",
        "vue-loader": "^15.9.5",
        "vue-template-compiler": "^2.6.12",
        "webpack": "^4.44.2",
        "webpack-cli": "^3.3.12",
        "webpack-merge": "^5.4.0",
        "webpack-node-externals": "^2.5.2"
      }
    }
    
    ```

- 启动应用

  - server.js

    ```
    const express = require('express')
    const fs = require('fs')
    
    const serverBundle = require('./dist/vue-ssr-server-bundle.json')
    const template = fs.readFileSync('./index.template.html', 'utf-8')
    const clientManifest = require('./dist/vue-ssr-client-manifest.json')
    const renderer = require('vue-server-renderer').createBundleRenderer(serverBundle, {
      template,
      clientManifest
    })
    
    const server = express()
    
    // 注册静态资源中间件
    server.use('/dist', express.static('./dist'))
    
    server.get('/', (req, res) => {
      renderer.renderToString({
        title: 'my-ssr',
        meta: `
          <meta name="description" content="ssr" />
        `,
    
      }, (err, html) => {
        if (err) {
          return res.status(500).end('Internal Server Error')
        }
        res.setHeader('Content-Type', 'text/html; charset=utf-8')
        res.end(html)
      })
    })
    
    server.listen(3000, () => {
      console.log('server running at port 3000')
    })
    ```

  - 启动应用

    - node server.js 启动应用
  
- 构建配置开发模式

  - 基本思路

    - 根据源代码的改变重新生成打包的资源文件，进而重新调用 createBundleRenderer 来生成新的 render 进行渲染

    - package.json

      ```
      {
        "name": "my-ssr2",
        "version": "1.0.0",
        "description": "my-ssr2",
        "main": "index.js",
        "scripts": {
          "build:client": "cross-env NODE_ENV=production webpack --config ./build/webpack.client.config.js",
          "build:server": "cross-env NODE_ENV=production webpack --config ./build/webpack.server.config.js",
          "build": "rimraf dist && npm run build:client && npm run build:server",
          // 生成模式启动的 web 服务
          "start": "corss-env NODE_ENV=production node ./server.js",
          // 开发模式启动的 web 服务
          "dev": "node ./server.js"
        },
        "author": "康小源",
        "license": "ISC",
        "dependencies": {
          "cross-env": "^7.0.3",
          "express": "^4.17.1",
          "vue": "^2.6.12",
          "vue-server-renderer": "^2.6.12"
        },
        "devDependencies": {
          "@babel/core": "^7.12.9",
          "@babel/plugin-transform-runtime": "^7.12.1",
          "@babel/preset-env": "^7.12.7",
          "babel-loader": "^8.2.2",
          "css-loader": "^5.0.1",
          "file-loader": "^6.2.0",
          "friendly-errors-webpack-plugin": "^1.7.0",
          "rimraf": "^3.0.2",
          "url-loader": "^4.1.1",
          "vue-loader": "^15.9.5",
          "vue-template-compiler": "^2.6.12",
          "webpack": "^4.44.2",
          "webpack-cli": "^3.3.12",
          "webpack-merge": "^5.4.0",
          "webpack-node-externals": "^2.5.2"
        }
      }
      
      ```

    - server.js

      ```
      const express = require('express')
      const fs = require('fs')
      const { createBundleRenderer } = require('vue-server-renderer')
      const setupDevServer = require('./build/setup-dev-server')
      
      const server = express()
      
      // express.static 处理的是物理磁盘中的资源文件
      server.use('/dist', express.static('./dist'))
      
      // 判断运行环境
      const isProd = process.env.NODE_ENV === 'production'
      
      let renderer
      let onReady
      if (isProd) {
        // 生产模式
        const serverBundle = require('./dist/vue-ssr-server-bundle.json')
        const template = fs.readFileSync('./index.template.html', 'utf-8')
        const clientManifest = require('./dist/vue-ssr-client-manifest.json')
        renderer = createBundleRenderer(serverBundle, {
          template,
          clientManifest
        })
      } else {
        // 开发模式 => 打包构建 -> 重新生成 renderer 渲染器
        // 设置开发模式服务
        onReady = setupDevServer (server, (serverBundle, template, clientManifest) => {
          renderer = createBundleRenderer(serverBundle, {
            template,
            clientManifest
          })
        })
      }
      
      const render = (req, res) => {
        renderer.renderToString({
          title: 'my-ssr',
          meta: `
            <meta name="description" content="ssr" />
          `,
      
        }, (err, html) => {
          if (err) {
            return res.status(500).end('Internal Server Error')
          }
          res.setHeader('Content-Type', 'text/html; charset=utf-8')
          res.end(html)
        })
    }
      
      server.get('/', isProd ? render : async (req, res) => {
        // 等待有了 renderer 渲染器之后，调用 render 进行渲染
        await onReady
        render(req, res)
      })
      
      server.listen(3000, () => {
        console.log('server running at port 3000')
      })
      ```
      
    - setup-dev-server.js

      ```
      const fs = require('fs')
      const path = require('path')
      const chokidar = require('chokidar')
      const webpack = require('webpack')
      const devMiddleware = require('webpack-dev-middleware')
      const hotMiddleware = require('webpack-hot-middleware')
      
      const resolve = file => path.resolve(__dirname, file)
      
      module.exports = (server, callback) => {
        let ready
        const onReady = new Promise(resolve => ready = resolve)
        // 监视构建 -> 更新 renderer
        let serverBundle
        let template
        let clientManifest
      
        const update = () => {
          if (serverBundle, template, clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
          }
        }
      
        // 监视构建 serverBundle -> 调用 update -> 更新 renderer 渲染器
        const serverConfig = require('./webpack.server.config')
        const serverCompiler = webpack(serverConfig)
        // devMiddleware 会自动执行打包构建，默认通过监视的方式打包构建
        const serverDevMiddleware = devMiddleware(serverCompiler, {
          // 关闭日志输出，由 friendlyErrorsWebpackPlugin 处理
          logLevel: 'silent'
        })
        serverCompiler.hooks.done.tap('server', () => {
          serverBundle = JSON.parse(serverDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-server-bundle.json'), 'utf-8'))
          update()
        })
        // 监视构建 template -> 调用 update -> 更新 renderer 渲染器
        const templatePath = resolve('../index.template.html')
        template = fs.readFileSync(templatePath, 'utf-8')
        update()
        // 监视文件变化
        chokidar.watch(templatePath).on('change', () => {
          template = fs.readFileSync(templatePath, 'utf-8')
          update()
        })
        // 监视构建 clientManifest -> 调用 update -> 更新 renderer 渲染器
        const clientConfig = require('./webpack.client.config')
        clientConfig.plugins.push(new webpack.HotModuleReplacementPlugin())
        clientConfig.entry.app = [
          // 和服务端交互处理热更新的客户端脚本
          // quite 关闭浏览器日志输出
          // 重新加载
          'webpack-hot-middleware/client?quite=true&reload=true',
          clientConfig.entry.app
        ]
        // 热更新模式下确保一致的 hash
        clientConfig.output.filename = '[name].js'
        const clientCompiler = webpack(clientConfig)
        // devMiddleware 会自动执行打包构建，默认通过监视的方式打包构建
        const clientDevMiddleware = devMiddleware(clientCompiler, {
          // 客户端打包需要单独配置
          // 配置构建输出中请求前缀路径
          publicPath: clientConfig.output.publicPath,
          logLevel: 'silent'
        })
        clientCompiler.hooks.done.tap('client', () => {
          clientManifest = JSON.parse(clientDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-client-manifest.json'), 'utf-8'))
          update()
        })
      
        // 将 hotMiddleware 挂载到 express 服务中
        server.use(hotMiddleware(clientCompiler, {
          // 关闭日志
          log: false
        }))
      
        // 将 clientDevMiddleware 挂载到 express 服务中，提供对其内部内存中数据的访问
        server.use(clientDevMiddleware)
      
        return onReady
      }
      ```

#### 3. 路由处理

- 配置 Router

  - 安装 vue-router，npm i vue-router

  - 在 src 目录新建 pages 目录，创建页面

  - 在 src 目录新建 router 目录

  - 在 router 目录新建 index.js 模块

  - index.js

    ```
    import Vue from 'vue'
    import VueRouter from 'vue-router'
    import Home from '@/pages/Home'
    
    Vue.use(VueRouter)
    
    export const createRouter = () => {
      const router = new VueRouter({
        // 同构应用不能使用 hash 模式，需要使用 history 方式
        mode: 'history',
        routes: [
          {
            name: 'home',
            path: '/',
            components: Home
          },
          {
            name: 'about',
            path: '/about',
            components: () => import(/* webpackChunkName: 'about' */ '@/pages/about')
          },
          {
            name: 'error404',
            path: '*',
            components: () => import(/* webpackChunkName: 'error404' */ '@/pages/404')
          }
        ]
      })
    }	
    ```

- 将路由注册到根实例

  - app.js

    ```
    import Vue from 'vue'
    import { createRouter } from '@/router'
    import App from './App.vue'
    
    export function createApp () {
      const router = createRouter()
      const app = new Vue({
        router,
        render: h => h(App)
      })
      return { app, router }
    }
    ```

- 适配服务端入口

  - entry-server.js

    ```
    import { createApp } from './app'
    
    export default async context => {
      const { app, router } = createApp()
      // 设置服务端 router 的位置
      router.push(context.url)
      // 等到 router 将可能的异步组件和钩子函数解析完
      await new Promise(router.onReady)
      return app
    }
    ```

- 服务端 server 适配

  - server.js

    ```
    const express = require('express')
    const fs = require('fs')
    const { createBundleRenderer } = require('vue-server-renderer')
    const setupDevServer = require('./build/setup-dev-server')
    
    const server = express()
    
    // express.static 处理的是物理磁盘中的资源文件
    server.use('/dist', express.static('./dist'))
    
    // 判断运行环境
    const isProd = process.env.NODE_ENV === 'production'
    
    let renderer
    let onReady
    if (isProd) {
      // 生产模式
      const serverBundle = require('./dist/vue-ssr-server-bundle.json')
      const template = fs.readFileSync('./index.template.html', 'utf-8')
      const clientManifest = require('./dist/vue-ssr-client-manifest.json')
      renderer = createBundleRenderer(serverBundle, {
        template,
        clientManifest
      })
    } else {
      // 开发模式 => 打包构建 -> 重新生成 renderer 渲染器
      // 设置开发模式服务
      onReady = setupDevServer (server, (serverBundle, template, clientManifest) => {
        renderer = createBundleRenderer(serverBundle, {
          template,
          clientManifest
        })
      })
    }
    
    const render = async (req, res) => {
      try {
        const html = await renderer.renderToString({
          title: 'my-ssr',
          meta: `
            <meta name="description" content="ssr" />
          `,
          url: req.url
        })
        res.setHeader('Content-Type', 'text/html; charset=utf-8')
        res.end(html)
      } catch (err) {
        res.status(500).end('Internal Server Error')
      }
    }
    
    // 服务端路由设置为 *，意味着所有的路由都会进入这里
    server.get('/', isProd ? render : async (req, res) => {
      // 等待有了 renderer 渲染器之后，调用 render 进行渲染
      await onReady
      render(req, res)
    })
    
    server.listen(3000, () => {
      console.log('server running at port 3000')
    })
    ```

- 适配客户端入口

  - entry-client.js

    ```
    import { createApp } from './app'
    
    const { app, router } = createApp()
    
    router.onReady(() => {
      app.$mount('#app')
    })
    ```

#### 4. 管理页面的 head

- 安装 vue-meta

- 设置公共的页面 title

  - app.js

    ```
    import Vue from 'vue'
    import App from './App.vue'
    import { createRouter } from '@/router'
    import VueMeta from 'vue-meta'
    
    Vue.use(VueMeta)
    
    Vue.mixin({
      metaInfo: {
        titleTemplate: '%s - 拉勾教育'
      }
    })
    
    export function createApp () {
      const router = createRouter()
      const app = new Vue({
        router,
        render: h => h(App)
      })
      return { app, router }
    }
    ```

- 将 meta 信息放入上下文

  - entry-server.js

    ```
    import { createApp } from './app'
    
    export default async context => {
      const { app, router } = createApp()
    
      const meta = app.$meta()
    
      // 设置服务端 router 的位置
      router.push(context.url)
    
      // 设置 meta 信息
      context.meta = meta
    
      // 等到 router 将可能的异步组件和钩子函数解析完
      await new Promise(router.onReady.bind(router))
      return app
    }
    
    ```

- 给页面设置自己的 meta 信息

  - Home.vue

    ```
    <template>
      <div>
        <h1>Home Page</h1>
      </div>
    </template>
    
    <script>
    export default {
      name: 'Home',
      metaInfo: {
        title: '首页'
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

#### 5. 数据预取和状态管理

- 安装 vuex，npm i vuex

- 在 src 目录下创建 store 目录

- 在 store 目录下创建 index.js

  - index.js

    ```
    import Vue from 'vue'
    import Vuex from 'vuex'
    import axios from 'axios'
    
    Vue.use(Vuex)
    
    export const createStore = () => {
      return new Vuex.Store({
        state: () => {
          return {
            posts: []
          }
        },
        mutations: {
          setPosts (state, posts) {
            state.posts = posts
          }
        },
        actions: {
          // 在服务端渲染期间务必让 action 返回一个 Promise
          async getPosts ({ commit }) {
            const { data } = await axios({
              method: 'GET',
              url: 'https://cnodejs.org/api/v1/topics'
            })
            commit('setPosts', data.data)
          }
        }
      })
    }
    ```

- 将 store 挂载到根实例

  - app.js

    ```
    import Vue from 'vue'
    import App from './App.vue'
    import { createRouter } from '@/router'
    import VueMeta from 'vue-meta'
    import { createStore } from '@/store'
    
    Vue.use(VueMeta)
    
    Vue.mixin({
      metaInfo: {
        titleTemplate: '%s - 拉勾教育'
      }
    })
    
    export function createApp () {
      const router = createRouter()
      const store = createStore()
      const app = new Vue({
        router,
        store,
        render: h => h(App)
      })
      return { app, router, store }
    }
    ```

- 在 pages 中创建 Posts.vue

  ```
  <template>
    <div>
      <h1>Post List</h1>
      <ul>
        <li v-for="post in posts" :key="post.id">{{ post.title }}</li>
      </ul>
    </div>
  </template>
  
  <script>
  import { mapState, mapActions } from 'vuex'
  export default {
    name: 'Posts',
    metaInfo: {
      title: 'Posts'
    },
    computed: {
      ...mapState(['posts'])
    },
    methods: {
      ...mapActions(['getPosts'])
    },
    // Vue SSR 特殊为服务端渲染提供的一个声明周期钩子函数
    serverPrefetch () {
      return this.getPosts()
    }
  }
  </script>
  
  <style>
  
  </style>
  ```

- entry-client.js

  ```
  import { createApp } from './app'
  
  const { app, router, store } = createApp()
  
  if (window.__INITIAL_STATE__) {
    store.replaceState(window.__INITIAL_STATE__)
  }
  
  router.onReady(() => {
    app.$mount('#app')
  })
  ```

- entry-server.js

  ```
  import { createApp } from './app'
  
  export default async context => {
    const { app, router, store } = createApp()
  
    const meta = app.$meta()
  
    // 设置服务端 router 的位置
    router.push(context.url)
  
    // 设置 meta 信息
    context.meta = meta
  
    // 等到 router 将可能的异步组件和钩子函数解析完
    await new Promise(router.onReady.bind(router))
  
    // 服务端渲染完毕之后被调用
    context.rendered = () => {
      // Renderer 会把 context.state 数据对象内联到页面模板中
      // 最终发送给客户端的页面中会包含一段脚本，window.__INITIAL_STATE__ = context.state
      // 客户端就要把页面中的 window.__INITIAL_STATE__ 拿出来填充客户端 store 容器中
      context.state = store.state
    }
  
    return app
  }
  
  ```


### 二. 静态站点生成

#### 1. Gridsome 基础

- 介绍
  - Gridsome 是什么
    - 一个免费，开源，基于 Vue.js 技术栈的静态网站生成器
    - 官方网站：https://gridsome.org/
    - Github：https://github.com/gridsome/gridsome
  - 什么是静态网站生成器
    - 静态网站生成器是使用一系列配置，模板以及数据，生成静态 HTML 文件及相关资源的工具
    - 这个功能也叫预渲染
    - 生成的网站不需要类似 PHP 这样的服务器
    - 只需要放到支持静态资源的 Web Server 或 CDN 上即可
  - 静态网站的好处
    - 省钱
      - 不需要专业的服务器，只要能托管静态文件的空间即可
    - 快速
      - 不经过后端服务器的处理，只传输内容
    - 安全
      - 没有后端程序的执行，自然会更安全
  - 静态应用的使用场景
    - 不适合有大量路由页面的应用
      - 如果您的站点有成百上千条路由页面，则预渲染将会非常缓慢。当然，您每次更新只需要做一次，但是可能要花费一些时间。大多数人不会最终获得千条静态路由页面，而只是以防万一
    - 不适合有大量动态内容的应用
      - 如果渲染路线中包含特定于用户查看其内容或其他动态资源的内容，则应确保您具有可以显示的占位符组件，直到动态内容加载到客户端为止，否则可能有点怪异

#### 2. 创建 Gridsome 项目

- 全局安装 gridsome，npm install --global @gridsome/cli
- 创建 gridsome 项目，gridsome create <项目名>
  - gridsome 依赖一个特殊的第三方模块 sharp（https://github.com/lovell/sharp），用于处理图片
  - 这个包很难安装成功，sharp 包含 c++ 文件，需要进行编译才可以正常安装，所以需要有一个 c++ 编译环境，sharp 依赖一个包（libvips），libvips 这个包比较大，有几十兆，由于国内网络原因，很难下载
  - 配置 sharp 镜像
    - npm config set sharp_binary_host "https://npm.taobao.org/mirrors/sharp"
    - npm config set sharp_libvips_binary_host "https://npm.taobao.org/mirrors/sharp_libvips"
  - 安装 c++ 环境
    - npm install node-gyp -g
    - windows 下配置 node-gyp 依赖包
      - npm install --global windows-build-tools

#### 3. 预渲染

- 服务端渲染首屏直出页面
- 客户端接管服务端渲染好的页面，转换为单页面应用

#### 4. 目录结构

```
|- .chche // 打包过程中缓存的目录
|- dist // 打包生成的目录
|- src
	|- .temp // 打包编译时，自动生成的目录
	|- components // 公共组件
	|- layouts // 布局组件
	|- pages // 路由页面
	|- templates // 集合的节点
	|- favicon.png // 图标
	|- main.js // 项目的启动入口
|- static // 静态资源
|- .gitignore // git 的忽略项
|- gridsome.config.js // gridsome 的配置文件
|- gridsome.server.js // gridsome 服务端配置文件

```

#### 5. 项目配置

- https://www.gridsome.cn/docs/config/

#### 6. pages

- https://www.gridsome.cn/docs/pages/

- 通过文件系统创建

  - src/pages/Index.vue => / // 根目录
  - src/pages/AboutAs.vue => /about-us/ // about-us 页面
  - src/pages/about/Vision.vue => /about/vision/ // about/vision 页面
  - src/pages/blog/Index.vue => /blog/ // blog 页面
  - src/pages/user/[id].vue => /user/:id // 动态路由，user 页面
  - src/pages/user/[id]/Settings.vue => /user/id/settings // 动态路由 settings 页面

- 通过编程（api）的方式创建

  - gridsome-srver.js

    ```
    // Server API makes it possible to hook into various parts of Gridsome
    // on server-side and add custom data to the GraphQL data layer.
    // Learn more: https://gridsome.org/docs/server-api/
    
    // Changes here require a server restart.
    // To restart press CTRL + C in terminal and run `gridsome develop`
    
    module.exports = function (api) {
      api.loadSource(({ addCollection }) => {
        // Use the Data Store API here: https://gridsome.org/docs/data-store-api/
      })
    
      api.createPages(({ createPage }) => {
        // Use the Pages API here: https://gridsome.org/docs/pages-api/
        createPage({
          path: '/my-page',
          component: './src/templates/MyPage.vue'
        })
      })
    }
    
    ```

#### 7. 集合

- 作用

  - 重载数据
  - 预渲染页面

- 使用

  - 集合

    ```
    // Server API makes it possible to hook into various parts of Gridsome
    // on server-side and add custom data to the GraphQL data layer.
    // Learn more: https://gridsome.org/docs/server-api/
    
    const axios = require("axios")
    
    // Changes here require a server restart.
    // To restart press CTRL + C in terminal and run `gridsome develop`
    
    module.exports = function (api) {
      api.loadSource(async ({ addCollection }) => {
        // Use the Data Store API here: https://gridsome.org/docs/data-store-api/
        const collection = addCollection('Post')
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/posts')
        for (const item of data) {
          collection.addNode({
            id: item.id,
            title: item.title,
            body: item.body
          })
        }
      })
    
      api.createPages(({ createPage }) => {
        // Use the Pages API here: https://gridsome.org/docs/pages-api/
      })
    }
    
    ```

  - 在 GraphQL 中查询数据

    - 在工具中查询

      - 打开 http://localhost:8080/___explore，输入查询规则查询数据

    - 在页面中查询

      ```
      <template>
        <Layout>
          <div>
            <h1>Posts 2</h1>
            <ul>
              <li v-for="edge in $page.posts.edges" :key="edge.node.id">
                <g-link to="/">{{ edge.node.title }}</g-link>
              </li>
            </ul>
          </div>
        </Layout>
      </template>
      
      <page-query>
        query {
          posts: allPost {
            edges {
              node {
                id
                title
              }
            }
          }
        }
      </page-query>
      // 组件页面用 <static-query></static-query>
      
      <script>
      export default {
        name: 'Posts2'
      }
      </script>
      
      <style>
      
      </style>
      ```

#### 8. 使用模板渲染节点页面

- 在 templates 目录新建 Post.vue

- Post.vue

  ```
  <template>
    <Layout>
      <div>
        <h1>{{ $page.post.title }}</h1>
        <p>{{ $page.post.body }}</p>
      </div>
    </Layout>
  </template>
  
  <page-query>
  query ($id: ID!) {
    post (id: $id) {
      id
      title
      body
    }
  }
  </page-query>
  
  <script>
  export default {
    name: 'Post',
    metaInfo () {
      return {
        title: this.$page.post.title
      }
    }
  }
  </script>
  
  <style>
  
  </style>
  ```

- 在 gridsome.config.js 中配置模板

  ```
  // This is where project configuration and plugin options are located.
  // Learn more: https://gridsome.org/docs/config
  
  // Changes here require a server restart.
  // To restart press CTRL + C in terminal and run `gridsome develop`
  
  module.exports = {
    // 设置公共页面标题
    siteName: 'my-gridsome',
    siteDescription: 'gridsome',
    // 页面标题模板
    titleTemplate: '%s - my-gridsome',
    templates: {
      // 方式一，会直接从 templates 中找同名的 .vue 文件
      // Post: '/posts',
      // 方式二
      Post: [
        {
          path: '/posts/:id',
          component: './src/templates/Post.vue'
        }
      ]
    },
    plugins: []
  }
  
  ```

### 三. Vue.js 组件库

#### 1. 处理组件的边界情况



- $root

  - 通过 $root 可以访问到 Vue 的根实例，进而操作根实例中的成员，在小型的只有少量组件的应用中，可以在 Vue 根实例里存储共享数据

  - 根实例

    ```
    import Vue from 'vue'
    import App from './App.vue'
    import router from './router'
    import store from './store'
    
    Vue.config.productionTip = false
    
    new Vue({
      router,
      store,
      render: h => h(App),
      data: {
        title: '根实例 - Root'
      },
      methods: {
        handle () {
          console.log(this.title)
        }
      }
    }).$mount('#app')
    
    ```

  - Root.vue

    ```
    <template>
      <div>
        <!-- 小型应用中可以在 Vue 的根实例存储共享数据
        组件中可以通过 $root 访问根实例 -->
        <h1>01 - $root</h1>
        <p>$root.title: {{ $root.title }}</p>
        <p>
          <button @click="$root.handle">获取 title</button>
          <button @click="$root.title = 'Hello $root'">改变 title</button>
        </p>
      </div>
    </template>
    
    <script>
    export default {
      name: 'Root'
    }
    </script>
    
    <style>
    
    </style>
    ```

    

- $parent

  - 通过 $parent 获取父组件

    - parent.vue

      ```
      <template>
        <div>
          <h1>02 - parent</h1>
          <child />
        </div>
      </template>
      
      <script>
      import Child from './Child'
      export default {
        name: 'Parent',
        components: {
          Child
        },
        data () {
          return {
            title: '获取父组件实例'
          }
        },
        methods: {
          handle () {
            console.log(this.title)
          }
        }
      }
      </script>
      
      <style>
      
      </style>
      ```

      

    - Child.vue

      ```
      <template>
        <div>
          <h1>02 - child</h1>
          <p>$parent.title: {{ $parent.title }}</p>
          <p>
            <button @click="$parent.handle">获取 $parent.title</button>
            <button @click="$parent.title = 'Hello $parent.title'">改变 $parent.title</button>
          </p>
          <grandson />
        </div>
      </template>
      
      <script>
      import Grandson from './Grandson'
      export default {
       name: 'Child',
       components: {
         Grandson
       }
      }
      </script>
      
      <style>
      
      </style>
      ```

      

    - Grandson.vue

      ```
      <template>
        <div>
          <h1>02 - grandson</h1>
          <p>$parent.$parent.title: {{ $parent.$parent.title }}</p>
          <p>
            <button @click="$parent.$parent.handle">获取 $parent.$parent.title</button>
            <button @click="$parent.$parent.title = 'Hello $parent.$parent.title'">改变 $parent.$parent.title</button>
          </p>
        </div>
      </template>
      
      <script>
      export default {
        name: 'Grandson'
      }
      </script>
      
      <style>
      
      </style>
      ```

      

  - 通过 $children 获取子组件

    - Parent.vue

      ```
      <template>
        <div>
          <h1>03 - parent</h1>
          <child1 />
          <child2 />
          <p>
            <button @click="getChild">获取子组件</button>
          </p>
        </div>
      </template>
      
      <script>
      import Child1 from './Child1'
      import Child2 from './Child2'
      export default {
        name: 'Parent',
        components: {
          Child1,
          Child2
        },
        methods: {
          getChild () {
            console.log(this.$children)
            console.log(this.$children[0].title)
            console.log(this.$children[1].title)
      
            this.$children[0].handle()
            this.$children[1].handle()
          }
        }
      }
      </script>
      
      <style>
      
      </style>
      ```

    - Child1.vue

      ```
      <template>
        <div>
          <h1>03 - child1</h1>
        </div>
      </template>
      
      <script>
      export default {
        name: 'Child1',
        data () {
          return {
            title: 'Child1 获取子组件 - title'
          }
        },
        methods: {
          handle () {
            console.log(this.title)
          }
        }
      }
      </script>
      
      <style>
      
      </style>
      ```

    - Child2.vue

      ```
      <template>
        <div>
          <h1>03 - child1</h1>
        </div>
      </template>
      
      <script>
      export default {
        name: 'Child1',
        data () {
          return {
            title: 'Child2 获取子组件 - title'
          }
        },
        methods: {
          handle () {
            console.log(this.title)
          }
        }
      }
      </script>
      
      <style>
      
      </style>
      ```

      

  - 操作父组件/子组件成员，可以替换 prop 使用，不推荐使用 $parnet/$children，如果通过这种方式改变了父组件/子组件中的成员，出现问题就很难找到更改的位置

- $refs

  - 通过 $refs 可以访问子组件中成员，开发自定义组件时会用到

  - Parnet.vue

    ```
    <template>
      <div>
        <h1>04 - parent</h1>
        <my-input ref="myTxt" />
        <p>
          <button @click="focus">获取焦点</button>
        </p>
      </div>
    </template>
    
    <script>
    import MyInput from './MyInput'
    export default {
      name: 'Parent',
      components: {
        MyInput
      },
      methods: {
        focus () {
          this.$refs.myTxt.focus()
          this.$refs.myTxt.value = 'Hello'
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

  - MyInput.vue

    ```
    <template>
      <div>
        <h1>04 - my-input</h1>
        <p>
          <input v-model="value" type="text" ref="txt">
        </p>
      </div>
    </template>
    
    <script>
    export default {
      name: 'MyInput',
      data () {
        return {
          value: 'default'
        }
      },
      methods: {
        focus () {
          this.$refs.txt.focus()
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

    

- 依赖注入 provide/inject

  - 如果组件嵌套比较多的话，用 $parent 获取父组件成员就会比较麻烦，这时就可以使用 provide/inject

  - 通过依赖注入的方式，注入到子组件的成员不是响应式的，应避免修改

  - Parent.vue

    ```
    <template>
      <div>
        <h1>05 - parent</h1>
        <child />
      </div>
    </template>
    
    <script>
    import Child from './Child'
    export default {
      name: 'Parent',
      components: {
        Child
      },
      provide () {
        return {
          title: this.title,
          handle: this.handle
        }
      },
      data () {
        return {
          title: '父组件 provide'
        }
      },
      methods: {
        handle () {
          console.log(this.title)
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

  - Child.vue

    ```
    <template>
      <div>
        <h1>05 - child</h1>
        <p>$parent.title: {{ $parent.title }}</p>
        <p>
          <button @click="handle">获取 title</button>
          <button @click="title = 'Hello xxx'">改变 title</button>
        </p>
        <grandson />
      </div>
    </template>
    
    <script>
    import Grandson from './Grandson'
    export default {
     name: 'Child',
     components: {
       Grandson
     },
     inject: ['title', 'handle']
    }
    </script>
    
    <style>
    
    </style>
    ```

  - Grandson.vue

    ```
    <template>
      <div>
        <h1>05 - grandson</h1>
        <p>$parent.$parent.title: {{ title }}</p>
        <p>
          <button @click="handle">获取 title</button>
          <button @click="title = 'Hello yyy'">改变 title</button>
        </p>
      </div>
    </template>
    
    <script>
    export default {
      name: 'Grandson',
      inject: ['title', 'handle']
    }
    </script>
    
    <style>
    
    </style>
    ```

- $attrs/$listeners

  - $attrs

    - 把父组件中非 prop 属性绑定到内部组件

  - $listeners

    - 把父组件中的 DOM 对象的原生事件绑定到内部组件

  - Parent.vue

    ```
    <template>
      <div>
        <h1>06 - parent</h1>
        <!-- <my-input required placeholder="Enter your username" class="theme-dark" data-test="test" /> -->
        <my-input required placeholder="Enter your username" class="theme-dark" @focus="onFocus" @input="onInput" data-test="test" />
      </div>
    </template>
    
    <script>
    import MyInput from './MyInput'
    export default {
      name: 'Parent',
      components: {
        MyInput
      },
      methods: {
        onFocus (e) {
          console.log(e)
        },
        onInput (e) {
          console.log(e.target.value)
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    ```

  - MyInput.vue

    ```
    <template>
      <!-- 1. 从父组件传给子组件的属性，如果没有 prop 接收
        会自动设置到子组件内部的最外层标签上
        如果是 class 和 style 的话，会合并最外层的 class 和 style -->
      <!-- <input type="text" class="form-control" :placeholder="placeholder"> -->
    
      <!-- 2. 如果子组件中不想继承父组件传入的非 prop 属性，可以使用 inheritAttrs 禁用继承
        然后通过 v-bind="$attrs" 把外部组件传入的非 prop 属性设置给希望的标签上
        但是这不会改变 class 和 style -->
        <!-- <div>
          <input type="text" v-bind="$attrs" class="form-control">
        </div> -->
        <!-- 3. 注册事件 -->
        <!-- <div>
          <input type="text" v-bind="$attrs" class="form-control" @focus="$emit('focus', $event)" @input="$emit('input', $event)">
        </div> -->
        <!-- 4. $listeners -->
        <div>
          <input type="text" v-bind="$attrs" class="form-control" v-on="$listeners">
        </div>
    </template>
    
    <script>
    export default {
      name: 'MyInput',
      // props: {
      //   placeholder: {
      //     type: String
      //   }
      // },
      inheritAttrs: false
    }
    </script>
    
    <style>
    
    </style>
    ```

#### 2. 快速原型开发

- Vue-Cli 中提供了一个插件可以进行原型快速开发

- 需要先额外安装一个全局的扩展

  - npm install @vue/cli-service-global -g

- 使用 vue serve 快速查看组件运行效果

  - vue serve 如果不指定参数默认会在当前目录找以下的入口文件
    - main.js，index.js，App.vue，app.vue
  - 可以指定要加载的组件
    - vue serve ./src/login.vue

- 结合 Element UI

  - 安装和配置

    - 初始化 package.json

      - npm init

    - 安装 Element UI

      - vue add element

    - 加载 Element UI，使用 Vue.use() 安装插件

      - main.js

      ```
      import Vue from 'vue'
      import ElementUI from 'element-ui'
      import 'element-ui/lib/theme-chalk/index.css'
      import Login from './src/Login.vue'
      
      Vue.use(ElementUI)
      
      new Vue({
        render: h => h(Login)
      }).$mount('#app')
      
      ```

      

  - 登录页面

    - Login.vue

      ```
      <template>
        <el-form class="form" ref="form" :model="user" :rules="rules">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="user.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="user.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="login">登 录</el-button>
          </el-form-item>
        </el-form>
      </template>
      
      <script>
      export default {
        name: 'Login',
        data () {
          return {
            user: {
              username: '',
              password: ''
            },
            rules: {
              username: [
                {
                  required: true,
                  message: '请输入用户名'
                }
              ],
              password: [
                {
                  required: true,
                  message: '请输入密码'
                },
                {
                  min: 6,
                  max: 12,
                  message: '请输入6-12位密码'
                }
              ]
            }
          }
        },
        methods: {
          login () {
            this.$refs.form.validate(valid => {
              if (valid) {
                alert('验证成功')
              } else {
                alert('验证失败')
                return false
              }
            })
          }
        }
      }
      </script>
      
      <style>
        .form {
          width: 30%;
          margin: 150px auto;
        }
      </style>
      
      ```

#### 3. 组件开发

- 组件的分类

  - 第三方组件
    - element ui，iview
  - 基础组件
    - 文本框，按钮
  - 业务组件
    - 结合特定的行业使用场景的组件，可以根据用户的行为输出特定的界面展示给用户

- 步骤条组件

  - steps.css

    ```
    .lg-steps {
      position: relative;
      display: flex;
      justify-content: space-between;
    }
    
    .lg-steps-line {
      position: absolute;
      height: 2px;
      top: 50%;
      left: 24px;
      right: 24px;
      transform: translateY(-50%);
      z-index: 1;
      background: rgb(223, 231, 239);
    }
    
    .lg-step {
      border: 2px solid;
      border-radius: 50%;
      height: 32px;
      width: 32px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-weight: 700;
      z-index: 2;
      background-color: white;
      box-sizing: border-box;
    }
    
    ```

  - Steps.vue

    ```
    <template>
      <div class="lg-steps">
        <div class="lg-steps-line"></div>
        <div v-for="index in count" :key="index" class="lg-step" :style="{color: active >= index ? activeColor : defaultColor}">{{ index }}</div>
      </div>
    </template>
    
    <script>
    import './steps.css'
    export default {
      name: 'Steps',
      props: {
        count: {
          type: Number,
          default: 3
        },
        active: {
          type: Number,
          default: 0
        },
        activeColor: {
          type: String,
          default: 'red'
        },
        defaultColor: {
          type: String,
          default: 'green'
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    
    ```

- 表单组件

  - Form.vue

    ```
    <template>
      <form>
        <slot />
      </form>
    </template>
    
    <script>
    export default {
      name: 'LgForm',
      provide () {
        return {
          form: this
        }
      },
      props: {
        model: {
          type: Object
        },
        rules: {
          type: Object
        }
      },
      methods: {
        validate (callback) {
          // tasks 中存储的是 validate 返回的 Promise 对象
          const tasks = this.$children.filter(item => item.prop).map(item => item.validate())
          Promise.all(tasks).then(() => callback(true)).catch(() => callback(false))
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    
    ```

  - FromItem.vue

    - 安装 async-validator，npm i async-validator

      ```
      <template>
        <div>
          <label>{{ label }}</label>
          <div>
            <slot />
            <p v-if="errMessage">{{ errMessage }}</p>
          </div>
        </div>
      </template>
      
      <script>
      import AsyncValidator from 'async-validator'
      
      export default {
        name: 'LgFormItem',
        inject: ['form'],
        props: {
          label: {
            type: String
          },
          prop: {
            type: String
          }
        },
        data () {
          return {
            errMessage: ''
          }
        },
        methods: {
          validate () {
            if (!this.prop) return
            const value = this.form.model[this.prop]
            const rules = this.form.rules[this.prop]
            const discriptor = {
              [this.prop]: rules
            }
            const validator = new AsyncValidator(discriptor)
            return validator.validate({[this.prop]: value}, errors => {
              if (errors) {
                this.errMessage = errors[0].message
              } else {
                this.errMessage = ''
              }
            })
          }
        },
        mounted () {
          this.$on('validate', () => {
            this.validate()
          })
        }
      }
      </script>
      
      <style>
      
      </style>
      
      ```

  - FormInput.vue

    ```
    <template>
      <div>
        <input :type="type" v-bind="$attrs" :value="value" @input="handleInput">
      </div>
    </template>
    
    <script>
    export default {
      name: 'LgInput',
      inheritAttrs: false,
      props: {
        value: {
          type: String
        },
        type: {
          type: String,
          default: 'text'
        }
      },
      methods: {
        handleInput (e) {
          this.$emit('input', e.target.value)
          const findParent = parent => {
            while (parent) {
              if (parent.$options.name === 'LgFormItem') {
                break
              } else {
                parent = parent.$parent
              }
            }
            return parent
          }
          const parent = findParent(this.$parent)
          if (parent) parent.$emit('validate') 
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    
    ```

  - FormButton.vue

    ```
    <template>
      <div>
        <button @click="handleClick">
          <slot />
        </button>
      </div>
    </template>
    
    <script>
    export default {
      name: 'LgButton',
      props: {
        type: {
          type: String
        }
      },
      methods: {
        handleClick (e) {
          this.$emit('click', e)
          e.preventDefault()
        }
      }
    }
    </script>
    
    <style>
    
    </style>
    
    ```

#### 4. Monorepo

- 两种项目的组织方式

  - Multirepo (Multiple Repository)

    - 每一个包对应一个项目

  - Monorepo (Monolithic Repository)

    - 一个项目仓库中管理多个模块/包

    - 目录结构

      ```
      |- __test__ // 存放测试文件
      |- dist // 存放打包后文件
      |- src // 存放源码
      |- index.js // 打包入口文件
      |- LICENSE // 版权信息
      |- README.md // 组件文档
      |- package.json // 配置文件
      
      ```


#### 5. Storybook

- 概念

  - 可视化的组件展示平台
  - 在隔离的开发环境中，以交互的方式展示组件
  - Storybook 在主程序之外运行，可以独立开发组件，不必担心应用程序特定的依赖关系
  - 支持的框架
    - React，React Native，Vue，Angular
    - Ember，HTML，Svelte，Mithril，Riot
  - 支持很多插件，并提供灵活的 API，可以根据需要自定义 Storybook，还可以构建 Storybook 的静态版本，并将其部署到 HTTP 服务器

- 安装

  - 自动安装
    - npx -p @storybook/cli sb init --type vue
    - yarn add vue
    - vue yarn add vue-loader vue-template-compiler --dev

- 使用

  - 在根目录新建 packages 目录

  - 修改 .storybook 目录中的 main.js 中 stories 的路径指向

    ```
    module.exports = {
      "stories": [
        "../packages/**/*.stories.mdx",
        "../packages/**/*.stories.@(js|jsx|ts|tsx)"
      ],
      "addons": [
        "@storybook/addon-links",
        "@storybook/addon-essentials"
      ]
    }
    ```

  - 在 packages 目录中按照 monorepo 的目录结构创建组件并编写组件

  - 在编写好的组件目录根目录新建 stories 目录

  - 在 stories 目录下新建 组件名.stories.js，以 input 组件为例

    ```
    import LgInput from '../'
    
    export default {
      title: 'LgInput',
      component: LgInput
    }
    
    export const Text = () => ({
      components: { LgInput },
      template: '<lg-input v-model="value" />',
      data () {
        return {
          value: 'admin'
        }
      }
    })
    
    export const Password = () => ({
      components: { LgInput },
      template: '<lg-input type="password" v-model="value"  />',
      data () {
        return {
          value: 'admin'
        }
      }
    })
    ```

#### 6. workspaces

- 开启 yarn 的工作区

  - package.json

    ```
    "private": true, // 防止根目录发布
    "workspaces": [ // 设置工作区中的子目录
    	"./packages/*"
    ]
    ```

- 使用方式

  - 给工作区根目录安装开发依赖
    - yarn add jest -D -W
  - 给指定工作区安装依赖
    - yarn workspace lg-button add lodash@4
  - 给所有工作区安装依赖
    - yarn install

#### 7. Lerna

- 概念
  - Lerna 是一个优化使用 git 和 npm 管理多仓库的工作流工具
  - 用于管理具有多个包的 JavaScript 项目
  - 它可以一键把代码提交到 git 和 npm 仓库
- 使用方式
  - 全局安装
    
    - yarn global add lerna
  - 初始化

    - lerna init

    - 在根目录生成 lerna.json 的配置文件

      ```
      {
        "packages": [
          "packages/*"
        ],
        "version": "0.0.1"
      }
      
      ```

    - 添加 .gitignore 文件，并将 node_modules 写入

    - 创建远程 github 仓库
  - 发布
    
    - lerna publish
    - 注意：必须是 npm 官方镜像源 http://registry.npmjs.org/

#### 8. 组件单元测试

- 单元测试概述

  - 使用断言的方式对一个函数的输入和输出进行测试，根据输入判断实际的输出和预测的输出是否相同，使用单元测试的目的是用来发现模块内部可能存在的各种错误

- 组件的单元测试概述

  - 使用单元测试工具对组件的各种状态和行为进行测试，确保组件发布之后在项目中使用组件的过程中不会导致程序出现错误

- 好处

  - 提供描述组件行为的文档
  - 节省手动测试的时间
  - 减少研发新特性时产生的 bug
  - 改进设计
  - 促进重构

- 配置组件单元测试环境

  - 安装依赖

    - Vue Test Utils，Vue 官方提供的组件单元测试的官方库
      - 常用 API
        - mount()
          - 创建一个包含被挂载和渲染的 Vue 组件的 wrapper
        - wrapper
          - vm wrapper 包裹的组件实例
        - props() 返回 Vue 实例选项中的 props 对象
          - html() 组件生成的 HTML 标签
        - find() 通过选择器返回匹配到组件中的 DOM 元素
          - trigger() 触发 DOM 原生事件，自定义事件 wrapper.vm.$emit()
  - Jest，facebook 的单元测试框架
      - 常用 API
        - 全局函数
          - describe(name, fn) 把相关测试组合在一起
          - test(name, fn) 测试方法
          - expect(value) 断言
      - 匹配器
          - toBe(value) 判断值是否相等
        - toEqual(obj) 判断对象是否相等
          - toContain(value) 判断数组或者字符串中是否包含
      - 快照
          - toMatchSnapshot()
    - vue-jest，Vue 官方提供的为 jest 服务的预处理器
    - babel-jest，对测试代码进行降级处理
    - 安装
      - yarn add jest @vue/test-utils vue-jest babel-jest -D -W
  
  - 配置测试脚本
  
    - package.json
  
      ```
      "scripts": {
      	"test": "jest"
      }
      ```
  
  - Jest 配置文件

    - jest.config.js

      ```
    module.exports = {
      	"testMatch": ["**/__tests__/**/*.[jt]s?(x)"],
    	"moduleFileExtensions": [
      		"js",
      		"json",
      		// 告诉 Jest 处理 *.vue 文件
      		"vue"
      	],
      	"transform": {
      		// 用 vue-jest 处理 *.vue 文件
    		".*\\.(vue)$": "vue-jest",
      		// 用 babel-jest 处理 js
    		".*\\(js)$": "babel-jest"
      	}
      }
      ```
  
      
  
  - babel 配置文件
  
    - babel.config.js
  
      ```
      module.exports = {
      	presets: [
      		"@babel/preset-env"
      	]
      }
      ```
  
  - 配置 babel 桥接
  
    - 作用
      - 解决 storybook (依赖 babel 7) 和 jest (依赖 babel 6) 版本不同
    - yarn add babel-core@bridge -D -W