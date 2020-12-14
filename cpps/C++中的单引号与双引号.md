## 概要
今天晚上跟着 `<<C++ primer plus>>` 一起学习，没想到刚上手就来了一异常。想到知识都还回去了，不知道大学能不能退钱。

```bash
/Users/jianglexing/repos/cpp-students/cpp_primer_plus/chapter4/src/4_4.cpp:15:17: warning: multi-character character constant [-Wmultichar]
    person p = {'tom',16};
                ^
/Users/jianglexing/repos/cpp-students/cpp_primer_plus/chapter4/src/4_4.cpp:15:17: error: no viable conversion from 'int' to 'std::__1::string' (aka
      'basic_string<char, char_traits<char>, allocator<char> >')
    person p = {'tom',16};
                ^~~~~
/Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/string:799:5: note: candidate constructor not viable: no known conversion from 'int' to
      'const std::__1::basic_string<char> &' for 1st argument
    basic_string(const basic_string& __str);
    ^
/Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/string:804:5: note: candidate constructor not viable: no known conversion from 'int' to
      'std::__1::basic_string<char> &&' for 1st argument
    basic_string(basic_string&& __str)
    ^
/Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/string:817:5: note: candidate constructor template not viable: no known conversion from 'int' to 'const char *' for
      1st argument
    basic_string(const _CharT* __s) : __r_(__default_init_tag(), __default_init_tag()) {
    ^
/Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/string:867:5: note: candidate constructor not viable: no known conversion from 'int' to 'initializer_list<char>' for
      1st argument
    basic_string(initializer_list<_CharT> __il);
    ^
1 warning and 1 error generated.
make[2]: *** [CMakeFiles/neeky.dir/src/4_4.o] Error 1
make[1]: *** [CMakeFiles/neeky.dir/all] Error 2
make: *** [all] Error 2
```
看到报错说是不能从 int 转 string ,突然就想到了我用错了单双引号。

C++ 中单引号只能来用表示字符，而本质上它是一个整数；双引号用来表示字符串。与 Python & JS 这类的语言不一样，这两个是有严格区分的。

![sqlpy](static/2020-46/string.jpg)

---

## 单引号
单引号本质上是整数
```c++
int 
main()
{
    int a = 'b';
    cout<<a<<endl;
    return 0;
}
```
运行效果如下。
```bash
make && ./neeky
[100%] Built target neeky
98
```

---

## 双引号
双引号的本质是字符串指针。
```c++
int 
main()
{
    char * name = "东风";
    cout<<name<<endl;
    return 0;
}
```
运行效果如下。
```bash
make && ./neeky
[100%] Linking CXX executable neeky
[100%] Built target neeky

东风
```
如果你使用的是 C++11 标准会看到下面这样一个告警。
```bash
/Users/jianglexing/repos/cpp-students/cpp_primer_plus/chapter4/src/4_4.cpp:15:19: warning: ISO C++11 does not allow conversion from string literal to 'char *'
      [-Wwritable-strings]
    char * name = "东风";
                  ^
1 warning generated.
```

---
