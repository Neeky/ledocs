## 背景
最近在看 spring 官方文档的 AOP 部分，发现用到了动态代码，由于刚上手 java ，所以我把动态代码的最简化代码打一遍，方便自己以后理解 spring 的代码。

![](static/2021-02/eberhard-grossgasteiger-LiDs8Vs0cBE-unsplash.jpg)


---

## 动态代码是整体架构

![](static/2021-02/dynamic-proxy.svg)

总的来讲在使用动态代码模式的情况下，我只要实现三个地方 1、工作接口 2、真正实例工作接口的类 3、根据工作接口实现其对应的动态代码。

---

## 工作接口
```java
package org.example;

public interface Worker {

    String doWork();
}
```

---

## 工作类
```java
package org.example;

import java.lang.Override;

public class Coder implements Worker {

    @Override
    public String doWork() {
        return "System.out.println(\"hello java\")";
    }
}

```

---

## 工作接口的动态代理
```java
package org.example;


import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.InvocationHandler;

public class DynamicProxy implements InvocationHandler{

    private Worker worker;

    public void setWorker(Worker worker) {
        this.worker = worker;
    }

    public Object getProxy() {
        System.out.println("进入动态代码的 getProxy 方法.");
        Object proxy = Proxy.newProxyInstance(this.getClass().getClassLoader(),
            worker.getClass().getInterfaces(),this);
        return proxy;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // check
        checks();

        System.out.println("通过代理调用 " + method.getName() + " 方法.");
        Object result = method.invoke(this.worker,args);

        // clean
        clean();
        return result;
    }


    void checks() {
        System.out.println("功能执行前的检查操作 -- 由代理实现");
    }

    void clean() {
        System.out.println("功能执行后的清理操作 -- 由代理实现");
    }
}

```

---

## 使用方式
```java
package org.example;

public class App {
    public static void main(String[] args) {
        Coder coder = new Coder();
        DynamicProxy dp = new DynamicProxy();
        dp.setWorker(coder);

        //Worker proxy = (Worker) dp.getProxy();
        Object object = dp.getProxy();
        System.out.println("getProxy 返回对象的类型 " + object.getClass().getName());
        System.out.println("getProxy 返回对象的类型 " + object.getClass().getInterfaces()[0].getName());

        Worker proxy = (Worker) object;
        String result = proxy.doWork();

        // 打到 coder.doWork() 执行后的结果
        System.out.println(result);
    }

}

```
运行效果。
```bash
进入动态代码的 getProxy 方法.
getProxy 返回对象的类型 com.sun.proxy.$Proxy0
getProxy 返回对象的类型 org.example.Worker
功能执行前的检查操作 -- 由代理实现
通过代理调用 doWork 方法.
功能执行后的清理操作 -- 由代理实现
System.out.println("hello java")
```

---