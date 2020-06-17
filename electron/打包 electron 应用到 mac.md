## 背景
用 electron 写了一个应用程序，想分发给别人用，总不可能告诉别人，你要在电脑上安装 electron 开发环境，不然运行不了这个程序。

为了解决这个问题，我们可以把 electron 程序打包成 mac 的 dmg 安装包。

![sqlpy](static/2020-25/sqlpy-electron-mac.jpg)


---

## 第一步 创建项目
给我们的 electron 项目起这个名字，我在这里就叫 emp (electron-mac-package) 吧。
```bash
mkdir emp
cd emp

npm init -y

Wrote to /Users/neekyjiang/gits/electrons/emp/package.json:

{
  "name": "emp",
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

---

## 第二步 安装 electron
安装 electron electron-packager electron-installer-dmg 。
```bash
npm install electron electron-packager electron-installer-dmg

+ electron-packager@14.2.1
+ electron@9.0.4
+ electron-installer-dmg@3.0.0
added 208 packages from 176 contributors in 51.092s

10 packages are looking for funding
  run `npm fund` for details
```

---

## 第三步 修改 package.json 
把入口文件设置为 main.js ，启动方式设置为 electron . 。
```json
{
  "name": "emp",
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
  "dependencies": {
    "electron": "^9.0.4",
    "electron-installer-dmg": "^3.0.0",
    "electron-packager": "^14.2.1"
  }
}

```

---

## 第四步 添加 main.js 
来一份最为简单的 main.js ，其内容如下。
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

---

## 第五步 添加 index.html 文件
main.js 是直接加载的 index.html 文件来渲染的，所以我们还是加上一个简单的 index.html 文件。
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

## 第六步 打包 electron 应用
使用第二步安装的 electron-packager  来打包应用。
```bash
npx electron-packager .
Packaging app for platform darwin x64 using electron v9.0.4
WARNING: Found 'electron' but not as a devDependency, pruning anyway
Wrote new app to /Users/neekyjiang/gits/electrons/emp/emp-darwin-x64
```

---

## 第七步 生成 dmg 安装文件
```bash
npx electron-installer-dmg ./emp-darwin-x64/emp.app/ emp

ll | grep dmg
-rw-r--r--@   1 neekyjiang  staff  79114105  6 17 10:27 emp.dmg
```

---

## 安装效果
1、双击 emp.dmg 进入安装页面。

![sqlpy](static/2020-25/sqlpy-installer.jpg)

2、运行效果如下。

![sqlpy](static/2020-25/sqlpy-electron-hello.jpg)

---