#BASH

##Bash 快捷键
-  `ctrl +a` ：光标移动行首
-  `ctrl +e` ：光标移动行尾
-  `ctrl +f ` ：光标右移一个字符
-  `ctrl +b` ：光标左移一个字符
-  `ctrl +l` ：清屏
-  `ctrl +u` ：删除光标至行首的字符
-  `ctrl +k` ：删除光标至行尾的字符
-  `ctrl +c` ：终止进程
-  `ctrl +z` ：挂起进程
-  `ctrl +w` ：删除光标前一个单词
-  `alt +d` ：删除光标后的一个单词
-  `TAB   ` ：代码自动补齐

## 重写向

标准的输入为`0`，标准的输出为`1`，标准的错误输出为`2`

如下示例：执行一安装脚本，正确信息重定向到null不输出，错误信息重写向到error.txt里面

eg: `./install.sh >/dev/null  2>error.txt`


##命令序列技巧

1. 使用 `；`  可以顺序执行命令；`ls /tmp; ls /root;`
2. 使用 `&&` 必须保证前面执行正确才执行后面的命令；`ls test.txt && cat test.txt`
3. 使用`||` 必须保证前面执行失败才执行后面的命令；`gedit a.txt || vim a.txt`

eg: `id tom &>/dev/null && echo 'hi tom' || echo 'no such user '  `


## 花括号{} 技巧
	
使用 `{}` 可以生成并列的项目,使用 `..` 连续的项目
		
- `echo {abc}`
- `echo user{1,2,3}`
- `mkdir /tmp/dir{1,2,3}`
- `echo {0..10}`


## 变量及环境变量

bash 变量语法 `name=value` , 使用变量时用 ` $name`

声明变量不赋值，`declare name` ,赋值时`name=value`

变量作用范围当前shell进程有效；


预设的系统环境变量

`PATH、PWD、HOME、HOSTNAME` 等

## 数组

语法：`name[index]=value`;  `sc[1]=100; sc[2]=110;`

读取语法：`$name[index]`

打印所有内容使用： `@/*`  echo sc[@]  或是 echo sc[@]

## 执行bash

- `/root/hi.sh` 绝对路径执行脚本
- `./hi.sh` 相对路径执行脚本
- `bash hi.sh` 通过bash调用执行
- `sh hi.sh` 通过 sh调用执行
- `source hi.sh` 通过source 运行
- `. hi.sh` 注意 . 和脚本之间有空格
