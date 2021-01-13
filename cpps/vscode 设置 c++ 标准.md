## 背景
前段时间在看 [vscode 文档 ](https://code.visualstudio.com/docs/cpp/config-clang-mac) 中关于 c++ 的部分，感觉自己看懂了，今天建了个新的项目试了一下立马就 GG 了。详细的报错信息如下。
```bash
Build finished with errors(s):
/private/tmp/ups/main.cpp:9:20: error: non-aggregate type 'vector<std::__1::string>' (aka 'vector<basic_string<char, char_traits<char>, allocator<char> > >') cannot be initialized with an initializer list
    vector<string> message = {"hello","world."};
                   ^         ~~~~~~~~~~~~~~~~~~
/private/tmp/ups/main.cpp:10:26: warning: range-based for loop is a C++11 extension [-Wc++11-extensions]
    for(const string& msg:message){
                         ^
```
源码直接就是官方文档上的 hello-world
```c++
#include<iostream>
#include<vector>
#include<string>

using namespace std;

int main()
{
    vector<string> message = {"hello","world."};
    for(const string& msg:message){
        cout<<msg<<endl;
    }
    cout<<endl;

    return 0;
}
```

![npm-schema](static/2021-01/vscode.jpg)

---

## 分析与解决

从报错的内容可以知道我们的程序中使用了 c++11 的语法，但是我们没有给编译器指定所用的 c++ 标准；这个问题如果是用命令行的话比较好办，只要加上 `-std=c++11` 就好了，但是 vscode 编辑器又是在哪里指定的呢？

想到 vscode 的配置文件就那么三个，我就一个个找，看 `-std=c++11` 写在哪里合适，我比较走运，第一个就发现了 `.vscode/task.json` 配置文件里面定义了一些编译器相关的选项，默认情况下它的值如下。

```json
{
"version": "2.0.0",
"tasks": [
	{
		"type": "cppbuild",
		"label": "C/C++: clang++ build active file",
		"command": "/usr/bin/clang++",
		"args": [
			"-g",
			"${file}",
			"-o",
			"${fileDirname}/${fileBasenameNoExtension}"
		],
		"options": {
			"cwd": "${workspaceFolder}"
		},
		"problemMatcher": [
			"$gcc"
		],
		"group": "build",
		"detail": "compiler: /usr/bin/clang++"
	}
]
}
```
我指定项目的 c++ 版本为 c++17(17包含11)，那么配置文件只要这么一改就行了。
```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: clang++ build active file",
			"command": "/usr/bin/clang++",
			"args": [
				"-g",
				"${file}",
				"-o",
				"${fileDirname}/${fileBasenameNoExtension}",
				"-std=c++17"
			],
			"options": {
				"cwd": "${workspaceFolder}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "compiler: /usr/bin/clang++"
		}
	]
}
```
再次用 vscode 进行编译，错误不在了。
```bash

Starting build...
Build finished successfully.
```

---