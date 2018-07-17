## markdown

``` js
// lib/prepare/resolveOptioins.js
const createMarkdown = require('../markdown')
```

1. 创建 markdown 编译器, 

2. 且暴露 编译给用户配置

3. 返回 markdown 编译器

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [lib/markdown/index.js](#libmarkdownindexjs)
  - [require](#require)
  - [exports](#exports)
  - [允许用户配置 slugify](#%E5%85%81%E8%AE%B8%E7%94%A8%E6%88%B7%E9%85%8D%E7%BD%AE-slugify)
  - [自定义插件](#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8F%92%E4%BB%B6)
  - [第三方插件](#%E7%AC%AC%E4%B8%89%E6%96%B9%E6%8F%92%E4%BB%B6)
  - [应用用户配置](#%E5%BA%94%E7%94%A8%E7%94%A8%E6%88%B7%E9%85%8D%E7%BD%AE)
  - [暴露slugify](#%E6%9A%B4%E9%9C%B2slugify)
  - [dataReturnable](#datareturnable)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## lib/markdown/index.js

### require

```js
const highlight = require('./highlight');
const highlightLines = require('./highlightLines');
const preWrapper = require('./preWrapper');
const lineNumbers = require('./lineNumbers');
const component = require('./component');
const hoistScriptStyle = require('./hoist');
const convertRouterLink = require('./link');
const containers = require('./containers');
const snippet = require('./snippet');
const emoji = require('markdown-it-emoji'); // emoji 解析
const anchor = require('markdown-it-anchor'); // 
const toc = require('markdown-it-table-of-contents'); // 解析 目录
const _slugify = require('./slugify');
const {parseHeaders} = require('../util/parseHeaders');
```

### exports
``` js
module.exports = ({markdown = {}} = {}) => {
```

### 允许用户配置 slugify
``` js
  // allow user config slugify
  const slugify = markdown.slugify || _slugify;

  const md = require('markdown-it')({ 
// markdown-it 作为 markdown文件的解析 有着许多强壮的功能
    html: true, // 比如: 转化为 html 格式
    highlight // 语法高亮
  })
```

> 1. 创建 markdown 编译器

- [x] [highlight](highlight.ex.md)
- [x] [_slugify](slugify.ex.md)

### 自定义插件
``` js
    // custom plugins
    .use(component) 
    .use(highlightLines)
    .use(preWrapper)
    .use(snippet)
    .use(
      convertRouterLink,
      Object.assign(
        {
          target: '_blank',
          rel: 'noopener noreferrer'
        },
        markdown.externalLinks
      )
    )
    .use(hoistScriptStyle)
    .use(containers)

```

- [ ] [component](component.ex.md)
- [ ] [highlightLines](highlightLines.ex.md)
- [ ] [preWrapper](preWrapper.ex.md)
- [ ] [snippet](snippet.ex.md)
- [ ] [convertRouterLink](link.ex.md)
- [ ] [hoistScriptStyle](hoist.ex.md)
- [ ] [containers](containers.ex.md)


### 第三方插件
``` js
    // 3rd party plugins
    .use(emoji)
    .use(
      anchor,
      Object.assign(
        {
          slugify,
          permalink: true,
          permalinkBefore: true,
          permalinkSymbol: '#'
        },
        markdown.anchor
      )
    )
    .use(
      toc,
      Object.assign(
        {
          slugify,
          includeLevel: [2, 3],
          format: parseHeaders
        },
        markdown.toc
      )
    );

```

### 应用用户配置
``` js
  // apply user config
  if (markdown.config) {
    markdown.config(md);
  }

  if (markdown.lineNumbers) {
    md.use(lineNumbers);
  }

  module.exports.dataReturnable(md);

```

> 2. 暴露 编译器 给用户配置



- [ ] [lineNumbers](lineNumbers.ex.md)

### 暴露slugify
``` js
  // expose slugify
  md.slugify = slugify;

  return md;
};

```

> 3. 返回 markdown 编译器


### dataReturnable
``` js
module.exports.dataReturnable = function dataReturnable(md) {
  // override render to allow custom plugins return data
  const render = md.render;
  md.render = (...args) => {
    md.__data = {};
    const html = render.call(md, ...args); // md = 转 > html 的结果
    return {
      html,
      data: md.__data // 带数据返回
    };
  };
};
```

补充 md 编译器的渲染函数, 可以带 数据返回