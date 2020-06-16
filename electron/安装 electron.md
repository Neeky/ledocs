## 背景
自从毕业之后我的主力语言就从 C# 切换到了 Python + SQL ，前者写 GUI 程序并不容易，后者根本就不能写。这不最近在学习 Typescript 以后写几个方便自己使用的桌面应用程序也行呀！于是我选择了 electron ，没有想到第一步安装就遇到了问题，现打算把这个问题记录下。

![sqlpy](static/2020-25/sqlpy-electron.jpg)

---

## 遇到的问题
在安装 electron 时卡在了 `node install.js`。
```bash
npm install electron --save-dev

> core-js@3.6.5 postinstall /private/tmp/electron-projects/node_modules/core-js
> node -e "try{require('./postinstall')}catch(e){}"

Thank you for using core-js ( https://github.com/zloirock/core-js ) for polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open Collective or Patreon: 
> https://opencollective.com/core-js 
> https://www.patreon.com/zloirock 

Also, the author of core-js ( https://github.com/zloirock ) is looking for a good job -)


> electron@9.0.4 postinstall /private/tmp/electron-projects/node_modules/electron
> node install.js

RequestError: connect ETIMEDOUT 52.216.226.64:443
    at ClientRequest.request.once.error (/private/tmp/electron-projects/node_modules/got/source/request-as-event-emitter.js:178:14)
    at Object.onceWrapper (events.js:277:13)
    at ClientRequest.emit (events.js:194:15)
    at ClientRequest.origin.emit.args (/private/tmp/electron-projects/node_modules/@szmarczak/http-timer/source/index.js:37:11)
    at TLSSocket.socketErrorListener (_http_client.js:392:9)
    at TLSSocket.emit (events.js:189:13)
    at emitErrorNT (internal/streams/destroy.js:82:8)
    at emitErrorAndCloseNT (internal/streams/destroy.js:50:3)
    at process._tickCallback (internal/process/next_tick.js:63:19)
npm WARN electron-projects@1.0.0 No description
npm WARN electron-projects@1.0.0 No repository field.

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! electron@9.0.4 postinstall: `node install.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the electron@9.0.4 postinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/jianglexing/.npm/_logs/2020-06-16T07_56_47_757Z-debug.log

```
google-adsense

---

## 解决方案
google 了一下发面大家都在说用 cnpm 就没有问题，作为一个还没有入门的人，我也跟着大家一起走吧，cnpm 走起！
```bash
npm install -g cnpm 
/usr/local/bin/cnpm -> /usr/local/lib/node_modules/cnpm/bin/cnpm
+ cnpm@6.1.1
added 685 packages from 957 contributors in 19.8s
```
这样就解决了，之后要是安装 electron 就用 cnpm 。

---

## 安装配置第一个 electron 项目
1、第一步 初始化项目。
```bash
cd /tmp
mkdir mac-ele
cd mac-ele

npm init -y
Wrote to /private/tmp/mac-ele/package.json:

{
  "name": "mac-ele",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

google-adsense

---

2、第二步 安装 electron。
```bash
cnpm install electron --save-dev
✔ Installed 1 packages
✔ Linked 82 latest versions
[1/2] scripts.postinstall electron@9.0.4 › @electron/get@1.12.2 › global-agent@2.1.12 › core-js@^3.6.5 run "node -e \"try{require('./postinstall')}catch(e){}\"", root: "/private/tmp/mac-ele/node_modules/_core-js@3.6.5@core-js"
Thank you for using core-js ( https://github.com/zloirock/core-js ) for polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open Collective or Patreon: 
> https://opencollective.com/core-js 
> https://www.patreon.com/zloirock 

Also, the author of core-js ( https://github.com/zloirock ) is looking for a good job -)

[1/2] scripts.postinstall electron@9.0.4 › @electron/get@1.12.2 › global-agent@2.1.12 › core-js@^3.6.5 finished in 65ms
[2/2] scripts.postinstall electron@* run "node install.js", root: "/private/tmp/mac-ele/node_modules/_electron@9.0.4@electron"
Downloading electron-v9.0.4-darwin-x64.zip: [==========================================================] 100% ETA: 0.0 seconds
[2/2] scripts.postinstall electron@* finished in 2m
✔ Run 2 scripts
Recently updated (since 2020-06-09): 2 packages (detail see file /private/tmp/mac-ele/node_modules/.recently_updates.txt)
✔ All packages installed (87 packages installed from npm registry, used 2m(network 3s), speed 550.61kB/s, json 83(225.61kB), tarball 1.26MB)
```
---

3、第三步 配置项目的入口点为 "main.js",启动程序为 electron, `package.json` 的内容如下。
```json
{
    "name": "mac-ele",
    "version": "1.0.0",
    "description": "",
    "main": "main.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "electron ."
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "electron": "^9.0.4"
    }
}
```

---

4、第四步 添加入口点文件"main.js"，其内容如下。
```js
const { app, BrowserWindow } = require('electron')

function createWindow() {
    // Create the browser window.
    let win = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            nodeIntegration: true
        }
    })

    // and load the index.html of the app.
    win.loadFile('index.html')
}

app.whenReady().then(createWindow)
```
当程序启动时会执行 main.js ,而 main.js 又会去加载 index.html 文件来渲染页面。

---

5、第五步 添加页面文件 index.html 文件，其内容如下。
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    <!-- https://electronjs.org/docs/tutorial/security#csp-meta-tag -->
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
</head>

<body>
    <h1>Hello World!</h1>
    We are using node
    <script>document.write(process.versions.node)</script>,
    Chrome
    <script>document.write(process.versions.chrome)</script>,
    and Electron
    <script>document.write(process.versions.electron)</script>.
</body>

</html>
```
---

6、启动。
```bash
npm start
```
![sqlpy](static/2020-25/sqlpy-electron-exsample.jpg)

第一个 electron 窗口程序就这样完了。

---

