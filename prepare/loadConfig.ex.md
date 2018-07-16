## loadConfig

``` js
// lib/prepare/resolveOptions.js
const siteConfig = loadConfig(vuepressDir)
```

加载 目录下`config`文件, 有三种文件格式

- **config.js**
- **config.yml**
- **config.toml**

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [lib/prepare/loadConfig.js](#libprepareloadconfigjs)
  - [require](#require)
  - [exports](#exports)
  - [解决 网站配置](#%E8%A7%A3%E5%86%B3-%E7%BD%91%E7%AB%99%E9%85%8D%E7%BD%AE)
  - [解析 配置文件[yml,toml]](#%E8%A7%A3%E6%9E%90-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6ymltoml)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## lib/prepare/loadConfig.js

### require
``` js
const fs = require('fs-extra')
const path = require('path')
const yamlParser = require('js-yaml') // yml 格式解析器
const tomlParser = require('toml') // toml 格式解析器

```

### exports
``` js
module.exports = function loadConfig (vuepressDir, bustCache = true) {
  const configPath = path.resolve(vuepressDir, 'config.js')
  const configYmlPath = path.resolve(vuepressDir, 'config.yml')
  const configTomlPath = path.resolve(vuepressDir, 'config.toml')

  if (bustCache) {
    delete require.cache[configPath] // 不缓存 网站配置 js文件
  }

```

### 解决 网站配置
``` js
  // resolve siteConfig
  let siteConfig = {}
  if (fs.existsSync(configYmlPath)) {
    siteConfig = parseConfig(configYmlPath) // 解析 yml 格式
  } else if (fs.existsSync(configTomlPath)) {
    siteConfig = parseConfig(configTomlPath) // 解析 toml 格式
  } else if (fs.existsSync(configPath)) {
    siteConfig = require(configPath)
  }

  return siteConfig
}

```

### 解析 配置文件[yml,toml]
``` js
function parseConfig (file) {
  const content = fs.readFileSync(file, 'utf-8')
  const [extension] = /.\w+$/.exec(file)
  let data

  switch (extension) {
  case '.yml':
  case '.yaml':
    data = yamlParser.safeLoad(content)
    break

  case '.toml':
    data = tomlParser.parse(content)
// 重新格式化以匹配配置，因为 TOML 不允许不同的数据类型
    // reformat to match config since TOML does not allow different data type
    // https://github.com/toml-lang/toml#array
    const format = []
    Object.keys(data.head).forEach(meta => {
      data.head[meta].forEach(values => {
        format.push([meta, values])
      })
    })
    data.head = format
    break
  }

  return data || {}
}

```