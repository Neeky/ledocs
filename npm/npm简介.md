## npm 简介
[npm](https://docs.npmjs.com/about-npm/) 是 nodejs 的一个包管理命令，可以用它来完成 nodejs 包的安装与发布。整个包管理系统包含三大块，1、npm 命令行 2、`npmjs.com` 这个网站 3、 registory 这个软件包数据库。 这三部分的关系如下图。

![npm-schema](static/2020-22/npm-schema.jpg)


为了可以使用 npm 的大部分功能，我们要在 npmjs.com 上有一个账号才行，我已经注册好了，这里直接登录。
```bash
npm login
Username: neeky
Password: 
Email: (this IS public) neeky@live.com
Logged in as neeky on http://registry.npmjs.org/.
```

google-adsense

---

## 从零开始发布一个 npm 模块
1、创建软件包目录。
```bash
mkdir nee-scope-package
cd nee-scope-package
```
2、把代码托管到 github。
```bash
echo "# nee-scope-package" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/Neeky/nee-scope-package.git
git push -u origin master
```
3、初始化 npm 包。
```bash
npm init -y --scope=@neeky
```
生成的配置文件 `package.json` 内容如下。
```ini
{
    "name": "@neeky/nee-scope-package",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "repository": {
        "type": "git",
        "url": "git+https://github.com/Neeky/nee-scope-package.git"
    },
    "author": "",
    "license": "ISC",
    "bugs": {
        "url": "https://github.com/Neeky/nee-scope-package/issues"
    },
    "homepage": "https://github.com/Neeky/nee-scope-package#readme"
```
name 指定包的名字，其中neeky是域名(与用户名保持一致，注意域名前面还要加上一个 @) `/`后面的是包名。

main 指定当包(模块)被其它程序加载时执行哪个文件，这里使用了默认的 index.js (我们这个时候还并没有创建这个文件，所以之后要补上)

version 表现包的版本号。

---

4、添加 `index.js` 文件，
```bash
touch inex.js

ll
total 24
-rw-r--r--  1 jianglexing  staff   20  6  7 12:54 README.md
-rw-r--r--  1 jianglexing  staff   36  6  7 12:57 index.js
-rw-r--r--  1 jianglexing  staff  514  6  7 13:06 package.json
```
为了让我们的模块在被导入时可以看到效果，要给 index.js 添加一行打印代码。
```js
console.log("this is in index.js");
```

---

5、发布。
```bash
npm publish --access public

npm notice 
npm notice 📦  @neeky/nee-scope-package@1.0.0
npm notice === Tarball Contents === 
npm notice 36B  index.js    
npm notice 514B package.json
npm notice 20B  README.md   
npm notice === Tarball Details === 
npm notice name:          @neeky/nee-scope-package                
npm notice version:       1.0.0                                   
npm notice package size:  421 B                                   
npm notice unpacked size: 570 B                                   
npm notice shasum:        6133dc011de54d67cfcd7ec56e729852d58c79a0
npm notice integrity:     sha512-e8Rpq+0Ox/Y89[...]1HKrkRmDLmCFA==
npm notice total files:   3                                       
npm notice 
+ @neeky/nee-scope-package@1.0.0
```
`--access public` 说明我们是发布一个公开的模块，要想发布私有的模块是要钱的。

google-adsense

---

## 安装刚才发布的模块
1、创建一个新的项目，然后在项目中安装刚才的模块。
```bash
# 创建创建目录
mkdir test-project
cd test-project/

# 初始化项目
npm init -y
Wrote to /private/tmp/test-project/package.json:

{
  "name": "test-project",
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

# 安装模块
npm i @neeky/nee-scope-package
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN test-project@1.0.0 No description
npm WARN test-project@1.0.0 No repository field.

+ @neeky/nee-scope-package@1.0.0
added 1 package and audited 1 package in 2.045s
found 0 vulnerabilities

```
下面目录下可以看到如下内容。
```bash
ll
total 16
drwxr-xr-x  3 jianglexing  wheel   96  6  7 22:00 node_modules
-rw-r--r--  1 jianglexing  wheel  406  6  7 22:00 package-lock.json
-rw-r--r--  1 jianglexing  wheel  292  6  7 22:00 package.json

tree node_modules/
node_modules/
└── @neeky
    └── nee-scope-package
        ├── README.md
        ├── index.js
        └── package.json

2 directories, 3 files
```
---

2、创建 index.js 文件，在文件中导入刚才发布的模块。

```bash
touch index.js
```
index.js 内容如下。
```js
require('@neeky/nee-scope-package');
```

---

3、运行 index.js 观察效果。

```bash
node index.js 
this is in index.js
```

---













## 包与模块的关系
总的来讲“包”包含“模块”。还有另一个角度用于观察包和模块的关系，npm 发布是的包，包里面可以只包含一个模块，当用户通过 npm 安装完包之后就可以使用包提供的模块了。也就是说包的重点在于发布和安装，而模块的重点在于能被`require()` 函数导入。

1、如果模块物理上表现为目录，那么目录中要有 `package.json` 文件并且 `package.json` 文件中要包含 `main` 字段。

2、如果模块物理上表现为目录，那么它还有另一种形式，就是目录下包含一个 index.js 文件。

3、模块还可以表现为一个单独的文件。

---

## 域
想想如果张三发布了一个叫 “A” 的包，李四也发布一个叫“A”的包，这不冲突了吗？npm 使用用户名作为域名，也就是说张三的“A”包叫 `@张三/A`，同理李四的“A" 叫 `@李四/A`，这个各个用户之间就不会冲突了。

---