## highlight

``` js
// lib/markdown/index.js
  const md = require('markdown-it')({ 
// markdown-it 作为 markdown文件的解析 有着许多强壮的功能
    html: true, // 比如: 转化为 html 格式
    highlight // 语法高亮
  })
```

语法高亮

---

<!-- START doctoc -->
<!-- END doctoc -->

## lib/markdown/highlight.js

### require
``` js
const chalk = require('chalk')
const prism = require('prismjs')
const loadLanguages = require('prismjs/components/index')
const escapeHtml = require('escape-html') // 用于HTML的转义字符串
const logger = require('../util/logger')

```

- [prismjs](https://github.com/PrismJS/prism) 轻巧，强大，优雅的语法高亮

### 加载嵌入式语法高亮
``` js
// required to make embedded highlighting work...
loadLanguages(['markup', 'css', 'javascript'])

```

### 默认代码包裹函数
``` js
function wrap (code, lang) {
  if (lang === 'text') {
    code = escapeHtml(code) // 转义字符串 安全·
  }
  return `<pre v-pre class="language-${lang}"><code>${code}</code></pre>`
}

```

### exports
``` js
module.exports = (str, lang) => {
  if (!lang) {
    return wrap(str, 'text')
  }
  lang = lang.toLowerCase()
  const rawLang = lang
  if (lang === 'vue' || lang === 'html') {
    lang = 'markup'
  }
  if (lang === 'md') {
    lang = 'markdown'
  }
  if (lang === 'ts') {
    lang = 'typescript'
  } // 转 语言名称 符合 prism 规范
  if (!prism.languages[lang]) { // 不支持
    try {
      loadLanguages([lang])
    } catch (e) {
      logger.warn(chalk.yellow(`[vuepress] Syntax highlight for language "${lang}" is not supported.`))
    }
  }
  if (prism.languages[lang]) { // 有支持
    const code = prism.highlight(str, prism.languages[lang], lang)
    return wrap(code, rawLang) // 包装 原语言 
  }
  return wrap(str, 'text')
}

```

- [ ] [logger](../util/logger.ex.md)