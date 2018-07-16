# Vuepress@0.12.0 [![explain]][source] 

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
    
「 Vue 驱动的静态网站生成器 」

[github source](https://github.com/vuejs/vuepress)

[中文](./readme.md) | ~~[english](./readme.en.md)~~

欢迎 `Issue` 和 `Pull` ❤️, 最好 `Pull` 👏

---

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [简单使用](#%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8)
- [package.json](#packagejson)
- [bin/vuepress.js](#binvuepressjs)
  - [命令行通用第一行](#%E5%91%BD%E4%BB%A4%E8%A1%8C%E9%80%9A%E7%94%A8%E7%AC%AC%E4%B8%80%E8%A1%8C)
  - [检查版本](#%E6%A3%80%E6%9F%A5%E7%89%88%E6%9C%AC)
  - [命令行分流](#%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%88%86%E6%B5%81)
    - [1. vuepress dev](#1-vuepress-dev)
    - [2. vuepress build](#2-vuepress-build)
    - [3. vuepress eject](#3-vuepress-eject)
    - [其他命令勘误 和 命令错误的输出](#%E5%85%B6%E4%BB%96%E5%91%BD%E4%BB%A4%E5%8B%98%E8%AF%AF-%E5%92%8C-%E5%91%BD%E4%BB%A4%E9%94%99%E8%AF%AF%E7%9A%84%E8%BE%93%E5%87%BA)
    - [包裹命令并且运行](#%E5%8C%85%E8%A3%B9%E5%91%BD%E4%BB%A4%E5%B9%B6%E4%B8%94%E8%BF%90%E8%A1%8C)
- [vurepss cli](#vurepss-cli)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## 简单使用

``` bash
# 安装
yarn global add vuepress # 或者：npm install -g vuepress

# 新建一个 markdown 文件
echo '# Hello VuePress!' > README.md

# 开始写作
vuepress dev .

# 构建静态文件
vuepress build .
```

可以看出,用户的使用 主要在 命令行-CLI 上

## package.json

``` json
  "main": "lib/index.js",
  "bin": {
    "vuepress": "bin/vuepress.js"
  },
  "scripts": {
    "dev": "node bin/vuepress dev docs",
    "build": "node bin/vuepress build docs",
    "lint": "eslint --fix --ext .js,.vue bin/ lib/ test/",
    "prepublishOnly": "conventional-changelog -p angular -r 2 -i CHANGELOG.md -s",
    "release": "/bin/bash scripts/release.sh",
    "test": "node test/prepare.js && jest --config test/jest.config.js"
  },
```

我们从命令行入口, `bin/vuepress.js` 开始

也许你可以先看看, [vuepress 已经或要去做的事情](https://vuepress.vuejs.org/zh/guide/#%E5%AE%83%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9F)

## bin/vuepress.js

### 命令行通用第一行

``` js
#!/usr/bin/env node
```

### 检查版本

``` js
const chalk = require('chalk') // 颜色库
const semver = require('semver')
const requiredVersion = require('../package.json').engines.node

if (!semver.satisfies(process.version, requiredVersion)) {
  console.log(chalk.red(
    `\n[vuepress] minimum Node version not met:` +
    `\nYou are using Node ${process.version}, but VuePress ` +
    `requires Node ${requiredVersion}.\nPlease upgrade your Node version.\n`
  ))
  process.exit(1)
}

```

[chalk] 作为`node的颜色库`是众所周知, 但是当你仅仅需要 不大不小的颜色输出时
[chalk] 就显得过大了 
[chalk 大小](https://bundlephobia.com/result?p=chalk@2.4.1)

也许我们可以使用[ansi-colors] 小点的库 [ansi-colors 大小](https://bundlephobia.com/result?p=ansi-colors@2.0.2)


[ansi-colors]: https://github.com/doowb/ansi-colors
[chalk]: https://github.com/chalk/chalk

### 命令行分流

下面可以说是 `vuejs 作者` 编写命令行的 通用形式了

> 用[vue-cli]和[本项目][local]做对比,除了命令选项不同, 关于 **其他命令勘误** 与 **命令错误输出**的处理大致相同

[local]: #%E5%85%B6%E4%BB%96%E5%91%BD%E4%BB%A4%E5%8B%98%E8%AF%AF-%E5%92%8C-%E5%91%BD%E4%BB%A4%E9%94%99%E8%AF%AF%E7%9A%84%E8%BE%93%E5%87%BA
[vue-cli]: https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli/bin/vue.js#L141

``` js
const path = require('path')
const { dev, build, eject } = require('../lib') // 3大命令

const program = require('commander')

program
  .version(require('../package.json').version)
  .usage('<command> [options]')

```

#### 1. vuepress dev

``` js
program
  .command('dev [targetDir]')
  .description('start development server')// 启动 开发服务器
  .option('-p, --port <port>', 'use specified port (default: 8080)')
  .option('-h, --host <host>', 'use specified host (default: 0.0.0.0)')
  .action((dir = '.', { host, port }) => {
    wrapCommand(dev)(path.resolve(dir), { host, port }) // 用 包裹函数 运行 命令选项
  })

```

- [vurepss `dev` Explain](#vurepss-cli)

#### 2. vuepress build 

``` js
program
  .command('build [targetDir]')
  .description('build dir as static site')
  .option('-d, --dest <outDir>', 'specify build output dir (default: .vuepress/dist)') // 构建版本 输出目录
  .option('--debug', 'build in development mode for debugging')
  .action((dir = '.', { debug, dest }) => {
    const outDir = dest ? path.resolve(dest) : null
    wrapCommand(build)(path.resolve(dir), { debug, outDir }) // 用 包裹函数 运行 命令选项
  })

```

- [vurepss `build` Explain](#vurepss-cli)

#### 3. vuepress eject

``` js
program
  .command('eject [targetDir]')
  .description('copy the default theme into .vuepress/theme for customization.') 
  //复制 默认主题到 .vuepress/theme , 提供用户自定义主题
  .action((dir = '.') => {
    wrapCommand(eject)(path.resolve(dir)) // 用 包裹函数 运行 命令选项
  })

```

- [vurepss `eject` Explain](#vurepss-cli)

#### 其他命令勘误 和 命令错误的输出

``` js
// output help information on unknown commands
program
  .arguments('<command>')
  .action((cmd) => {
    program.outputHelp()
    console.log(`  ` + chalk.red(`Unknown command ${chalk.yellow(cmd)}.`))
    console.log()
  })

// add some useful info on help
program.on('--help', () => {
  console.log()
  console.log(`  Run ${chalk.cyan(`vuepress <command> --help`)} for detailed usage of given command.`)
  console.log()
})

program.commands.forEach(c => c.on('--help', () => console.log()))

// enhance common error messages
const enhanceErrorMessages = (methodName, log) => {
  program.Command.prototype[methodName] = function (...args) {
    if (methodName === 'unknownOption' && this._allowUnknownOption) {
      return
    }
    this.outputHelp()
    console.log(`  ` + chalk.red(log(...args)))
    console.log()
    process.exit(1)
  }
}

enhanceErrorMessages('missingArgument', argName => {
  return `Missing required argument ${chalk.yellow(`<${argName}>`)}.`
})

enhanceErrorMessages('unknownOption', optionName => {
  return `Unknown option ${chalk.yellow(optionName)}.`
})

enhanceErrorMessages('optionMissingArgument', (option, flag) => {
  return `Missing required argument for option ${chalk.yellow(option.flags)}` + (
    flag ? `, got ${chalk.yellow(flag)}` : ``
  )
})

program.parse(process.argv)

if (!process.argv.slice(2).length) {
  program.outputHelp()
}

```

#### 包裹命令并且运行

``` js
function wrapCommand (fn) {
  return (...args) => {
    return fn(...args).catch(err => { // 真实运行 不过加了 catch 错误❌
      console.error(chalk.red(err.stack)) // 红色错误
      process.exitCode = 1
    })
  }
}
```

比如: `vuepress dev .`

``` js
wrapCommand(dev)(path.resolve(dir), { host, port })
// 第一括号 => 命令
// 第二括号 => 命令选项

// ===> 真实运行
return dev(".",{})
```

## vurepss cli

- [ ] [1. vurepss `dev` Explain](dev.ex.md)
- [ ] [2. vurepss `build` Explain](build.ex.md)
- [ ] [3. vurepss `eject` Explain](eject.ex.md)
