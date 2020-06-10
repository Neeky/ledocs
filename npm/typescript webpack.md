## 背景
typescript 的文档都是直接写 ts 代码，写完之后再用 tsc 编译成 js 代码，就像下面这样。

![typescript-webpack](static/2020-22/sqlpy-0610-ts-webpack.jpg)

1、创建项目。
```bash
mkdir ts-projects
cd ts-projects
```

---

2、创建 ts 文件并写上几行伟大的代码。
```bash
touch index.js
```

index.ts 文件的内容如下。

```ts
console.log("hello typescript");
```

---

3、把 ts 代码编译成 js 并运行。
```bash
tsc index.ts
```
运行编译好的 js 代码。
```bash
node index.js

hello typescript
```

---


## 存在的问题
看到身边的前端同学打包时多用的是 `webpack` ，反正我也想深入的学习一些前端的知识，定不能在工具链上落后了，也应当试着用一下 webpack。

下面的步骤记录了怎么一步一步的配置 webpack 让它可以正常的打包 typescript 代码。

google-adsense

---

## 第一步 创建 webpack 项目
创建一个 webpack 项目，之后的所有操作都在这个项目中进行。
```bash
mkdir ts-project-a
cd ts-project-a

npm init -y
npm install webpack webpack-cli --save-dev
```
看到下面样式的输出说明 wepback 安装成功了。
```bash
+ webpack-cli@3.3.11
+ webpack@4.43.0
added 414 packages from 228 contributors in 35.328s
```

---

## 第二步 安装 ts 编译器和加载器
安装编译 typescript 要用到的编译器和加载器。

```bash
npm install --save-dev typescript ts-loader
```
有如下输出表明安装成功。
```bash
+ ts-loader@7.0.5
+ typescript@3.9.5
added 8 packages from 21 contributors in 14.891s
```
---

## 第三步 配置 webpack
配置 webpack 以 src/index.ts 文件作为入口，并把打包后的文件输出到 dist/bundle.js 。
```js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: [ '.tsx', '.ts', '.js' ],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

google-adsense

---

## 第四步 配置 tsconfig.json
配置 typescript 编译器让它支持 es6 语法，并把编译后的文件输出到 dist 目录。
```json
{
    "compilerOptions": {
        "outDir": "./dist/",
        "noImplicitAny": true,
        "module": "es6",
        "target": "es5",
        "allowJs": true
    }
}
```

---

## 第五步 添加入口文件

```bash
mkdir src
touch src/index.ts
```
src/index.ts 内容如下。
```ts
console.log("hello ts webpack");
```

---

## 第六步 使用 webpack 打包
```bash
npx webpack

Hash: 02eef0b7c7b960a44220
Version: webpack 4.43.0
Time: 1352ms
Built at: 2020-06-10 11:19:48
    Asset       Size  Chunks             Chunk Names
bundle.js  961 bytes       0  [emitted]  main
Entrypoint main = bundle.js
[0] ./src/index.ts 33 bytes {0} [built]

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/
```
---

## 第七步 执行打包后的文件
```bash
node dist/bundle.js 

hello ts webpack
```

---

