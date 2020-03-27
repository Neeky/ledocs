## 概要
之前的主力语言一直都是 SQL + Python 这两种语言都不太容易出成果，也就是说都不太可能做那种给一般消费者用的产品；js 也学了一段时间，但是又总感觉少了点什么。于是想换成 typescript 看一下。

![tsc](static/2020-13/tsc.png)

google-adsense

---

## typescript 编译器安装
typescript 在 js 的基础之上加上了强类型，它的实现方式就是代码先用 typescript 书写，然后再用 tsc 这个编译器把 typescript 代码编译成 js 代码。转了一圈，不过这样就可以在编译时做类型检查了。

typescript 的编译器是一个 node 软件包，所以可以直接通过 npm 来安装。
```bash
npm install -g typescript
```

---

## hello-word
还是先用 typescript 写一个经典的 hello-world 吧。

第一步：创建 main.ts 文件
```bash
touch main.ts
```
main.ts 文件的内容如下
```js
function hi(){
    console.log("hello world");
}
hi();
```
第二步：编译 main.ts 编译完成之后就会得到一个叫 main.js 的文件。
```bash
#编译
tsc main.ts 
#运行
node main.js
hello world
```

---