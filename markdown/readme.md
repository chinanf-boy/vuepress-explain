## markdown

``` js
// lib/prepare/resolveOptioins.js
const createMarkdown = require('../markdown')
```



---

<!-- START doctoc -->
<!-- END doctoc -->

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
const emoji = require('markdown-it-emoji');
const anchor = require('markdown-it-anchor');
const toc = require('markdown-it-table-of-contents');
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

- [ ] [highlight](highlight.ex.md)

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

### 暴露slugify
``` js
  // expose slugify
  md.slugify = slugify;

  return md;
};

```

### dataReturnable
``` js
module.exports.dataReturnable = function dataReturnable(md) {
  // override render to allow custom plugins return data
  const render = md.render;
  md.render = (...args) => {
    md.__data = {};
    const html = render.call(md, ...args);
    return {
      html,
      data: md.__data
    };
  };
};
```
