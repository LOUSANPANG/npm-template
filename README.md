---
title: 构建一个npm包管理工具
date: 2020-12-31
tags: 
    - npm
categories: cli
keywords: [npm]
description: 如何整合一个共享的代码库
top_img: # 除非特定需要，可以不写
comments: # 是否显示评论 除非设置false,可以不写
cover: https://s1.ax1x.com/2020/10/21/BCVpFg.png # 缩略图
toc: # 章节目录 除非特定文章设置，可以不写
toc_number: # 是否显示toc数字 除非特定文章设置，可以不写
copyright: # 是否显示版权 除非特定文章设置，可以不写
---

[![npm webpack](https://s1.ax1x.com/2020/10/23/BE0IOg.jpg)](https://imgchr.com/i/BE0IOg)

## 一、介绍
技术：
`npm webpack ts`

项目结构：
```json
|- .github // ISSUES模板
|- .vscode // vscode配置
|- build // webpack
    |- webpack.base.conf.js
    |- webpack.dev.conf.js
    |- webpack.build.conf.js
|- docs // 文档
|- example // 示例
|- packages // 源码目录
    |- src
    |- template
    |- static
    |- types
    |- utils
|- .babelrc
|- .browerslistrc
|- .editorconfig
|- .eslintrc
|- .gitgnore
|- .prettierrc
|- LICENSE
|- LICENSE
|- tsconfig.json
``` 


## 二、模板构建

### 2.1 项目初始化
[构建`package.json`](https://docs.npmjs.com/cli/v6/configuring-npm/package-json)
```bash
$ npm init
```


### 2.1 创建项目文件
- .browerslistrc 浏览器版本
- .gitignore 代码提交忽略文件
- LICENSE 版权
- .github 关于GitHub ISSUES的模板等
- .vscode vscode配置
- build 打包配置文件夹
- docs 文档描述文件夹
- example 示例文件夹
- packages 源码文件夹
    - src 源码
    - static 静态文件
    - template 模板文件
    - types ts类型文件
    - utils 工具文件


### 2.2 配置webpack
```bash
$ yarn add -D webpack webpack-cli
```

配置webpack文件merge
```bash
$ yarn add -D webpack-merge
```

配置webpack清除dist插件
```bash
$ yarn add -D clean-webpack-plugin
```

配置webpack添加html模板插件
```bash
$ yarn add -D html-webpack-plugin
```

应用
```js
// /build/webpack.base.conf.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const path = require('path')

module.exports = {
  entry : path.join(__dirname, '../packages/src/index.ts'),

  output : {
    filename : 'bundle.js',
    path : path.join(__dirname, '../dist'),
    libraryTarget: 'umd'
  },

  resolve: {
    extensions: [ '.tsx', '.ts', '.js' ]
  },

  module: {
    rules: [
      {
        test: /\.(ts|js)?$/,
        use: 'babel-loader',
        exclude: /node_modules/,
      },
    ],
  },

  plugins: [
    new CleanWebpackPlugin()
  ]
}


// /build/webpack.dev.conf.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { merge } = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')

const devWebpackConfig = {
  mode: 'development',
  devtool: 'source-map',

  plugins: [
    new HtmlWebpackPlugin({
      filename: path.join(__dirname, '../example/test.html'),
      template: path.join(__dirname, '../packages/template/index.htm')
    }),
  ]
}

module.exports = merge(baseWebpackConfig, devWebpackConfig)


// /build/webpack.prod.conf.js
const { merge } = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')

const prodWebpackConfig = {
  mode: 'production',
  devtool: false,

  plugins: []
}

module.exports = merge(baseWebpackConfig, prodWebpackConfig)


// package.json
{
  "main": "dist/bundle.js",
  "scripts": {
    "dev": "webpack --config build/webpack.dev.conf.js",
    "build": "webpack --config build/webpack.prod.conf.js",
  },
}
```


### 2.3 配置ts的eslint、prettier
```bash
$ yarn add -D eslint eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser 

$ yarn add -D prettier
```

应用
```json
// .editorconfig
# http://editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false


// .eslintrc
{
  "env": {
    "browser": true,
    "es6": true,
    "commonjs": true,
    "node": true
  },
  "parser": "@typescript-eslint/parser",
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "prettier"
  ],
  "plugins": [
    "prettier",
    "@typescript-eslint"
  ],
  "parserOptions": {
    "sourceType": "module"
  },
  "globals": {},
  "rules": {}
}

// .prettierrc
{
  "endOfLine": "auto" ,
  "printWidth": 100,
  "tabWidth": 2,
  "singleQuote": true,
  "trailingComma": "none",
  "semi": false
}

// package.json
  "scripts": {
    "lint": "eslint packages --fix --ext .ts,.tsx"
  },
```


### 2.4 配置 git-commit
```bash
$ yarn add -D commitizen cz-conventional-changelog
```

应用
```json
  "scripts": {
    "commit": "git-cz",
    "release": "standard-version"
  },
  "config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
```


### 2.5 配置babel+ts
```bash
$ yarn add -D @babel/core @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread @babel/preset-env @babel/preset-typescript babel-loader

$ yarn add -D @babel/plugin-transform-runtime
$ yarn add -S @babel/runtime @babel/runtime-corejs3
```

-注意：
  - 需要注意转化 `Promise Map Set` 等语法（@babel/plugin-transform-runtime）
  - 需要注意转化 `ESNext` 新语法（@babel/preset-env）
  - 注意使用 `Class static` 语法（@babel/plugin-proposal-class-properties）


使用
```js
// packgage.json
"scripts": {
  "check": "tsc -w"
}


// tsconfig.json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "allowJs": true,
    "sourceMap": true,
    "noImplicitAny": true,
    "removeComments": true,
    "noImplicitThis": true,
    "strictNullChecks": true,
    "preserveConstEnums": true,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "outDir": "./dist/",
    "typeRoots": [
      "node_modules/@types",
      "global.d.ts"
    ]
  },
  "exclude": [
    "node_modules"
  ],
  "compileOnSave": false
}


// .babelrc
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-typescript"
  ],
  "plugins": [
    "@babel/plugin-proposal-class-properties",
    "@babel/plugin-proposal-object-rest-spread",
    [
      "@babel/plugin-transform-runtime",
      { "corejs": 3 }
    ]
  ]
}


// /build/webpack.base.conf.js
module: {
  rules: [
    {
      test: /\.(ts|js)?$/,
      use: 'babel-loader',
      exclude: /node_modules/,
    },
  ],
},
```


## 三、测试

### 3.1 内部测试
```bash
$ npm run dev

// example/tets.html
测试
```

### 3.2 在项目中
插件中
```bash
$ npm run build
$ npm link
```

在项目中
```bash
// xx 指的是 package.json内的 name 字段 （也就是发布包的的名称）
$ npm link xx
```


## 四、发布包

```bash
$ npm login

$ npm publish
```


<br>
<br>
<br>
<br>