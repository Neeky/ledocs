## 概要
用 typescript 实现一个二进制转二进制的算法。

---

## 栈
定义一个栈，用于保存余数。
```ts
class Stack {
    // 定义一个栈对象，用于保存余数。
    private items: number[];

    constructor() {
        this.items = [];
    }

    push(element: number) {
        // 向栈顶压入一个元素
        this.items.push(element);
    }

    pop() {
        // 从栈顶弹出一个元素
        return this.items.pop();
    }

    peek() {
        // 返回栈顶元素
        return this.items[this.items.length - 1];
    }

    isEmpty() {
        // 测试栈是否为空
        return this.items.length == 0;
    }

    size() {
        // 返回栈元素个数
        return this.items.length;
    }

    length() {
        // 返回栈元素个数
        return this.size()
    }

    clear() {
        this.items = [];
    }

    toString() {
        return this.items;
    }
}

```

---

## 进制转换
基于栈实现进制转换函数。
```ts
function decimalToBinary(decimal: number) {
    //  把十进制数转化为二进制数
    let binary_string = '';
    if (decimal < 0) {
        binary_string = '-'
        decimal = 0 - decimal;
    }
    else if (decimal === 0) {
        return "0"
    }
    else {
        binary_string = "+"
    }

    let stk = new Stack()
    while (decimal > 0) {
        stk.push(Math.floor(decimal % 2))
        decimal = Math.floor(decimal / 2)
    }

    let size = stk.size()
    for (let i = 0; i < size; i++) {
        binary_string = binary_string + stk.pop();
    }
    
    return binary_string;
}

console.log(decimalToBinary(-3)); // -11
console.log(decimalToBinary(-4)); // -100
console.log(decimalToBinary(0));  // 0
console.log(decimalToBinary(7));  // +111
```

