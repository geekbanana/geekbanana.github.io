---
layout: single
title: Shell Script
excerpt: "shell script shell脚本"
author_profile: false
sidebar:
  nav: "linux"
categories: linux
---

## 什么是 shell script (shell 脚本)  
shell script 类似于DOS中的批处理文件(.bat), 可以将许多命令组合在一起, 用户执行一个shell script就可以执行多个命令.  

## shell script 注意事项  
1. 命令从上至下,从左至右地分析与执行  
2. 命令,参数间的多个空白会被忽略掉  
3. [tab]按键所得的空白同样视为空格键, 空白行也会被忽略掉  
4. 读到一个Enter符号(CR), 就开始执行该行命令  
5. 如果一行内容太多,可以使用`\[Enter]`来扩展至下一行  
6. "#"可作为注释.  


## shell script 执行方式  
假设程序为`/home/script/myshell.sh`  
- 直接命令执行:shell script必须具备可读可执行权限(RX)  
  - 绝对路径: 使用`/home/script/myshell.sh`执行  
  - 相对路径: 假设工作目录为/home/script/, 可以使用`./myshell.sh`来执行  
  - 变量"PATH"功能,将myshell.sh放在PATH内,即可痛过`myshell.sh`来执行  
- 以bash进程来执行  
  - `bash myshell.sh`  
  - `sh myshell.sh`  

## shell script 编写  

### 1.编写一个打印Hello World的shell脚本  
{% highlight shell linenos %}
#!/bin/bash
# Program:
#       This program shows "Hello World!" on your screen.
# History:
#       2016-12-12 20:03:57
# Author:
#       Cavalry

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World \a \n"
exit 0
{% endhighlight %}
第一行`#!/bin/bash`声明这个文件使用bash语法  
第2~7行,以#开头为注释  
引入PATH的好处是: 执行外部命令时, 可以不用写绝对路径  
主程序是 echo 那一行: 打印一个 Hello World  
`exit 0`中断程序执行,并回传一个0  

### 2.交互式脚本: 变量由用户输入  
用户输入first name 和 last name, 结果输出完整name  
{% highlight shell linenos %}
#!/bin/bash
# Program:
#       User input his first name and last name. Program shows his full name.
# History:
#       2016-12-12 20:24:28
# Author:
#       Cavalry
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please enter your first name: " firstname      # 提示用户输入
read -p "Please enter your last name: " lastname        # 提示用户输入
echo -e "\nYour full name is: $firstname $lastname"     # 结果由屏幕输处
{% endhighlight %}

