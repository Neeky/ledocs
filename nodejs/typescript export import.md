## ts 模块
ts 在模块方面的语法就三个 1、export 导出 2、import 导入 3、default export 默认导出。

![sqlpy](static/2020-25/sqlpy-export-import.jpg)

---

## export 导出
按官方文档的说法任何方式声明的对象(变量、函数、类、接口)都能被 export 导出，语法上来看导出就是在声明前面加上 `export` 就行。
```ts
// src/tools.ts
// 开始执行
console.log("begin tools.ts");

// 导出 Version
export let Version:string = "0.0.1"

// 导出 funPrint
export function funPrint(message:string){
    console.log(message);
}


// 执行完成
console.log("end tools.ts");
```

---

## import 导入
入口文件  src/index.ts 的内容如下。
```ts
// src/index.ts

import {Version,funPrint} from './tools';

function main(){
    funPrint(`this is in main function version = ${Version} .`);
}

main();
```

---

## 执行效果
```bash
npx webpack

Hash: e9ff450583aa1eb77c5a
Version: webpack 4.43.0
Time: 970ms
Built at: 2020/06/18 下午4:19:28
  Asset      Size  Chunks             Chunk Names
main.js  1.05 KiB       0  [emitted]  main
Entrypoint main = main.js
[0] ./src/index.ts + 1 modules 329 bytes {0} [built]
    | ./src/index.ts 143 bytes [built]
    | ./src/tools.ts 186 bytes [built]



node dist/index.js 

begin tools.ts
end tools.ts
this is in main function version = 0.0.1 .
```
可以看到输出打印了 "begin tools.ts" 和 "end tools.ts" ,这个可以看出被导入模块会被从头到尾执行一遍。

---

## export default 
在上面的代码中导入要用 `{ }` 解包，ts 中有一种情况可以不用写这一对括号，那就是当对象是以 `export default` 方式导出时。
```ts
// src/tools.ts

// 开始执行
console.log("begin tools.ts");

export let Version:string = "0.0.1"

export default function funPrint(message:string){
    console.log(message);
}


// 执行完成
console.log("end tools.ts");
```
可以看到 funPrint 是以 export default 的方式导出的，那么当导入 funPrint 的时候可以写简单一点。
```ts
// 非 export default 方式导出的只能是解包
import {Version} from './tools';

// export default 方式导出的可以不解包
import funPrint from './tools';

function main(){
    funPrint(`this is in main function version = ${Version} .`);
}

main();
```

运行效果。
```bash
npx webpack 

Hash: 7f32debeca98e83197a0
Version: webpack 4.43.0
Time: 808ms
Built at: 2020/06/18 下午4:36:38
  Asset      Size  Chunks             Chunk Names
main.js  1.05 KiB       0  [emitted]  main
Entrypoint main = main.js
[0] ./src/index.ts + 1 modules 403 bytes {0} [built]
    | ./src/index.ts 209 bytes [built]
    | ./src/tools.ts 194 bytes [built]


node dist/main.js

begin tools.ts
end tools.ts
this is in main function version = 0.0.1 .
```

---

## 总结
1、在一个模块中 export 和 export default 可以同时存在。

2、多次从一个模块中导入对象，被导入模块只会执行一次。

---
