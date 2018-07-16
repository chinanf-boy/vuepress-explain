## resolveOptions.ex.md

``` js
// lib/prepare.js
await resolveOptions(sourceDir)
```

加载 选项配置

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [lib/prepare/resolveOptions.js](#libprepareresolveoptionsjs)
  - [require](#require)
  - [resolveOptions 是 `async/await`函数](#resolveoptions-%E6%98%AF-asyncawait%E5%87%BD%E6%95%B0)
  - [规范化基本标题网址](#%E8%A7%84%E8%8C%83%E5%8C%96%E5%9F%BA%E6%9C%AC%E6%A0%87%E9%A2%98%E7%BD%91%E5%9D%80)
  - [解决outDir](#%E8%A7%A3%E5%86%B3outdir)
  - [解决主题](#%E8%A7%A3%E5%86%B3%E4%B8%BB%E9%A2%98)
  - [使用默认主题](#%E4%BD%BF%E7%94%A8%E9%BB%98%E8%AE%A4%E4%B8%BB%E9%A2%98)
  - [解决主题布局](#%E8%A7%A3%E5%86%B3%E4%B8%BB%E9%A2%98%E5%B8%83%E5%B1%80)
  - [使用外部主题](#%E4%BD%BF%E7%94%A8%E5%A4%96%E9%83%A8%E4%B8%BB%E9%A2%98)
  - [使用自定义主题](#%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%BB%E9%A2%98)
  - [解决主题 NotFound](#%E8%A7%A3%E5%86%B3%E4%B8%BB%E9%A2%98-notfound)
  - [解决主题 增强应用](#%E8%A7%A3%E5%86%B3%E4%B8%BB%E9%A2%98-%E5%A2%9E%E5%BC%BA%E5%BA%94%E7%94%A8)
  - [解析主题配置](#%E8%A7%A3%E6%9E%90%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE)
  - [解决 algolia](#%E8%A7%A3%E5%86%B3-algolia)
  - [解决 markdown](#%E8%A7%A3%E5%86%B3-markdown)
  - [解析 pageFiles](#%E8%A7%A3%E6%9E%90-pagefiles)
  - [解决 lastUpdated](#%E8%A7%A3%E5%86%B3-lastupdated)
  - [解析 pagesData](#%E8%A7%A3%E6%9E%90-pagesdata)
  - [提取 yaml frontmatter](#%E6%8F%90%E5%8F%96-yaml-frontmatter)
  - [推断标题](#%E6%8E%A8%E6%96%AD%E6%A0%87%E9%A2%98)
  - [解决网站数据](#%E8%A7%A3%E5%86%B3%E7%BD%91%E7%AB%99%E6%95%B0%E6%8D%AE)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## lib/prepare/resolveOptions.js

### require
``` js
const fs = require('fs-extra')
const path = require('path')
const globby = require('globby') // 查找 匹配 文件/目录
const createMarkdown = require('../markdown')
const loadConfig = require('./loadConfig')
const { encodePath, fileToPath, sort, getGitLastUpdatedTimeStamp } = require('./util')
const {
  inferTitle,
  extractHeaders,
  parseFrontmatter
} = require('../util/index')

```

### resolveOptions 是 `async/await`函数
``` js
module.exports = async function resolveOptions (sourceDir) {
  const vuepressDir = path.resolve(sourceDir, '.vuepress')
  const siteConfig = loadConfig(vuepressDir) // 加载 目录下的配置
```

- [x] [`loadConfig` Ex](loadConfig.ex.md)

### 规范化基本标题网址
``` js
  // normalize head tag urls for base
  const base = siteConfig.base || '/'
  if (base !== '/' && siteConfig.head) {
    siteConfig.head.forEach(tag => {
      const attrs = tag[1]
      if (attrs) {
        for (const name in attrs) {
          if (name === 'src' || name === 'href') { //条件 src / href 链接
            const value = attrs[name]
            if (value.charAt(0) === '/') { // 第一位字符是不是'/'
              attrs[name] = base + value.slice(1) // 拼接 此文档网站的根目录与head配置的链接
            }
          }
        }
      }
    })
  }
```

- 下面例子🌰结果是: ['link', { rel: 'icon', href: **`test/logo.png`** }],
``` js
// config.js
module.exports = {
  base: "test"
  head: [
    ['link', { rel: 'icon', href: `/logo.png` }], //<== 解析
    // ...
  ],
}
```


### 解决 outDir
``` js
  // resolve outDir
  const outDir = siteConfig.dest // 输出目录确认, 不然
    ? path.resolve(siteConfig.dest)
    : path.resolve(sourceDir, '.vuepress/dist') // 默认是 ./.vuepress/dist
```

### 解决主题:默认/其他
``` js
  // resolve theme
  const useDefaultTheme = (
    !siteConfig.theme && // 外部主题
    !fs.existsSync(path.resolve(vuepressDir, 'theme')) // 本地定义的主题
  )
  const defaultThemePath = path.resolve(__dirname, '../default-theme')
  let themePath = null
  let themeLayoutPath = null
  let themeNotFoundPath = null
  let themeEnhanceAppPath = null

```
  
### 1.使用默认主题
``` js
  if (useDefaultTheme) {
    // use default theme
    themePath = defaultThemePath
    themeLayoutPath = path.resolve(defaultThemePath, 'Layout.vue')
    themeNotFoundPath = path.resolve(defaultThemePath, 'NotFound.vue')
```
  
### 2.使用其他主题布局
``` js
  } else {
    // resolve theme Layout
```
    
### 2.1 使用外部主题
``` js
    if (siteConfig.theme) {
      // use external theme
      try { 
        themeLayoutPath = require.resolve(`vuepress-theme-${siteConfig.theme}/Layout.vue`, {
          paths: [
            path.resolve(__dirname, '../../node_modules'),
            path.resolve(sourceDir)
          ]
        })
        themePath = path.dirname(themeLayoutPath)
      } catch (e) {
        throw new Error(`[vuepress] Failed to load custom theme "${
          siteConfig.theme
        }". File vuepress-theme-${siteConfig.theme}/Layout.vue does not exist.`)
      }
    } 
```

- 外部主题的 *npm包名称* 是:`vuepress-theme-` + **主题名**

> 比如 vuepress的主题是: `vuepress-theme-vue` > 可以 局部或全局 安装. 
    
### 2.2 使用自定义主题
``` js
    else {
      // use custom theme
      themePath = path.resolve(vuepressDir, 'theme')
      themeLayoutPath = path.resolve(themePath, 'Layout.vue')
      if (!fs.existsSync(themeLayoutPath)) {
        throw new Error(`[vuepress] Cannot resolve Layout.vue file in .vuepress/theme.`)
      }
    }
```

### 解决主题 NotFound
``` js
    // resolve theme NotFound
    themeNotFoundPath = path.resolve(themePath, 'NotFound.vue')
    if (!fs.existsSync(themeNotFoundPath)) {
      themeNotFoundPath = path.resolve(defaultThemePath, 'NotFound.vue') // 没有就默认
    }
```

- **404页面**, 发挥想象力罗

### 解决主题 增强应用
``` js
    // resolve theme enhanceApp
    themeEnhanceAppPath = path.resolve(themePath, 'enhanceApp.js')
    if (!fs.existsSync(themeEnhanceAppPath)) {
      themeEnhanceAppPath = null
    }
  }
```

- `enhanceApp.js` 可以用来 体现 vue组件的 使用和效果呈现

> 比如: [**vue-foldable**可折叠组件](https://github.com/ulivz/vue-foldable) ,通过`enhanceApp.js`文件中[导入自己的vue组件: <**csdn-foladable**>](https://github.com/ulivz/vue-foldable/blob/master/docs/.vuepress/enhanceApp.js#L22),然后在 md文档中[应用该组件](https://sourcegraph.com/github.com/ulivz/vue-foldable@master/-/blob/docs/README.md?view=code#L173), 最终是[这样的网页呈现](https://vue-foldable.ulivz.com/#vue-foldable-csdn)

### 解析主题配置
``` js
  // resolve theme config
  const themeConfig = siteConfig.themeConfig || {}
```

### 解决 algolia 搜寻
``` js
  // resolve algolia
  const isAlgoliaSearch = (
    themeConfig.algolia ||
    Object.keys(siteConfig.locales && themeConfig.locales || {})
          .some(base => themeConfig.locales[base].algolia)
  )
```

### 解决 markdown
``` js
  // resolve markdown
  const markdown = createMarkdown(siteConfig)
```

- [ ] [`createMarkdown` Ex](../markdown/readme.md)

### 找到 pageFiles 路径
``` js
  // resolve pageFiles
  const pageFiles = sort(await globby(['**/*.md', '!.vuepress', '!node_modules'], { cwd: sourceDir }))
```

### 要不要 lastUpdated
``` js
  // resolve lastUpdated
  const shouldResolveLastUpdated = (
    themeConfig.lastUpdated ||
    Object.keys(siteConfig.locales && themeConfig.locales || {})
          .some(base => themeConfig.locales[base].lastUpdated)
  )
```

### 解析 pagesData
``` js
  // resolve pagesData
  const pagesData = await Promise.all(pageFiles.map(async (file) => {
    const filepath = path.resolve(sourceDir, file)
    const key = 'v-' + Math.random().toString(16).slice(2)
    const data = {
      key,
      path: encodePath(fileToPath(file))
    }

    if (shouldResolveLastUpdated) {
      data.lastUpdated = getGitLastUpdatedTimeStamp(filepath)
    }
```

### 提取 yaml frontmatter
``` js
    // extract yaml frontmatter
    const content = await fs.readFile(filepath, 'utf-8')
```

### 推断标题
``` js
    const frontmatter = parseFrontmatter(content)
    // infer title
    const title = inferTitle(frontmatter)
    if (title) {
      data.title = title
    }
    const headers = extractHeaders(
      frontmatter.content,
      ['h2', 'h3'],
      markdown
    )
    if (headers.length) {
      data.headers = headers
    }
    if (Object.keys(frontmatter.data).length) {
      data.frontmatter = frontmatter.data
    }
    if (frontmatter.excerpt) {
      const { html } = markdown.render(frontmatter.excerpt)
      data.excerpt = html
    }
    return data
  }))
```

### 解决网站数据
``` js
  // resolve site data
  const siteData = {
    title: siteConfig.title || '',
    description: siteConfig.description || '',
    base,
    pages: pagesData,
    themeConfig,
    locales: siteConfig.locales
  }

  const options = {
    siteConfig,
    siteData,
    sourceDir,
    outDir,
    publicPath: base,
    pageFiles,
    pagesData,
    themePath,
    themeLayoutPath,
    themeNotFoundPath,
    themeEnhanceAppPath,
    useDefaultTheme,
    isAlgoliaSearch,
    markdown
  }

  return options
}

```