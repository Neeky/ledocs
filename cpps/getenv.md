## 背景
最近在看 MySQL 的源代码，不得不说对于一个初学者来说受益良多，至少可以养成一个严谨的习惯。

![npm-schema](static/2020-46/getenv.jpg)

---


## MySQL 源码一角
```c++
const char *tmpenv = getenv("MY_BASEDIR_VERSION");
if (tmpenv != nullptr) {
  strmake(mysql_home, tmpenv, sizeof(mysql_home) - 1);
} else {
  char progdir[FN_REFLEN];
  size_t dlen = 0;
  dirname_part(progdir, my_progname, &dlen);
  if (dlen > runtime_output_directory_addon.length() &&
      !strcmp(progdir + (dlen - runtime_output_directory_addon.length()),
              runtime_output_directory_addon.c_str())) {
    char cmake_binary_dir[FN_REFLEN];
    progdir[strlen(progdir) - 1] = '\0';  // remove trailing "/"
    dirname_part(cmake_binary_dir, progdir, &dlen);
    strmake(mysql_home, cmake_binary_dir, sizeof(mysql_home) - 1);
  } else {
    strcat(progdir, "/../");
    cleanup_dirname(mysql_home, progdir);
  }
}
```
对于尽可能使用常量指针，对于临时变量要用 tmp 打头，对于函数的返回值也要有检查。

---

## getenv

c++ 中使用 getenv 函数来获取给定环境变量的值，原型如下。
```cpp
char* getenv( const char* env_var );
```
当给定的环境变量存在的时候返回对应值的字符串指针，如果给定的变量名不存在就返回空指针。

---

## 样例

```cpp
#include<iostream>
#include<cstdlib> // getenv 在这个头文件中定义

int main()
{
    using namespace std;
    char * tmp_ptr = getenv("PATH");
    if(tmp_ptr == nullptr)
    {
        cout<<"got null point"<<endl;
        return 1;
    }
    string path = tmp_ptr;
    cout<<path<<endl;
}
```
运行效果
```bash
./neeky-test

/usr/local/mysql-8.0.22-linux-glibc2.12-x86_64/bin/:/usr/local/cmake-3.19.0-rc3-Linux-x86_64/bin:/usr/local/git-2.9.5/bin:/usr/local/node-v14.15.0-linux-x64/bin:/usr/local/python-3.8.6/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bi
```
---



