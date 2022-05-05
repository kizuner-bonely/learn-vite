## 指定根目录

`vite.config.ts`

```ts
import { defineConfig, normalizePath } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

// 全局 scss 文件
// 使用 normalizePath 解决 window 下的路径问题
const variablePath = normalizePath(
  path.resolve('./src/global/css/variable.scss'),
)

// https://vitejs.dev/config/
export default defineConfig({
  //* 指定入口文件在 src，这时 vite 会主动找 src 下的 index.html
  root: path.join(__dirname, 'src'),

  //* 插件
  plugins: [react()],
})
```

注意此时 src 下的 index.html 引入的路径也要改
比如说引入 src/main.ts，在 src/index.html 中引入的路径为 `/main.ts`

## CSS

**问题**

- 不支持选择器嵌套
- 样式污染
- 浏览器兼容
- 代码体积 ( 没用的 CSS 也会被打包 )

**解决方案**

- CSS 预处理器
  - sass/scss
  - less
  - stylus
- CSS Modules
- PostCSS
- CSS in JS
  - emotion
  - styled-components
- CSS 原子化框架
  - Tailwind CSS
  - Windi CSS

### CSS 预处理器

`vite.config.ts`

```ts
import { defineConfig, normalizePath } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

// 全局 scss 文件
// 使用 normalizePath 解决 window 下的路径问题
const variablePath = normalizePath(
  path.resolve('./src/global/css/variable.scss'),
)

// https://vitejs.dev/config/
export default defineConfig({
  //* 指定入口文件在 src，这时 vite 会主动找 src 下的 index.html
  // root: path.join(__dirname, 'src'),

  //* 插件
  plugins: [react()],

  //* css
  css: {
    // 预处理器
    preprocessorOptions: {
      scss: {
        // 该属性值会被每个 scss 文件的开头自动注入
        additionalData: `@import "${variablePath}";`,
      },
    },
  },
})
```

### CSS Modules

`vite.config.ts`

```ts
import { defineConfig, normalizePath } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

// 全局 scss 文件
// 使用 normalizePath 解决 window 下的路径问题
const variablePath = normalizePath(
  path.resolve('./src/global/css/variable.scss'),
)

// https://vitejs.dev/config/
export default defineConfig({
  //* 指定入口文件在 src，这时 vite 会主动找 src 下的 index.html
  // root: path.join(__dirname, 'src'),

  //* 插件
  plugins: [react()],

  //* css
  css: {
    // 预处理器
    preprocessorOptions: {
      scss: {
        // 该属性值会被每个 scss 文件的开头自动注入
        additionalData: `@import "${variablePath}";`,
      },
    },

    // CSS Modules
    modules: {
      // 该属性对生成的样式哈希值名称做定义
      // [name] 表示当前文件名  [local] 表示类名
      //   比如说 index.module.scss 下有个 header 属性
      //   那使用该 header 的类名就会显示为: index-module__header__IdNfn
      generateScopedName: '[name]__[local]__[hash:base64:5]',
    },
  },
})
```