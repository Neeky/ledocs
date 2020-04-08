## Optional Properties
可选属性，用于声明可能存在于对象当中，也有可能不存在于对象当中的属性。

![optional-readonly](static/2020-15/optional-readonly.png)

google-adsense

---

## 声明语法
语法就是在声明的背后加上一个`?`号用于表现可选。
```ts
interface People {
    'name': string,
    'age': number,
    'job'?: string, // 不是每一个人都有工作的所以 job 属性是可选的
}
```

---

## 可选属性的例子
举一个可选属性用于编译时检查的例子。
```ts
interface People {
    'name': string,
    'age': number,
    'job'?: string, // 不是每一个人都有工作的所以 job 属性是可选的
}


function printPeople(people: People) {
    console.log(people.jack); // 这样错误的拼写就能被检查出来了。
}
```
由于 jack 没有在“必要”声明当中 并且 jack 也没有出现在可选声明当中，所以 jack 不应该出现在 people 当中。使得 tsc 可以在编译时报错，让程序员尽可能早的发现问题。

```bash
tsc main.ts 
main.ts:9:24 - error TS2339: Property 'jack' does not exist on type 'People'.

9     console.log(people.jack);
                         ~~~~


Found 1 error.

```

---

## Readonly properties 
如果对象的一些属性自对象创建之后就不应该有更新，那么就应该把这个些属性声明为 readonly ，这也是 typescript 为了尽可能早的发现错误的一种手段。

---

## 声明语法
直接在属性前面加上 `readonly` 关键字。
```ts
interface People {
    readonly 'id': number // 身份证号这种定一下之后就不可能再改的属性可以加上 readonly 。
    'name': string,
    'age': number,
    'job'?: string,
}


let tim: People = { 'id': 100, 'name': 'tim', 'age': 16 };
// 更新时会报错
tim['id'] = 200;
```
---

## 只读属性的例子

```ts
interface People {
    readonly 'id': number // 身份证号这种定一下之后就不可能再改的属性可以加上 readonly 。
    'name': string,
    'age': number,
    'job'?: string,
}
```
编译。
```bash
tsc main.ts 
main.ts:10:5 - error TS2540: Cannot assign to ''id'' because it is a read-only property.

10 tim['id'] = 200;
       ~~~~


Found 1 error.
```
在对象创建后给只读属性赋值会报错。


---


## readonly 与 const
这两个关键字使用的场景不一样，如果作用的目标是“对象属性”那么应该用 `readonly` 如果目标是变量那么应该是 `const` 。

---



