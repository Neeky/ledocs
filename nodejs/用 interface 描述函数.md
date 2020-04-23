## typescript 支持用接口来描述函数
用接口描述一个打印函数来举例子。
```ts
interface PrintStr {
    // k 部分是函数的形参数 v 部分是函数的返回值
    (msg: string): void;
}

// 通过匿名函数的形式来定义函数
let funPrint = function (msg) {
    console.log(msg);
}

funPrint("hello world.");
```
运行效果。
```bash
tsc main.ts && node main.js
hello world.
```

---

## 注意事项
不要随便使用关键字来做变量名会出错的。
```ts
interface PrintStr {
    // k 部分是函数的形参数 v 部分是函数的返回值
    (msg: string): void;
}

// 这里使用 print 
let print = function (msg) {
    console.log(msg);
}

print("hello world.");
```
编译时报错。
```bash
tsc main.ts && node main.js
main.ts:7:5 - error TS2451: Cannot redeclare block-scoped variable 'print'.

7 let print = function (msg) {
      ~~~~~

  ../../../../../usr/local/lib/node_modules/typescript/lib/lib.dom.d.ts:19705:18
    19705 declare function print(): void;
                           ~~~~~
    'print' was also declared here.

main.ts:11:7 - error TS2554: Expected 0 arguments, but got 1.

11 print("hello world.");
         ~~~~~~~~~~~~~~

../../../../../usr/local/lib/node_modules/typescript/lib/lib.dom.d.ts:19705:18 - error TS2451: Cannot redeclare block-scoped variable 'print'.

19705 declare function print(): void;
                       ~~~~~

  main.ts:7:5
    7 let print = function (msg) {
          ~~~~~
    'print' was also declared here.


Found 3 errors.
```





---