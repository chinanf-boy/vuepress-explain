## codegen

``` js
// lib/prepare/index.js
const { genRoutesFile, genComponentRegistrationFile } = require('./codegen')
```

生成 路由和用户组件 注册代码

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## lib/prepare/codegen.js

### require
``` js
const path = require('path')
const { fileToComponentName, resolveComponents } = require('./util')

```

### genRoutesFile
``` js
exports.genRoutesFile = async function ({
  siteData: { pages },
  sourceDir,
  pageFiles
}) {
  function genRoute ({ path: pagePath, key: componentName }, index) {
    const file = pageFiles[index]
    const filePath = path.resolve(sourceDir, file)
    let code = `
  {
    name: ${JSON.stringify(componentName)},
    path: ${JSON.stringify(pagePath)},
    component: ThemeLayout,
    beforeEnter: (to, from, next) => {
      import(${JSON.stringify(filePath)}).then(comp => {
        Vue.component(${JSON.stringify(componentName)}, comp.default)
        next()
      })
    }
  }`

    const dncodedPath = decodeURIComponent(pagePath)
    if (dncodedPath !== pagePath) {
      code += `,
  {
    path: ${JSON.stringify(dncodedPath)},
    redirect: ${JSON.stringify(pagePath)}
  }`
    }

    if (/\/$/.test(pagePath)) {
      code += `,
  {
    path: ${JSON.stringify(pagePath + 'index.html')},
    redirect: ${JSON.stringify(pagePath)}
  }`
    }

    return code
  }

  const notFoundRoute = `,
  {
    path: '*',
    component: ThemeNotFound
  }`

  return (
    `import ThemeLayout from '@themeLayout'\n` +
    `import ThemeNotFound from '@themeNotFound'\n` +
    `import { injectMixins } from '@app/util'\n` +
    `import rootMixins from '@app/root-mixins'\n\n` +
    `injectMixins(ThemeLayout, rootMixins)\n` +
    `injectMixins(ThemeNotFound, rootMixins)\n\n` +
    `export const routes = [${pages.map(genRoute).join(',')}${notFoundRoute}\n]`
  )
}

```

### genComponentRegistrationFile
``` js
exports.genComponentRegistrationFile = async function ({ sourceDir }) {
  function genImport (file) {
    const name = fileToComponentName(file)
    const baseDir = path.resolve(sourceDir, '.vuepress/components')
    const absolutePath = path.resolve(baseDir, file)
    const code = `Vue.component(${JSON.stringify(name)}, () => import(${JSON.stringify(absolutePath)}))`
    return code
  }

  const components = (await resolveComponents(sourceDir)) || []
  return `import Vue from 'vue'\n` + components.map(genImport).join('\n')
}


```