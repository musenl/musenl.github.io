---
layout: post
title: grep与egrep的使用
categories: linux基础
description: grep egrep使用指南
keywords: grep, egrep,linux
---
grep（global search regular expression_r(RE) and print out the line,全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。Unix的grep家族包括grep、egrep和fgrep。egrep和fgrep的命令只跟grep有很小不同。egrep是grep的扩展，支持更多的re元字符， fgrep就是fixed grep或fast

grep，它们把所有的字母都看作单词，也就是说，正则表达式中的元字符表示回其自身的字面意义，不再特殊。linux使用GNU版本的grep。它功能更强，可以通过-G、-E、-F命令行选项来使用egrep和fgrep的功能。

grep的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到屏幕，不影响原文件内容。

grep可用于shell脚本，因为grep通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。


## 2. grep的用法及常用选项  

### 格式：
grep [options] 'PATTERN' file,...

eg：找出/etc/passwd下关于root的的行

```
[root@localhost ~]# grep 'root' /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```

### 常用选项：

|      选项       | 功能                       |
| :-----------: | :----------------------- |
|      -v       | 反向，显示不能被模式所匹配到的行         |
|      -o       | 仅显示被模式匹配到的字串，而不是整行       |
|      -i       | 不区分字符大小写（ignore-case）    |
|      -E       | 支持扩展的正则表达式与egrep具有相同意义   |
|      -A#      | 显示被模式匹配到的行及下方的#行（after）  |
|      -B#      | 显示被模式匹配到的行及上方的#行（before） |
|      -C#      | 显示被模式匹配到的行及上下#行          |
| --colour=auto | 匹配到字串标红                  |

## 3. 基本正则表达式的元字符
正则表达式：一类字符所书写出的模式（pattern），来处理字符串的方法，它是以行为单位来进行字符串的处理行为，正则表达式通过一些特殊称号的辅助，可以让用户轻易达到查找、删除、替换某特定字符串的处理程序。

元字符：不表示字符本身的意义，用于额外功能性的描述。

###（1）字符匹配

|        字符         | 意义                            |
| :---------------: | :---------------------------- |
|         .         | 任意单个字符                        |
|        []         | 指定范围内的单个字符                    |
| [0-9]或[[:digit:]] | 匹配数字0-9范围内的任意单个数字             |
| [a-z]或[[:lower:]] | 匹配小写字母a-z范围内的任意单个字母           |
|    [[:alpha:]]    | 匹配大小字母a-z和A-Z范围内的任意单个字母       |
|    [[:alnum:]]    | 匹配大小写字母a-z、A-Z、数字0-9范围内任意单个字符 |
|    [[:space:]]    | 匹配空格                          |
|    [[:punct:]]    | 匹配标点符号                        |
|        [^]        | 指定范围外的任意单个字符                  |

###（2）次数匹配 用来指定匹配其前面的字符的次数

|   字符    | 意义          | 例子                          |
| :-----: | :---------- | :-------------------------- |
|    *    | 任意次数        | eg：x*y ------  xxy，xy，y均被匹配 |
|   .*    | 匹配任意长度的任意字符 |                             |
|   \?    | 0次或1次       | eg：x\?y------ xy，y，xxy      |
|  \{m\}  | 匹配m次        |                             |
| \{m,n\} | 匹配m到n次      |                             |
| \{m,\}  | 匹配至少m次      |                             |
| \{0,n\} | 匹配至多n次      |                             |

```
[root@localhost ~]# grep --colour=auto "x\?y" xy
xxy
xy
y
```

以上代码之所以能匹配到xxy是因为匹配原则中的贪婪模式：即尽可能的长的去匹配字符

###（3）位置锚定

|   字符    | 意义              | 例子    |
| :-----: | :-------------- | :---- |
|    ^    | 锚定行首            | ^char |
|   \$    | 锚定行尾            | char$ |
|   ^$    | 空白行             |       |
| \\<char | 锚定词首或者使用\bchar  |       |
| char\\> | 锚定词尾 或者使用char\b |       |
eg：在/etc/passwd中锚定以r开头的单词

```
[root@localhost ~]# grep '\<[rR]' /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/cache/rpcbind:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
ricci:x:140:140:ricci daemon user:/var/lib/ricci:/sbin/nologin
```

###（4）分组

\\(\\)

eg: \\(ab\\)*xy 表示ab出现重复多次

###（5）引用

\1 后向引用，引用前面第1个左括号以及与之对应的右括号中的模式匹配到的内容

\2 后向引用，引用前面第2个左括号以及与之对应的右括号中的模式匹配到的内容

...

eg：\(a.b\)xy\1

文件内容：abxy，abbxy，a6bxya6b

表示：可以匹配到a6bxya6b
　
## 4. egrep的用法及扩展正则表达式的元字符

egrep是grep的扩展，与grep的不同在于它支持更多的扩展元字符。

### 1)字符匹配

|  字符  | 意义           |
| :--: | :----------- |
|  .   | 任意单个字符       |
|  []  | 指定范围内的单个字符   |
| [^]  | 指定范围外的任意单个字符 |

###（2）次数匹配 用来指定匹配其前面的字符的次数

|  字符   | 意义      |
| :---: | :------ |
|   *   | 任意次数    |
|   ?   | 0次或1次   |
|   +   | 匹配到至少1次 |
|  {m}  | 匹配m次    |
| {m,n} | 匹配m到n次  |
| {m,}  | 匹配至少m次  |
| {0,n} | 匹配至多n次  |

###（3）位置锚定

|   字符    | 意义              |  使用   |
| :-----: | :-------------- | :---: |
|    ^    | 锚定行首            | ^char |
|   \$    | 锚定行尾            | char$ |
|   ^$    | 空白行             |       |
| \\<char | 锚定词首 或者使用\bchar |       |
| char\\> | 锚定词尾 或者使用char\b |       |

###（4）分组

|  字符  | 意义   | 例子             |
| :--: | :--- | -------------- |
|  ()  | 分组   |                |
|  \|  | 或者   | eg：a\|b 表示a或者b |

## 5.实例

1、显示/proc/meminfo文件中以大小写s开头的行；

`# grep ^[sS] /proc/meminfo`

2、取出默认shell为非bash的用户；

`# grep -v 'bash$' /etc/passwd |cut -d: -f1`

3、取出默认shell为bash的且其ID号最大的用户；

`# grep 'bash$' /etc/passwd|sort -t: -k3|tail -1|cut -d: -f1`

4、显示/etc/rc.d/rc.sysinit文件中，以#开头，后面跟至少一个空白字符，而后又有至少一个非空白字符的行；

`# grep '^#[[:space:]]\{1,\}[^[:space:]]\{1,\}' /etc/rc.d/rc.sysinit`

5、显示/boot/grub/grub.conf中以至少一个空白字符开头的行；

`# grep "^[[:space:]]\{1,\}" /boot/grub/grub.conf`

6、找出/etc/passwd文件中一位数或两位数；

`# grep "\<[0-9]\{1,2\}\>" /etc/passwd`

7、找出ifconfig命令结果中的1到255之间的整数；

`# ifconfig | egrep "\<([1-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>"`

8、查看当前系统上root用户的所有信息;

`# grep "^root\>" /etc/passwd`

9、添加用户bash和testbash、basher，而后找出当前系统上其用户名和默认shell相同的用户；

`# grep "^\<\([[:alnum:]]\{1,\}\)\>.*\1$" /etc/passwd`

10、找出netstat -tan命令执行的结果中以“LISTEN”或“ESTABLISHED”结尾的行；

`# netstat -tan |egrep "\<LISTEN\>[[:space:]]*$|\<ESTABLISHED\>[[:space:]]*$"`

11、取出当前系统上所有用户的shell，要求：每种shell只显示一次，且按升序

`# cut -d: -f7 /etc/passwd| sort -u`

12、写一个模式，能匹配IP地址

`grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" file`


