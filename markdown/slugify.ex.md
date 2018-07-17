## slugify

``` js
// lib/markdown/index.js
    .use(
      anchor,
      Object.assign(
        {
          slugify,
// or
    .use(toc,
      Object.assign(
        {
          slugify,          
```

字符串 slugify 删除 非ascii字符，所以我们
在这里使用自定义实现

---

<!-- START doctoc -->
<!-- END doctoc -->

## lib/markdown/slugify.js

### require

``` js
// string.js slugify drops non ascii chars so we have to
// use a custom implementation here
const removeDiacritics = require('diacritics').remove

```

### 自定义字符匹配
``` js
// eslint-disable-next-line no-control-regex
const rControl = /[\u0000-\u001f]/g
const rSpecial = /[\s~`!@#$%^&*()\-_+=[\]{}|\\;:"'<>,.?/]+/g

```

### exports
``` js
module.exports = function slugify (str) {
  return removeDiacritics(str)
    // 删除控制字符
    .replace(rControl, '')
    // 替换特殊字符
    .replace(rSpecial, '-')
    // 取下连续分离器
    .replace(/\-{2,}/g, '-')
    // 删除前缀和尾随分隔符
    .replace(/^\-+|\-+$/g, '')
    // 确保它不以数字开头（＃121）
    .replace(/^(\d)/, '_$1')
    // 小写
    .toLowerCase()
}

```