## 概要
几乎每年都能听到有人在生产环境，删库、删除文件这样的事。就拿 `rm -rf `这件情事来说吧，取证的过程中、安全人员是怎么拿到这个操作记录的呢？

![sqlpy](static/2020-24/sqlpy-conection.jpg)

---

## 原理分析
实际上我们向 linux 操作系统输入的任何命令最终都是交给了 /usr/bin/bash 这个程度来处理。更进一步地说，/usr/bin/bash 中有一个叫 readline 的函数，它会负责读取我们输入的命令；

如果要审计我们的操作内容。技术上讲，就是要实时地拿到这个函数的返回值并保存下来。有了数据事后的审计工作就有根据了。

以下是 /usr/bin/bash 源代码中对 readline 的定义。

```cpp

/* Read a line of input.  Prompt with PROMPT.  An empty PROMPT means
   none.  A return value of NULL means that EOF was encountered. */
char *
readline (const char *prompt)
{
  char *value;
  ...
  这里有一大堆逻辑
  ...
  return (value);
}
```

---


## 一个简单审计实现
原理清楚了，后面就是落地。这里还要补充另一个大家应该都知道的东西。现在应该已经没有直接运行在裸机上的应用程序了，应用程序都运行在内核之上。

要是想知道应用程序执行完一个函数的返回值，只需要让内核把这个函数的返回值，也返回一分给“审计”程序就行。如此简单！

说写就写，放心写一个 demo 并不长，20 行以内解决问题

```bpftrace

#!/usr/bin/env bpftrace

BEGIN 
{
    printf("Tracing bash readline function.\n");
    printf("%s %s\n", "pid", "cmd");
}

uretprobe:/bin/bash:readline 
{
    time("%H:%M:%S  ");
    printf("%-6d %s\n", pid, str(retval));
}
```

---

## 实际验证
第一步 把“审计”程序运行起来
```
./trace_bash_cmd.bt 
Attaching 2 probes...
Tracing bash readline function.
pid cmd
```

第二步 打开一个新的 shell 窗口连接到服务器并执行命令
```

[root@git-sqlpy-com ~]# ll /tmp/
total 24
drwxr-xr-x  6 root root 4096 Oct 10 21:07 hello-vue-3
drwx------  2 root root 4096 Oct 10 20:18 pyright-2831285-HxqpaT3spmZS
drwxr-xr-x  2 root root 4096 Oct 10 21:12 python-languageserver-cancellation
-rw-r--r--  1 root root    0 Oct 10 20:15 stargate.lock
drwx------  3 root root 4096 Oct 15 14:08 systemd-private-099a59c8279f4f079ea283b4cf961d83-systemd-hostnamed.service-qDtA30
drwxr-xr-x 18 root root 4096 Oct 14 12:00 v8
drwxr-xr-x  2 root root 4096 Oct 10 21:12 vscode-typescript0
[root@git-sqlpy-com ~]# rm -rf /tmp/v8
```

第三步 观察“审计”程序的输出结果
```

./trace_bash_cmd.bt 
Attaching 2 probes...
Tracing bash readline function.
pid cmd
14:08:51  2993767 ll /tmp/
14:09:04  2993767 rm -rf /tmp/v8
```

可以看到，就这么简单就能把输入的命令拿到了，所以啊！若要人不知，除非己莫为 。

---
   