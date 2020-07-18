## 背景
今天在看 typescript 的官方文档 [Iterators and Generators](https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html) 这个部分的时候，突然想到 javascript 中不是有三种 for 循环吗？官方并没有讲最最古老
的那种，可能是有意的不讲了吧。

![sqlpy](static/2020-29/sqlpy-js-for.jpg)

---

## C 语言式的下标迭代风格
我记得 javascript 中最古老的就是这种风格的 for 循环了。
```typescript
let nums = [100, 200, 300];

console.log("C 语言风格的下标迭代.");
for (let i = 0; i < nums.length; i++) {
    console.log(i);
}
```
运行效果如下。
```bash
C 语言风格的下标迭代.
0
1
2
```

---

## js 风格的下标迭代
这种风格更加通常一些，用于迭代下标和对象的属性。
```typescript
// 迭代下标
let nums = [100, 200, 300];

console.log("JS 风格的下标迭代.");
for (let i in nums) {
    console.log(i);
}

// 迭代属性
class Person {
    name: string = '';
    age: number = 0;

    constructor(name: string = '', age: number = 0) {
        this.name = name;
        this.age = age;
    }
}

console.log("JS 风格的对象属性迭代.")
let tom: Person = new Person('tom', 10);
for (let p in tom) {
    console.log(p);
}

```
运行效果如下
```bash
JS 风格的下标迭代.
0
1
2

JS 风格的对象属性迭代.
name
age
```
---

## JS 风格的值迭代
这种风格下返回的再是下标(或属性)，而是直接是下标(或属性)对应的值。
```typescript
let nums = [100, 200, 300];

console.log("JS 风格的值迭代.");
for (let i of nums) {
    console.log(i);
}

```
运行效果。
```bash
JS 风格的值迭代.
100
200
300
```

---

总的来讲第二种比较常用，也比较通用。

---

