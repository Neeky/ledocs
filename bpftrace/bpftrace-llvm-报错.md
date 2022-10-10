## 背景
最近打算把自己的编译环境从 gcc 切到 clang ，切完之后用 bpftrace 的时候发现有如下报错.
```bash
bpftrace -l
bpftrace: symbol lookup error: /lib64/libclang-cpp.so.14: undefined symbol: _ZSt21__glibcxx_assert_failPKciS0_S0_, version LLVM_14


```

![mgr-error](static/2020-10/mgr-error.png)

---

## 分析与解决
看了下报错像是没有安装 llvm 引起的，于是安装了一下 llvm ，测试后发现问题解决了。
```bash
yum install llvm

bpftrace -l 
hardware:backend-stalls:
hardware:branch-instructions:
hardware:branch-misses:
...
...
```
---


## 总结
之前用 gcc 一套个就行了，看了一下 clang 和 llvm 才知道， clang 只负责了编译前端，所以它要和 llvm 配合使用才行。看来要安装的话还是要两个一起安装。
```bash
yum -y install clang llvm
```
---