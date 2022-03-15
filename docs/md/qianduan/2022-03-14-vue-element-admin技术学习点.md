## 前言
> 作为后端工程师，目前接触前端最多的项目就是[PanJiaChen](https://github.com/PanJiaChen/vue-admin-template) 大佬写的vue-element-admin，但是又因前端基础过差，所以便想系统学习一下该项目，做好记录。


## 1.1 vue.config.js配置文件解析

```js
'use strict'

// 导入nodejs 的path模块
const path = require('path')
const defaultSettings = require('./src/settings.js')

function resolve(dir) {
  // 函数文档 http://nodejs.cn/api/path.html#pathjoinpaths
  // __dirname 返回当前js文件所在的绝对路径 path.join()作用为拼接路径
  return path.join(__dirname, dir)
}

const name = defaultSettings.title || 'vue Admin Template' // 配置网页title

// If your port is set to 80,
// use administrator privileges to execute the command line.
// For example, Mac: sudo npm run
// You can change the port by the following methods:
// port = 9528 npm run dev OR npm run dev --port = 9528
const port = process.env.port || process.env.npm_config_port || 9528 // dev port

// All configuration item explanations can be find in https://cli.vuejs.org/config/
module.exports = {
  /**
   * You will need to set publicPath if you plan to deploy your site under a sub path,
   * for example GitHub Pages. If you plan to deploy your site to https://foo.github.io/bar/,
   * then publicPath should be set to "/bar/".
   * In most cases please use '/' !!!
   * Detail: https://cli.vuejs.org/config/#publicpath
   */
  publicPath: '/',
  outputDir: 'dist',
  assetsDir: 'static',
  // lintOnSave: process.env.NODE_ENV === 'development',
  lintOnSave: false,
  // https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html source map详解
  productionSourceMap: false,
  devServer: {
    port: port,
    // 启动后打开默认浏览器
    open: true,
    // 当出现编译错误或警告时，在浏览器中显示全屏覆盖。
    overlay: {
      warnings: false,
      errors: true
    },
    // 返回Mock数据 且可以看到network请求
    before: require('./mock/mock-server.js')
  },
  configureWebpack: {
    // 在webpack的name字段中提供应用的标题 这样它就可以在index.html中被访问以注入正确的标题
    // vue.config.js中无法直接配置name属性 configureWebpack中配置后，会被wabpack-merge合并到最终的配置中
    name: name,
    // 创建 import 或 require 的别名，来确保模块引入变得更简单。例如，一些位于 src/ 文件夹下的常用模块：
    resolve: {
      alias: {
        '@': resolve('src')
      }
    }
  },
  // 分包策略配置
  chainWebpack(config) {
    // it can improve the speed of the first screen, it is recommended to turn on preload
    config.plugin('preload').tap(() => [
      {
        rel: 'preload',
        // to ignore runtime.js
        // https://github.com/vuejs/vue-cli/blob/dev/packages/@vue/cli-service/lib/config/app.js#L171
        fileBlacklist: [/\.map$/, /hot-update\.js$/, /runtime\..*\.js$/],
        include: 'initial'
      }
    ])

    // when there are many pages, it will cause too many meaningless requests
    // 当页面过多时，会导致无意义的请求过多
    config.plugins.delete('prefetch')

    // set svg-sprite-loader
    config.module
      .rule('svg')
      .exclude.add(resolve('src/icons'))
      .end()
    config.module
      .rule('icons')
      .test(/\.svg$/)
      .include.add(resolve('src/icons'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
      .end()

    config
      .when(process.env.NODE_ENV !== 'development',
        config => {
          config
            .plugin('ScriptExtHtmlWebpackPlugin')
            .after('html')
            .use('script-ext-html-webpack-plugin', [{
            // `runtime` must same as runtimeChunk name. default is `runtime`
              inline: /runtime\..*\.js$/
            }])
            .end()
          config
            .optimization.splitChunks({
              chunks: 'all',
              cacheGroups: {
                libs: {
                  name: 'chunk-libs',
                  test: /[\\/]node_modules[\\/]/,
                  priority: 10,
                  chunks: 'initial' // only package third parties that are initially dependent
                },
                elementUI: {
                  name: 'chunk-elementUI', // split elementUI into a single package
                  priority: 20, // the weight needs to be larger than libs and app or it will be packaged into libs or app
                  test: /[\\/]node_modules[\\/]_?element-ui(.*)/ // in order to adapt to cnpm
                },
                commons: {
                  name: 'chunk-commons',
                  test: resolve('src/components'), // can customize your rules
                  minChunks: 3, //  minimum common number
                  priority: 5,
                  reuseExistingChunk: true
                }
              }
            })
          // https:// webpack.js.org/configuration/optimization/#optimizationruntimechunk
          config.optimization.runtimeChunk('single')
        }
      )
  }
}

```

- [publicPath](https://cli.vuejs.org/zh/config/#publicpath)
> 默认情况下，Vue CLI 会假设你的应用是被部署在一个域名的根路径上，例如 https://www.my-app.com/。 如果应用被部署在一个子路径上，你就需要用这个选项指定这个子路径。例如，如果你的应用被部署在 https://www.my-app.com/my-app/， 则设置 publicPath 为 /my-app/。
- [outputDir](https://cli.vuejs.org/zh/config/#outputdir) 
> 默认为 'dist'，当运行 vue-cli-service build 时生成的生产环境构建文件的目录。注意目标目录的内容在构建之前会被清除 (构建时传入 --no-clean 可关闭该行为)  
> 提示：请始终使用 **outputDir** 而不要修改 webpack 的**output.path**。
- [assetsDir](https://cli.vuejs.org/zh/config/#assetsdir) 
> 默认为 '', 放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录。  
> 提示: 从生成的资源覆写 filename 或 chunkFilename 时，assetsDir 会被忽略。
- [lintOnSave](https://cli.vuejs.org/zh/config/#lintonsave) 
> 可设置项为 boolean | 'warning' | 'default' | 'error'，默认为'default'，配置可与当前开发环境挂钩，如  **process.env.NODE_ENV === 'development'**
- [productionSourceMap](https://cli.vuejs.org/zh/config/#productionsourcemap)
> 是否构建生产环境的source map，source map可参考[阮一峰老师文章](https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
- [devServer](https://cli.vuejs.org/zh/config/#devserver)
    - port: string | number  配置端口
    - open: boolean 项目启动后是否通过默认浏览器请求项目
    - overlay: object 当出现编译错误或警告时，在浏览器中显示全屏覆盖。
    ```js
    overlay: {
        warnings: false,
        errors: true
    }
    ```
    - before: require('./mock/mock-server.js')  返回Mock数据 且可以看到network请求
    
- [configureWebpack](https://cli.vuejs.org/zh/config/#configurewebpack)
> 在webpack的name字段中提供应用的标题 这样它就可以在index.html中被访问以注入正确的标题  
> vue.config.js中无法直接配置name属性 configureWebpack中配置后，会被wabpack-merge合并到最终的配置中  
> resolve.alias 创建 import 或 require 的别名，来确保模块引入变得更简单。例如，一些位于 src/ 文件夹下的常用模块：  
> 常用的如 '@': resolve('src')，此时导入js模块或vue组件即可使用'@/views/components'
- [chainWebpack](https://cli.vuejs.org/zh/config/#chainwebpack)
> Type: Function  
> 是一个函数，会接收一个基于 webpack-chain 的 ChainableConfig 实例。允许对内部的 webpack 配置进行更细粒度的修改。  
> 更多细节可查阅：[配合 webpack > 链式操作](https://cli.vuejs.org/zh/guide/webpack.html#%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C-%E9%AB%98%E7%BA%A7)  
> 
> tips：相对于其他配置，对于本人而言最难理解的就是PanJiaChen大佬在此配置项下对webpack分包策略的优化，由于对webpack没有详细的了解，所以此处引用掘金[@馒头大大](https://juejin.cn/user/2946346893977262) 的文章--[20220215 | 学习vue-element-admin中的webpack优化](https://juejin.cn/post/7065316192684752927)

至此，vue.config.js配置文件项解读完成。
