## interface 
typescript 中的 interface 用于定义约束，就功能上来说 interface 的描述能力非常强，几乎所有的对象都能描述。

![interface](static/2020-27/sqlpy-interface.jpg)

---

## 描述一般属性
定义一个叫 Person 的接口，这个接口要求属于它的对象都要有一个 name 属性。
```ts
// 接口描述对象
interface Person {
    // 约定对象需要包含一个 name 属性
    name: string;
}

// 声明 Person 类的变量 p 
let p: Person;
// 赋值
p = {
    'name': 'Tom',
};

function sayHello(p: Person) {
    console.log(`My name is ${p.name} .`);
}

sayHello(p);
```
运行效果。
```bash
tsc index.ts && node index.js
My name is Tom .
```

google-adsense

---

interface 是强约束的，对于没有出现在接口中的内容，如果出现在了对象里面会报错。
```ts
// 接口描述对象
interface Person {
    // 约定对象需要包含一个 name 属性
    name: string;
}

// 声明 Person 类的变量 p 
let p: Person;
// 赋值
p = {
    'name': 'Tom',
    'age':16 // 这个会引发报错，原因是 age 并没有出现在 接口中
};

```

---

## 描述可选属性
对于那些可有可无的属性可以用 `name?:type` 这样的形式来描述。
```ts
// 接口描述对象
interface Person {
    // 约定对象需要包含一个 name 属性
    name: string;
    age?: number; // 现在 age 将是一个可选的属性
}

// 声明 Person 类的变量 p 
let p: Person;
// 赋值
p = {
    'name': 'Tom',
    'age': 16 // 这个会引发报错，原因是 age 并没有出现在 接口中
};

let q: Person;
q = {
    'name': 'Jerr'
    //  因为 age 是可选属性，所以缺少 age 也不会报错
}

```


---

## 描述只读属性
对于只读属性可以使用 `readonly name:type` 的方式来描述。
```ts
// 接口描述对象
interface Person {
    // 约定对象需要包含一个 name 只读属性
    readonly name: string;
    age?: number; // 现在 age 将是一个可选的属性
}

// 声明 Person 类的变量 p 
let p: Person;
// 赋值
p = {
    'name': 'Tom',
    'age': 16 // 这个会引发报错，原因是 age 并没有出现在 接口中
};

let q: Person;
q = {
    'name': 'Jerr'
    //  因为 age 是可选属性，所以缺少 age 也不会报错
}

```
google-adsense

---

## 描述函数
描述函数的话要使用函数调用符号`()`，下面以描述一个加法函数为例。
```ts
interface PlusInterface {
    (x: number, y: number): number;
}

let add: PlusInterface;
add = function (x, y) {
    return x + y
}

console.log(add(1, 1));
```
运行效果。
```bash
tsc index.ts && node index.js
2
```

---


## 描述可索引对象
要描述可索引对象要使用索引符号`[]`，形式上和描述函数差不多，用 interface 描述的索引对象通常是数组。
```ts
interface Names {
    [index: number]: string;
}

let students: Names;
students = ['Tom', 'Jerr', 'Bob'];

console.log(students);
```

---

## 描述类
使用 interface 描述类。
```ts
interface PersonInterface {
    name: string;

    sayHello(): void;
}

class Person implements PersonInterface {
    name: string = '';
    constructor(name: string) {
        this.name = name;
    }

    sayHello() {
        console.log(this.name);
    }
}

let p: Person;
p = new Person('tom')

p.sayHello();
```

运行效果。

```bash
tsc index.ts  && node index.js
tom
```


---
