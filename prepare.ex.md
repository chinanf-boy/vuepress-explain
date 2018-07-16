## prepare

``` js
// build.js or dev.js or etc.
await prepare(dir)
```

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [prepare/index.js](#prepareindexjs)
  - [require](#require)
  - [prepare是 `async/await`函数](#prepare%E6%98%AF-asyncawait%E5%87%BD%E6%95%B0)
  - [1.加载选项](#1%E5%8A%A0%E8%BD%BD%E9%80%89%E9%A1%B9)
  - [2.生成 路由和用户组件 注册代码](#2%E7%94%9F%E6%88%90-%E8%B7%AF%E7%94%B1%E5%92%8C%E7%94%A8%E6%88%B7%E7%BB%84%E4%BB%B6-%E6%B3%A8%E5%86%8C%E4%BB%A3%E7%A0%81)
  - [3.生成 siteData](#3%E7%94%9F%E6%88%90-sitedata)
  - [4.处理用户覆盖](#4%E5%A4%84%E7%90%86%E7%94%A8%E6%88%B7%E8%A6%86%E7%9B%96)
  - [5.处理 enhanceApp.js](#5%E5%A4%84%E7%90%86-enhanceappjs)
  - [6.处理主题 enhanceApp.js](#6%E5%A4%84%E7%90%86%E4%B8%BB%E9%A2%98-enhanceappjs)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## prepare/index.js

### require
``` js
const path = require('path')
const fs = require('fs-extra')
const resolveOptions = require('./resolveOptions')
const { genRoutesFile, genComponentRegistrationFile } = require('./codegen')
const { writeTemp, writeEnhanceTemp } = require('./util')
const logger = require('../util/logger')
const chalk = require('chalk')

```

### prepare是 `async/await`函数
``` js
module.exports = async function prepare (sourceDir) {
```

### 1.加载选项
``` js
  // 1. load options
  const options = await resolveOptions(sourceDir)

```

- [ ] [`resolveOptions` Explain](./prepare/resolveOptions.ex.md)

### 2.生成 路由和用户组件 注册代码
``` js
  // 2. generate routes & user components registration code
  const routesCode = await genRoutesFile(options)
  const componentCode = await genComponentRegistrationFile(options)

  await writeTemp('routes.js', [
    componentCode,
    routesCode
  ].join('\n'))

```

- [ ] [`genRoutesFile` Explain](./prepare/codegen.ex.md#genroutesfile)
- [ ] [`genComponentRegistrationFile` Explain](./prepare/codegen.ex.md#gencomponentregistrationfile)
- [ ] [`writeTemp` Explain](./prepare/util.md#writetemp)

### 3.生成 siteData
``` js
  // 3. generate siteData
  const dataCode = `export const siteData = ${JSON.stringify(options.siteData, null, 2)}`
  await writeTemp('siteData.js', dataCode)

```

### 4.处理用户覆盖
``` js
  // 4. handle user override
  const overridePath = path.resolve(sourceDir, '.vuepress/override.styl')
  const hasUserOverride = fs.existsSync(overridePath)
  await writeTemp('override.styl', hasUserOverride ? `@import(${JSON.stringify(overridePath)})` : ``)

  const stylePath = path.resolve(sourceDir, '.vuepress/style.styl')
  const hasUserStyle = fs.existsSync(stylePath)
  await writeTemp('style.styl', hasUserStyle ? `@import(${JSON.stringify(stylePath)})` : ``)

  // 临时提示,将在下一个版本中删除。
  if (hasUserOverride && !hasUserStyle) {
    logger.tip(
      `${chalk.magenta('override.styl')} has been split into 2 APIs, we recommend you upgrade to continue.\n` +
      `      See: ${chalk.magenta('https://vuepress.vuejs.org/default-theme-config/#simple-css-override')}`
    )
  }

```

### 5.处理 enhanceApp.js
``` js
  // 5. handle enhanceApp.js
  const enhanceAppPath = path.resolve(sourceDir, '.vuepress/enhanceApp.js')
  await writeEnhanceTemp('enhanceApp.js', enhanceAppPath)

```

### 6.处理主题 enhanceApp.js
``` js
  // 6. handle the theme enhanceApp.js
  await writeEnhanceTemp('themeEnhanceApp.js', options.themeEnhanceAppPath)

  return options
}

```