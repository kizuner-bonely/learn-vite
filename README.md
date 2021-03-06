## 指定根目录

`vite.config.ts`

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

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

---

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

### PostCSS

首先需要添加开发依赖:

```shell
pnpm add -D autoprefixer
```

`vite.config.ts`

```ts
import { defineConfig, normalizePath } from 'vite'

// 引入
import autoprefixer from 'autoprefixer'

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

    // PostCSS
    postcss: {
      plugins: [
        autoprefixer({
          // 指定目标浏览器
          overrideBrowserslist: ['Chrome > 40', 'ff > 31', 'ie 11'],
        }),
      ],
    },
  },
})
```

**其他常见插件**

- postcss-pxtorem
  将 px 转换成 rem，常用于适配移动端
- postcss-preset-env
  可以编写最新的 css 语法
- cssnano
  用于压缩 CSS 代码，该插件相对于其他的更智能

---

## 代码检查和格式化

### JS/TS 检查

#### **ESLint**

准备工作

```shell
# 1.安装依赖
pnpm add -D eslint

# 2.ESLint 初始化
npx eslint --init

# 3.补充安装依赖 (在执行第二步之后根据选择会自动执行)
pnpm add -D eslint-plugin-react@latest @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest
```

`.eslintrc.js`

* parser
  ESLint -> Espree ( Acron ) <-- `@typescript-eslint/parser` -- TS

* parserOptions

  * ecmaVersion
    * ES6
    * ES2015
    * latest
  * sourceType
    * script ( default )
    * module ( ES Module )
  * ecmaFeatures
    * jsx

* rules

  * **!!! 如果使用了 prettier，最好只配置 prettierrc，否则对同一个配置不同的选项一直报错**

  * { [key in string]: [string | number, string] }

  * 配置数组第一项为规则 ID

    * off | 0 关闭规则
    * warn | 1 开启规则，只警告不报错 ( 不使程序退出 )
    * error | 2 开启规则，违反规则后抛出 error

  * 规则文档

    https://cn.eslint.org/docs/rules/

  * eg.

    ```js
    {
      'no-cond-assign': ['error', 'always'],
    	// 'no-cond-assign': 'error'
    }
    ```

* plugins

  ```js
  // pnpm add -D @typescript-eslint/eslint-plugin@latest
  
  // .eslintrc.js
  module.exports = {
    // 添加 TS 规则，可省略`eslint-plugin`
    plugins: ['@typescript-eslint']
  }
  ```

  使用插件后还要在规则中配置相应的 TS 规则

  ```js
  // .eslintrc.js
  module.exports = {
    // 开启一些 TS 规则
    rules: {
      '@typescript-eslint/ban-ts-comment': 'error',
      '@typescript-eslint/no-explicit-any': 'warn',
    }
  }
  ```

* extends
  继承另一份 ESLint 配置，这样就不用手动一一配置  plugins、rules 之类的

  * 从 ESLint 本身继承
  * 从类似 `eslint-config-xxx` 的 npm 包继承
  * 从 ESLint 插件继承

  ```js
  // .eslintrc.js
  module.exports = {
     "extends": [
       // 第1种情况 
       "eslint:recommended",
       // 第2种情况，一般配置的时候可以省略 `eslint-config`
       "standard"
       // 第3种情况，可以省略包名中的 `eslint-plugin`
       // 格式一般为: `plugin:${pluginName}/${configName}`
       "plugin:react/recommended"
       "plugin:@typescript-eslint/recommended",
     ]
  }
  ```

* env
  运行环境

  ```js
  // .eslint.js
  module.export = {
    "env": {
      "browser": "true", // 会启用 window 等全局变量
      "node": "true" // 会启用 global 等全局变量
    }
  }
  ```

  如果生成的 .eslint.js 使用了 `module.export`，这时不开启 `{ node: true }` 的话就会报警告

* globals
  声明全局变量

  * 'writable' | true  变量可重写
  * 'readonly' | false 变量不可重写
  * 'off' 禁用该全局变量

  ```js
  // .eslintrc.js
  module.exports = {
    "globals": {
      // 不可重写
      "$": false, 
      "jQuery": false 
    }
  }
  ```

如果有报关于 React 版本错误的，`.eslintrc.js` 添加以下配置

```js
{
  settings: {
    react: {
      version: 'detect',
    },
  },
}
```



在 vite 中接入接入 ESLint

```shell
pnpm add -D vite-plugin-eslint
```

`vite.config.ts`

```ts
import viteESLint from 'vite-plugin-eslint'

{
  plugins: [viteESLint()]
}
```

接着，只要启动项目就能实时检查 `pnpm run dev`



#### **prettier**

1. 安装主要依赖

   ```shell
   pnpm add -D prettier
   ```

2. 在项目根目录创建 `.prettierrc.js`

   ```js
   module.exports = {
     printWidth: 80, //一行的字符数，如果超过会进行换行，默认为80
     tabWidth: 2, // 一个 tab 代表几个空格数，默认为 2 个
     useTabs: false, //是否使用 tab 进行缩进，默认为false，表示用空格进行缩减
     singleQuote: true, // 字符串是否使用单引号，默认为 false，使用双引号
     semi: false, // 行尾是否使用分号，默认为true
     trailingComma: 'all', // 是否使用尾逗号
     bracketSpacing: true, // 对象大括号直接是否有空格，默认为 true，效果：{ a: 1 }
   }
   ```

3. 将 prettier 集成到现有的 ESLint 工具中

   1. 安装依赖

      ```shell
      # eslint-config-prettier  用来覆盖 ESLint 本身的规则配置
      # eslint-plugin-prettier  让 prettier 接管 `eslint --fix` ( 代码修复能力 )
      
      pnpm add -D eslint-config-prettier eslint-plugin-prettier
      ```

   2. 在 `eslintrc.js` 中接入 prettier 相关工具链
      https://prettier.io/docs/en/options.html

      ```js
      // .eslintrc.js
      module.exports = {
        env: {
          browser: true,
          es2021: true
        },
        extends: [
          "eslint:recommended",
          "plugin:react/recommended",
          "plugin:@typescript-eslint/recommended",
          // 1. 接入 prettier 的规则
          "prettier",
          "plugin:prettier/recommended"
        ],
        parser: "@typescript-eslint/parser",
        parserOptions: {
          ecmaFeatures: {
            jsx: true
          },
          ecmaVersion: "latest",
          sourceType: "module"
        },
        // 2. 加入 prettier 的 eslint 插件
        plugins: ["react", "@typescript-eslint", "prettier"],
        rules: {
          // 3. 注意要加上这一句，开启 prettier 自动修复的功能
          "prettier/prettier": "error",
          "react/react-in-jsx-scope": "off"，
          // 注意这里最好不要写规则，规则都统一放在 prettierrc
          // 否则可能会冲突
          // quotes: ["error", "single"],
          // semi: ["error", "always"],
        }
      }
      ```

4. 通过命令行检查并修复
   `package.json`

   ```json
   {
     "scripts": {
       // 省略已有 script
       "lint:script": "eslint --ext .js,.jsx,.ts,.tsx --fix --quiet ./",
     }
   }
   ```

   在终端执行命令

   ```shell
   pnpm run lint:script
   ```

5. 也可以在 IDE 设置"保存时格式化"



### CSS 检查

**Stylelint**

1. 安装依赖

   ```shell
   pnpm add -D stylelint stylelint-prettier stylelint-config-prettier stylelint-config-recess-order stylelint-config-standard stylelint-config-standard-scss
   ```

2. 在根目录创建 `.stylelintrc.js` 并作如下配置

   ```js
   module.exports = {
     // 注册 stylelint 的 prettier 插件
     plugins: ['stylelint-prettier'],
     // 继承一系列规则集合
     extends: [
       // standard 规则集合
       'stylelint-config-standard',
       // standard 规则集合的 scss 版本
       'stylelint-config-standard-scss',
       // 样式属性顺序规则
       'stylelint-config-recess-order',
       // 接入 Prettier 规则
       'stylelint-config-prettier',
       'stylelint-prettier/recommended'
     ],
     // 配置 rules
     rules: {
       // 开启 Prettier 自动格式化功能
       'prettier/prettier': true
     }
   };
   ```

3. 在 `package.json ` 中添加相关命令

   ```json
   {
     "scripts": {
       // 整合 lint 命令
       "lint": "npm run lint:script && npm run lint:style",
       // stylelint 命令
       "lint:style": "stylelint --fix \"src/**/*.{css.scss}\""
     }
   }
   ```

   执行 `pnpm run lint:style` 即可完成对样式代码的检查和格式化



对 `rules` 的配置

* null 表示关闭规则
* 通过简单值开启规则 ( 根据规则的不同而不同，可以是布尔值或字符串 )
* 数组 `[简单值, 自定义配置]`



在 vite 中使用

1. 安装依赖

   ```shell
   pnpm add -D @amatlash/vite-plugin-stylelint
   ```

2. 在 vite 中引入配置

   ```ts
   import viteStylelint from '@amatlash/vite-plugin-stylelint'
   
   {
     plugins: [
       // 省略其他插件
       viteStylelint({
         exclude: /windicss|node_modules/
       })
     ]
   }
   ```

   然后启动项目就能在终端看到项目所有问题  `pnpm run dev`



### 提交前检查

**Git 提交工作流: Husky + lint-staged**

1. 安装依赖

   ```shell
   pnpm add -D husky
   ```

2. 在 `package.json` 中配置 `husky` 启动脚本

   ```json
   {
     "scripts": {
       // 会在安装 npm 依赖后自动执行
       "postinstall": "husky install"
     }
   }
   ```

3. 通过终端添加 Husky 钩子

   ```shell
   npx husky add .husky/pre-commit "npm run lint"
   ```

   如果报错了，先重新安装一下依赖 `pnpm i`

   

**优化: 增量检查**

1. 安装依赖

   ```shell
   pnpm add -D lint-staged
   ```

2. 配置 `package.json`

   ```json
   {
     "lint-staged": {
       "**/*.{js,jsx,tsx,ts}": [
         "npm run lint:script",
         "git add ."
       ]
     },
     "**/*.{scss}": [
       "npm run lint:style",
       git add .
     ]
   }
   ```

3. 修改 `.husky/pre-commit` 脚本

   ```shell
   #!/bin/sh
   . "$(dirname "$0")/_/husky.sh"
   
   # npm run lint
   npx --no -- lint-staged
   ```



**规范 commit 信息**

1. 安装依赖

   ```shell
   pnpm i commitlint @commitlint/cli @commitlint/config-conventional -D
   ```

2. 在项目根目录创建 `.commitlintrc.js`，并作如下编辑

   ```js
   module.exports = {
     extends: ["@commitlint/config-conventional"]
   }
   ```

   ```
   提交信息
   type: 提交类型
   subject: 提交的摘要信息
   
   <type>: <subject>
   
   type:
   	feat: 添加新功能
   	fix: 修复 Bug
   	chore: 一些不影响功能的修改( 删除无用代码、注释等 )
   	docs: 文档的修改
   	perf: 性能优化
   	refactor: 代码重构
   	test: 添加测试代码
   ```

3. 将 `commitlint` 的功能集成到 `Husky` 钩子中

   ```shell
   npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
   ```

   

---

## 静态资源处理

路径别名

`vite.config.ts`

```ts
import path from 'path'

{
  resolve: {
    alias: {
      '@assets': path.resolve(__dirname, './src/assets')
    }
  }
}
```



### 普通图片引入

**组件内部**

```react
import { useEffect } from 'react'
import styles from './index.module.scss'
import logo from '@assets/imgs/vite.png'

export function Header() {
  useEffect(() => {
    const img = document.getElementById('logo') as HTMLImageElement
    img.src = logo
  }, [])

  return (
    <div className={styles.header}>
      <p>This is header</p>

      <div>
        <h2>引入图片</h2>
        {/* 引入图片方式1 */}
        <img src={logo} alt="" />
        {/* 引入图片方式2 */}
        <img id="logo" alt="" />
      </div>
    </div>
  )
}

```

**css 背景图**

```scss
.header {
  background: url('@assets/imgs/background.png') no-repeat;
}

```



### SVG

1. 安装相关插件

   ```shell
   pnpm add -D vite-plugin-svgr
   ```

   * React  `vite-plugin-svgr`
   * Vue3 `vite-svg-loader`
   * Vue2 `vite-plugin-vue2-svg`

2. 在 `vite.config.ts` 中使用插件

   ```ts
   import svgr from 'vite-plugin-svgr'
   
   {
     plugins: [svgr()]
   }
   ```

3. 在 `tsconfig.json` 添加以下配置防止类型报错

   ```json
   {
     "compilerOptions": {
       "types": ["vite-plugin-svgr/client"]
     }
   }
   ```

4. 在组件中使用 SVG 组件

   ```react
   import { ReactComponent as Logo } from '@assets/icons/logo.svg'
   
   export function Header() {
     return <Logo />
   }
   ```

   

### JSON

**直接引入就能使用**

```react
import { version } from '../../../package.json'
```

**在数据量较大时可在 `vite.config.ts` 中作以下配置**

```ts
{
  json: {
    stringify: true
  }
}
```

这样 JSON 就失去了按名导入的能力，就不能像上述引入 JSON 方式使用。



### Web Worker

1. 新建一个 worker 文件

   ```js
   const start = () => {
     let count = 0
     setInterval(() => {
       // 给主线程传值
       postMessage(++count)
     }, 2000)
   }
   
   start()
   ```

2. 在组件中引入，要注意在引入路径后加上 `?worker` 以告诉 vite 这是个 web worker 脚本文件

   ```react
   import Worker from './example.js?worker'
   
   // 1.初始化 Worker 实例
   const worker = new Worker()
   // 2.主线程监听 worker 信息
   worker.addEventListener('message', e => {
     console.log(e)
   })
   ```



### wasm

1. 定义一个 wasm，其对应的 JS 源码如下

   ```js
   export function fib(n) {
     var a = 0,
       b = 1;
     if (n > 0) {
       while (--n) {
         let t = a + b;
         a = b;
         b = t;
       }
       return b;
     }
     return a;
   }
   ```

2. 直接在组件中导入使用

   ```react
   import init from './fib.wasm'
   
   type FibFunc = (num: number) => number
   
   init({}).then((exports) => {
     const fibFunc = exports.fib as FibFunc
     console.log('Fib result:', fibFunc(10))
   })
   ```

   

### 其他静态资源

* **媒体类**
  * mp4
  * webm
  * ogg
  * mp3
  * wav
  * flac
  * aac
* **字体类**
  * woff
  * woff2
  * eot
  * ttf
  * otf
* **文本类**
  * webmanifest
  * pdf
  * txt

直接在 `vite.config.ts` 中配置

```ts
{
  assetsInclude: ['.gltf']
}
```



### 特殊资源后缀

* `?url`
  只获取文件路径而不是内容
* `?raw`
  只获取资源的字符串内容
* `?inline`
  资源强制内联而不是打包成单独文件



### 生产环境处理

#### 自定义部署域名

1. 在根目录新建文件 `.env.development`

   ```
   NODE.ENV=development
   ```

2. 在根目录新建文件 `.env.production`

   ```
   NODE_ENV=production
   ```

3. 编辑 `vite.config.ts`

   ```ts
   // 判断是否为生产环境
   const isProduction = process.env.NODE_ENV === 'production'
   // 填入项目的 CDN 域名地址
   const CDN_URL = 'xxx'
   
   {
     base: isProduction ? CDN_URL : '/'
   }
   ```

这时执行 `pnpm run build` 可以发现静态资源已经加上 CDN 前缀

**如果需要另外的存储地址**

1. 在项目根目录新增文件 `.env`

   ```
   // 开发环境优先级: .env.development > .env
   // 生产环境优先级: .env.production > .env
   VITE_IMG_BASE_URL=https://my-image-cdn.com
   ```

2. 在 `src/vite.env.d.ts` 增加类型声明

   ```ts
   /// <reference types="vite/client" />
   
   interface ImportMetaEnv {
     readonly VITE_APP_TITLE: string
     // 自定义环境变量
     readonly VITE_IMG_BASE_URL: string
   }
   
   interface ImportMeta {
     readonly env:ImportMetaEnv
   }
   ```

3. 在组件中使用

   ```react
   <img src={new URL('./logo.png', import.meta.env.VITE_IMG_BASE_URL).href} />
   ```

   

