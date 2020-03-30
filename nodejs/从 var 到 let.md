## 概要
var 是老版本 js 中声明变量的方式，除非对它理解的很深入，不然非常容易出问题。

![var-vs-let](static/2020-14/var-vs-let.png)

google-adsense

---

## var 容易遇到的问题
在 let 声明出现之前 js 只有全局作用域和函数作用域，通常情况下不会有什么问题，比如下面的代码。
```js
function outer() {
    var o = 100;

    function inner() {

        return o * 2; // 可以正常访问到外部作用域中的 o 变量。
    }

    var i = inner();
    console.log(i);  //  这里可以打印出 200  
}


outer();
```
编译运行。
```bash
tsc main.ts && node main.js
200
```
---

非正常情况一、可以访问到其它块中声明的变量。
```ts
function outer(isChecking: boolean = true) {
    if (isChecking) {
        // 
        var o = 100;
    }

    // 在 if 语句块中定义的变量 o，在块外面也访问到了
    console.log(`isChcking = ${isChecking}     o = ${o}`);

    function inner() {
        return o * 2;
    }

    var i = inner();
    console.log(i);
}


outer(true);
```
编译运行。
```bash
tsc main.ts && node main.js
isChcking = true     o = 100
200
```
由于 var 声明没有块级作用域，只有函数作用域，所以在 if 块里面声明的 o 变量，在 if 块外面也可以访问到(它们在同一个函数作用域当中)。

---

非正常情况二：
```ts
function main() {
    for (var i = 0; i < 10; i++) {
        console.log(i);
    }
}
main();
```
编译运行。
```bash
tsc main.ts && node main.js
0
1
2
3
4
5
6
7
8
9
```
目前看没有问题，不过只要代码改动一点点就可以看到问题了。
```ts
function main() {
    console.log("start for");
    for (var i = 0; i < 10; i++) {
        // 延时执行
        setTimeout(function () {
            console.log(i);
        }, 100 * i);
    }
    console.log("end for");
}
main();
```
看到了吧，每一次打印出来的都是 10 。
```bash
tsc main.ts && node main.js
start for
end for
10
10
10
10
10
10
10
10
10
10
```

为什么会打印 10 ？1、函数 main 的作用域中只有一个 `i`  2、等到 setTimout 的回调要执行的时候 for 循环已经执行完成，这个时候 i = 10 ，而回调函数每一次都读的同一个 `i`;

---

## let 引入块级作用域
let 声明会把包含它的块变成它的作用域。
```ts
function main() {
    if (true) {
        let isChecking = true;
    }
    console.log(isChecking); // 作用域之外不再能访问到 isChecking 了
}
main();
```
编译会报错。
```bash
tsc main.ts && node main.js
main.ts:5:17 - error TS2304: Cannot find name 'isChecking'.

5     console.log(isChecking);
                  ~~~~~~~~~~


Found 1 error.
```

---

## let 对 for 的影响
let 对 for 的影响也是全方位的，首先在块作用域上。
```ts
function main() {
    for (var i = 0; i < 3; i++) {
        //console.log(i);
    }
    // for 的外面也可以访问到 i
    console.log(i); // 打印3
}
main();
```
如果改成 let 那么 for 的外面就不再能访问到 i 了。
```ts
function main() {
    for (let i = 0; i < 3; i++) {
        //console.log(i);
    }
    console.log(i);
}
main();
```
编译运行。
```bash
tsc main.ts && node main.js
main.ts:5:17 - error TS2304: Cannot find name 'i'.

5     console.log(i);
                  ~


Found 1 error
```
---

另外 let 会为 for 的每一次执行都生成一个唯一的值 i 值。
```ts
function main() {
    console.log("start for");
    for (let i = 0; i < 10; i++) {
        setTimeout(function () {
            console.log(i);
        }, 100 * i);
    }
    console.log("end for");
}
main();
```
编译运行。
```bash
tsc main.ts && node main.js
start for
end for
0
1
2
3
4
5
6
7
8
9
```

---

## let 不再提升
var 时使用可以在声明之前。
```js
function main() {
    console.log(i);
    var i = 100;
    console.log(i);
}
main();
```
由于声明会提升和赋值不会，所以会先打印出 undefined 后打印出 100 。
```js
tsc main.ts && node main.js
undefined
100
```
let 要求一定要先声明后使用。
```ts
function main() {
    console.log(i);
    let i = 100;
    console.log(i);
}
main();
```
编译报错。
```bash
tsc main.ts && node main.js
main.ts:2:17 - error TS2448: Block-scoped variable 'i' used before its declaration.

2     console.log(i);
                  ~

  main.ts:3:9
    3     let i = 100;
              ~
    'i' is declared here.


Found 1 error.
```

---

## let 不再能重复声明
var 声明是支持重复声明的。
```ts
function main() {
    var x = 100;
    var x = 200;
}
main();
```
let 如果重复声明会报错。
```ts
function main() {
    let x = 100;
    let x = 200;
}
main();
```
编译报错。
```bash
tsc main.ts && node main.js
main.ts:2:9 - error TS2451: Cannot redeclare block-scoped variable 'x'.

2     let x = 100;
          ~

main.ts:3:9 - error TS2451: Cannot redeclare block-scoped variable 'x'.

3     let x = 200;
          ~


Found 2 errors.
```
注意这样的写法也算是重复声明。
```ts
function main(x) {
    let x = 100;
}
```
---


