## 背景
最近在看 javascript 的官方文档，发现用 Proxy 进行元编程的话，程序可以非常的灵活；于是手贱的去 typescript 中试了下，一下就报错了。
```bash
index.ts:28:13 - error TS2304: Cannot find name 'Proxy'.

28 let x = new Proxy(p, person_handler);
               ~~~~~


Found 1 error.
```

![sqlpy](static/2020-32/sqlpy-proxy.jpg)

---

## 完整的ts代码
实现一个 Person 类，它只有 name 属性，如果要访问 name 之外的属性都统一返回 100 。之前要实现这样的逻辑并不优雅，有 Proxy 之后好多了。

```ts
class Person {
    name: string;

    constructor(name: string) {
        this.name = name;
    }

    say_hello() {
        console.log(`Hello my name is ${this.name} .`);
    }
}

// 当访问的属性不在对象里，就直接返回 100
let person_handler = {
    get: function (target: any, proname: string) {
        if (proname in target) {
            return target.proname;
        }

        return 100;
    }
}

let p = new Person('tom');
p.say_hello();

// 加上代理
let x = new Proxy(p, person_handler);

// 由于 p 中没有 age 所以会直接返回 100 .
console.log(x.age);

```

---

## 编译时报错
当我编译上面的代码时，tsc 报错了。
```bash
tsc index.ts
index.ts:28:13 - error TS2304: Cannot find name 'Proxy'.

28 let x = new Proxy(p, person_handler);
               ~~~~~


Found 1 error.


```
google-adsense

---

## 解决方案一
当我们直接调用 tsc xxx.ts 来编译代码，tsc 并不会读取 tsconfig.json 配置文件，我们要手动引入要导入的库。
```bash
tsc --lib dom,ES2015,ES2015.Proxy  index.ts

node index.js
Hello my name is tom .
100
```

---

## 解析方案二
每次编译都指定参数还是不方便，还是把参数写到配置文件里，让 tsc 运行时读取的比较好；在 lib 下加上 "dom" 和 "ES2015" 。
```json
{
    "compilerOptions": {
        "target": "es3", 
        "module": "commonjs", 
        "lib": [
            "dom",
            "ES2015",
        ],
        "strict": true, 
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    }
```
使用 --build 来编译。
```bash
tsc --build tsconfig.json
```

---



