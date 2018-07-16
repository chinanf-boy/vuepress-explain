## vuepress dev

``` js
// bin/vuepress.js
const { dev, build, eject } = require('../lib')
```

启动 开发服务器

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [lib/dev.js](#libdevjs)
  - [dev 是 `async/await` 函数](#dev-%E6%98%AF-asyncawait-%E5%87%BD%E6%95%B0)
  - [require](#require)
  - [dev的准备工作](#dev%E7%9A%84%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
  - [定义 重新准备 的函数](#%E5%AE%9A%E4%B9%89-%E9%87%8D%E6%96%B0%E5%87%86%E5%A4%87-%E7%9A%84%E5%87%BD%E6%95%B0)
  - [设置文件变化后的操作](#%E8%AE%BE%E7%BD%AE%E6%96%87%E4%BB%B6%E5%8F%98%E5%8C%96%E5%90%8E%E7%9A%84%E6%93%8D%E4%BD%9C)
  - [确定webpack配置,且获得构建编译](#%E7%A1%AE%E5%AE%9Awebpack%E9%85%8D%E7%BD%AE%E4%B8%94%E8%8E%B7%E5%BE%97%E6%9E%84%E5%BB%BA%E7%BC%96%E8%AF%91)
  - [webpack-serve服务器配置](#webpack-serve%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AE)
  - [`lib/dev.js`的工具函数](#libdevjs%E7%9A%84%E5%B7%A5%E5%85%B7%E5%87%BD%E6%95%B0)
    - [检测 主机-host 与 端口-port](#%E6%A3%80%E6%B5%8B-%E4%B8%BB%E6%9C%BA-host-%E4%B8%8E-%E7%AB%AF%E5%8F%A3-port)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## lib/dev.js

###  dev 是 `async/await` 函数

``` js
module.exports = async function dev (sourceDir, cliOptions = {}) {
```

### require

``` js
  const fs = require('fs')
  const path = require('path')
  const chalk = require('chalk')
  const webpack = require('webpack') // 构建工具
  const chokidar = require('chokidar') // 观察文件的库
  const serve = require('webpack-serve') // 构建工具服务器
  const convert = require('koa-connect')
  const mount = require('koa-mount')
  const range = require('koa-range')
  const serveStatic = require('koa-static')
  const history = require('connect-history-api-fallback')

  const prepare = require('./prepare')
  const logger = require('./util/logger') // 日志记录
  const HeadPlugin = require('./webpack/HeadPlugin')
  const DevLogPlugin = require('./webpack/DevLogPlugin')
  const createClientConfig = require('./webpack/createClientConfig')
  const { applyUserWebpackConfig } = require('./util')
  const { frontmatterEmitter } = require('./webpack/markdownLoader')
```

### dev的准备工作

``` js
  logger.wait('\nExtracting site metadata...')
  const options = await prepare(sourceDir) // 准备

```

- [ ] [`prepare` Explain](./prepare.ex.md)

> 

### 定义 重新准备 的函数

``` js
  // setup watchers to update options and dynamically generated 
  // 设置 观察者 以 更新选项 和 动态生成的文件
  const update = () => {
    prepare(sourceDir).catch(err => {
      console.error(logger.error(chalk.red(err.stack), false))
    })
  }

```

### 设置文件变化后的操作

``` js
  // watch add/remove of files
  const pagesWatcher = chokidar.watch([
    '**/*.md',
    '.vuepress/components/**/*.vue'
  ], {
    cwd: sourceDir,
    ignored: '.vuepress/**/*.md',
    ignoreInitial: true
  })
  pagesWatcher.on('add', update)
  pagesWatcher.on('unlink', update)
  pagesWatcher.on('addDir', update)
  pagesWatcher.on('unlinkDir', update)

  // watch config file
  const configWatcher = chokidar.watch([
    '.vuepress/config.js',
    '.vuepress/config.yml',
    '.vuepress/config.toml'
  ], {
    cwd: sourceDir,
    ignoreInitial: true
  })
  configWatcher.on('change', update)

  // also listen for frontmatter changes from markdown files
  frontmatterEmitter.on('update', update)

```

### 确定webpack配置,且获得构建编译

``` js
  // resolve webpack config
  let config = createClientConfig(options, cliOptions)

  config
    .plugin('html')
    // using a fork of html-webpack-plugin to avoid it requiring webpack
    // internals from an incompatible version.
    .use(require('vuepress-html-webpack-plugin'), [{
      template: path.resolve(__dirname, 'app/index.dev.html')
    }])

  config
    .plugin('site-data')
    .use(HeadPlugin, [{
      tags: options.siteConfig.head || []
    }])

  const port = await resolvePort(cliOptions.port || options.siteConfig.port)
  const { host, displayHost } = await resolveHost(cliOptions.host || options.siteConfig.host)

  config
  .plugin('vuepress-log')
  .use(DevLogPlugin, [{
    port,
    displayHost,
    publicPath: options.publicPath
  }])

  config = config.toConfig()
  const userConfig = options.siteConfig.configureWebpack
  if (userConfig) {
    config = applyUserWebpackConfig(userConfig, config, false /* isServer */)
  }

  const compiler = webpack(config)

```

### webpack-serve服务器配置

``` js
  const nonExistentDir = path.resolve(__dirname, 'non-existent')
  await serve({
    // avoid project cwd from being served. Otherwise if the user has index.html
    // in cwd it would break the server
    content: [nonExistentDir],
    compiler,
    host,
    dev: { logLevel: 'warn' },
    hot: {
      port: port + 1,
      logLevel: 'error'
    },
    logLevel: 'error',
    port,
    add: app => {
      const userPublic = path.resolve(sourceDir, '.vuepress/public')

      // enable range request
      app.use(range)

      // respect base when serving static files...
      if (fs.existsSync(userPublic)) {
        app.use(mount(options.publicPath, serveStatic(userPublic)))
      }

      app.use(convert(history({
        rewrites: [
          { from: /\.html$/, to: '/' }
        ]
      })))
    }
  })
}

```

### `lib/dev.js`的工具函数

#### 检测 主机-host 与 端口-port

``` js
function resolveHost (host) {
  // webpack-serve hot updates doesn't work properly over 0.0.0.0 on Windows,
  // but localhost does not allow visiting over network :/
  const defaultHost = process.platform === 'win32' ? 'localhost' : '0.0.0.0'
  host = host || defaultHost
  const displayHost = host === defaultHost && process.platform !== 'win32'
    ? 'localhost'
    : host
  return {
    displayHost,
    host
  }
}

async function resolvePort (port) {
  const portfinder = require('portfinder')
  portfinder.basePort = parseInt(port) || 8080
  port = await portfinder.getPortPromise()
  return port
}
```