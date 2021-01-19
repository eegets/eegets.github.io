---
layout: post
title:  "Bash Commands Guide"
categories: Linux Base命令
---
Bash是一个交互式命令行解释器或shell。我想分享一下shell命令的常见用法以及它们是什么。我创建这个列表是为了节省时间。因为没有GUI，所有的事务在不使用鼠标的情况下移动得更快。

来源:https://medium.com/@duruldalkanat/bash-commands-guide-129c81cbfe87


###History

```
$ history              用于对前一个命令排序。
 
```

###Nslookup

```
$ nslookup             允许您查询由DNS服务器定向的IP地址
 
```



###ifconfig


```
$ ifconfig             显示机器的IP配置。
 
```


###ls


列出当前工作目录中的文件夹和文件名。

```
$ ls -a     列出所有文件，包括隐藏文件


$ ls -A     列出了所有文件，包括隐藏文件，除了一个顶级目录


$ ls -F     向条目添加一个指示符(*/=>@|中的一个)


$ ls -S     按文件大小排序


$ ls -al    提供同一目录中所有文件的列表


$ ls -l     使用较长的清单格式


$ ls -nl    使用带有用户ID的长清单格式


$ ls -c     按列列出内容
```

###MAN


它是用来查看系统参考手册的界面。man ls还显示了运行该命令的所有可用选项。

```
$ man       打印帮助信息并退出


$ man -V    版本显示版本信息并退出


$ man -C    使用配置文件而不是默认的~/.manpath。


$ man -d    打印调试信息。
```
###ENV


返回当前用户的环境变量列表。

```
$ env
```
###CHMOD


更改文件或目录的权限。

```
$ chmod 777      任何人都可以读取、写入和执行chmod 777 my_file


$ chmod 755f     文件应该是可读和可执行的，但只能由发行用户更改


$ chmod 700      只有用户可以对文件做任何事情
```
###CHOWN

更改文件和目录的所有权。

```
$ chown --help && chown --version

```
###PATH


它存储由冒号分隔的目录列表。

```
$ echo $PATH
```

###GREP


搜索文件中与模式匹配的行并返回结果。

```
$ grep "keyword" file.tx 在单个文件中搜索给定的字符串


$ grep -i "keyword" file.tx  该命令不区分大小写


$ grep -R "keyword" file.tx  搜索目录中的所有文件并输出包含匹配结果的文件名和行
```
###AWK


只提取特定事物的列表。

```
$ awk {keyword} file.tx
```

###OPEN


在finder中打开当前文件夹。


```
$ open .
```

###CHGRP


更改文件和目录的组。

```
$ chgrp groupname file.txt

```
###PWD


命令输出工作目录的名称

```
$ pwd
```

###CD


更改当前工作目录。

```
$ cd /       转到根目录


$ cd . .     转到父目录


$ cd         在不带参数执行时进入用户的主目录

$ cd ~root  当用户名在“~”后面给出时，转到用户的主目录

```


###HOME


显示主目录的路径。

```
$ echo $HOME

```

###NANO

命令行文本编辑器只接受键盘输入
```
$ nano myscript.sh
```

###FIND


搜索文件和目录。

```
$ find目录名文件名:按名称搜索文件


$ find directory -c             较新的文件:最后更改文件的时间比修改文件的时间更近


$ find目录-amin n:              该文件最后一次被访问是在n分钟之前


$ find目录-cmin n:              文件最后一次更改是在n分钟之前


$ find目录-atime n:             该文件最后一次被访问是在n天以前


$ find目录-mtime n:             文件的数据最后一次修改是在n天之前


$ find目录-ctime n:             文件最后一次更改是在n天以前


$ find directory -print:        显示使用其余条件找到的文件的路径名


$ find directory -exec:         对结果搜索结果执行命令
```
###MKDIR


使用此命令创建目录。
```
$ mkdir folder: 此命令将创建子文件夹
```

###MV


将一个文件或目录移动为另一个文件和目录。

```
mv -i source:     提示之前覆盖一个现有的文件

mv -f source:     总是覆盖现有的文件没有提示

mv -n source:     从不覆盖任何现有的文件

mv -v source:     打印源文件和目标文件
```

###RM


删除对象。

```
rm -f file:       在删除期间不询问问题，不为不存在的文件提供信息


rm -i file:       删除文件前提示


rm -r directory:  连续删除目录和子目录的内容


rm -v file:       提供有关删除过程的更详细信息
````

###RMDIR


删除目录，此命令仅在文件夹为空时才有效。

```
rmdir -p directoryname
```
###TOUCH


创建或打开一个文件并保存它，而不改变文件内容。

```
$ touch -file:       更新文件访问时间


$ touch -c file:     如果该文件不存在，它将创建该文件


$ touch -m file:     更改文件修改时间


$ touch -t file:     根据指定更改文件访问或替换时间
```
###CAT


显示文件的内容。

```
$ cat -n file:      通过为所有行提供一个数字，在屏幕上打印文件的内容
```
###WC


打印每个文件的换行、字和字节计数，如果指定了多个文件，则打印总数

```
$ wc -l file:       返回行数


$ wc -m file:       返回字符数


$ wc -w file:       计算字数
```
###HEAD


显示文件的第一行。

```
$ head file:        显示前几行
```
###TAIL


显示文件的最后几行。

```
$ tail file:        显示最后一个文件
```

###MORE

文件的内容用于显示页面
```
$ more file
```


###LESS

允许文件中的向后移动和向前移动


```
$ less file
```


###NL
每行添加一个行号
```
$ nl file
```



