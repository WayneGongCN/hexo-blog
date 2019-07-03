---
title: React 开发浏览器扩展 webpack 配置
date: 2018-06-03
tags:
    - react
    - webpack
categories: notes
---

## 用 webpack 为浏览器扩展项目打包
浏览器扩展项目与普通 react 项目有些许差异，做个记录。

### 目录结构
```
├─ .babelrc
├─ .editorconfig
├─ .gitignore
├─ webpack.config.js
├─ package.json
├─ dist
└─ src
    ├─ manifest.json
    ├─ icons
    ├─ background
    │    └─ index.js
    ├─ content
    │    └─ index.js
    └─ popup
            ├─ index.html
            ├─ index.jsx
            └─ style.css
```

### 项目依赖
```
npm install webpack\
 html-webpack-plugin\
 copy-webpack-plugin\
 clean-webpack-plugin\
 style-loader\
 css-loader\
 babel-core\
 babel-preset-react\
 babel-preset-env\
 babel-loader --save-dev

npm install react\
 react-dom --save
```

### webpack 配置
`webpack.config.js`
```javascript
const path = require('path')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')

module.exports = {
    entry: {
        backgournd: './src/background/index.js',
        popup: './src/popup/index.jsx'
    },

    watch: true,

    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name].bundle.js'
    },

    resolve: {
        extensions: ['.js', '.jsx']
    },

    devtool: 'inline-source-map',

    plugins: [
        new CleanWebpackPlugin(['dist']),
        new HtmlWebpackPlugin({
            inject: true,
            chunks: ['popup'],
            filename: 'popup.html',
            template: './src/popup/index.html'
        }),
        new CopyWebpackPlugin([
            { from: './src/manifest.json' },
            { from: './src/icons/', to: './icons' }
        ])
    ],

    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.jsx$/,
                exclude: /node_modules/,
                use: ['babel-loader'],
            }
        ]
    }
}
```

### babel 配置
`.babelrc`
```json
{
    "presets": [
        "env",
        "react"
    ]
}
```