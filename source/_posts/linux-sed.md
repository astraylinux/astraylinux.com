title: sed命令笔记
tags:
  - Linux
  - tools
  - Bash
categories:
  - Linux
date: 2014-02-16 15:50:44
keywords: Linux bash awk sed 脚本 服务器
---

前几天记录了一下awk的知识点，今天把sed也记录一下，毕竟它们是两好兄弟:D

>sed（意为流编辑器，源自英语“stream editor”的缩写）是Unix常见的命令行程序。sed用来把文档或字符串里面的文字经过一系列编辑命令转换为另一种格式输出。sed通常用来匹配一个或多个正则表达式的文本进行处理。
分号（;）可以用作分隔命令的指示符。尽管sed脚本固有的很多限制，一连串的sed指令加起来可以编程像 仓库番、快打砖块、甚至俄罗斯方块等电脑游戏的复杂程序。

<!--more-->

以上是来自wiki的介绍，sed同awk一样，支持脚本，可以实现比较复杂的功能。

sed是非交互式的编辑器。它不会修改文件，除非使用shell重定向来保存结果。

默认情况下，所有的输出行都被打印到屏幕上。

sed编辑器逐行处理文件（或输入），并将结果发送到屏幕。具体过程如下：

* 首先sed把当前正在处理的行保存在一个临时缓存区中（也称为模式空间）
* 然后处理临时缓冲区中的行，完成后把该行发送到屏幕上。
* sed每处理完一行就将其从临时缓冲区删除，然后将下一行读入，进行处理和显示。
* 处理完输入文件的最后一行后，sed便结束运行。

sed把每一行都存在临时缓冲区中，对这个副本进行编辑，所以不会修改原文件。

sed跟grep不一样，不管是否找到指定的模式，它的退出状态都是0。只有当命令存在语法错误时，sed的退出状态才不是0。

##定址
定址用于决定对哪些行进行编辑。地址的形式可以是数字、正则表达式、或二者的结合。如果没有指定地址，sed将处理输入文件的所有行。地址是一个数字(从1开始)，则表示行号；是“$"符号，则表示最后一行。范围地址用逗号分隔的，需要处理的地址是这两行之间的范围（包括这两行在内）。格式如下:

`sed '[定址][操作]' 文件`

例如;
```bash
#只打印第10行(-n 不输出原文， 操作p 打印)
sed -n '10p' file

#打印100到结束的行
sed -n '100,$p' file

#删除第10到20行
sed '10,20d' file

#删除包含'one' 到包含'two'这间的行
sed '/one/,/two/d' file

#删除包含'one'到第20行
sed '/one/,20d' file

#删除不包含'one'的行
sed '/one/!d' file
```

##命令与选项
`sed [option] 'command' file`

常用命令:

* /模式/p  ：打印匹配'模式'的行
* /模式/d  ：删除匹配'模式'的行
* s/模式/替换文字/  ：将匹配“模式”的行转换成“替换文字”
* s/模式/替换文字/g  ：将所有匹配“模式”的字符串转换成“替换文字”

常用选项:

* -f  ：按照指定的sed脚本里面的命令来进行转换
* -i  ：表示将转换结果直接插入文件中（若不用-i，一般sed命令不会改变原文档里的内容，而只会输出到命令行。当然命令行输出的内容也可以用“>”转存到另外一个文件里。）
* -e  ：表示在e后面的文字是正则表达式。有的版本不需要加注e选项也同样可以在命令中使用正则表达式。
* -n  ：取消默认的输出

完整命令列表：

|命令|功能|
|----|----|
|a\ |在当前行后添加一行或多行。多行时除最后一行外，每行末尾需用“\”续行
|c\ | 用此符号后的新文本替换当前行中的文本。多行时除最后一行外，每行末尾需用"\"续行
|i\ | 在当前行之前插入文本。多行时除最后一行外，每行末尾需用"\"续行
|d|删除行
|h|把模式空间里的内容复制到暂存缓冲区
|H|把模式空间里的内容追加到暂存缓冲区
|g|把暂存缓冲区里的内容复制到模式空间，覆盖原有的内容
|G|把暂存缓冲区的内容追加到模式空间里，追加在原有内容的后面
|l|列出非打印字符
|p|打印匹配的行
|=|打印匹配行号
|n|读入下一输入行，并从下一条命令而不是第一条命令开始对其的处理
|q|结束或退出sed
|r|从文件中读取输入行
|!|对所选行以外的所有行应用命令
|s|用一个字符串替换另一个
|g|在行内进行全局替换
|w|将所选的行写入文件
|x|交换暂存缓冲区与模式空间的内容
|y|将字符替换为另一字符（不能对正则表达式使用）例:'y/12345/ABCDE' 1换A, 2换B...

##正则表达式
与grep一样，sed也支持特殊元字符，来进行模式查找、替换。不同的是，sed使用的正则表达式是括在斜杠线"/"之间的模式。
如果要把正则表达式分隔符"/"改为另一个字符，比如'-'，只要在这个字符前加一个反斜线，在字符后跟上正则表达式，再跟上这个字符即可。例如：`sed -n '\-^My-p' file`

正则匹配元字符列表:


|元字符|功能|示例|
|------|----|----|
|^| 行首定位符| /^my/  匹配所有以my开头的行
|\$| 行尾定位符| /my$/  匹配所有以my结尾的行
|.| 匹配除换行符以外的单个字符| /m..y/  匹配包含字母m，后跟两个任意字符，再跟字母y的行
|* | 匹配零个或多个前导字符| /my*/  匹配包含字母m,后跟零个或多个y字母的行
|[]| 匹配指定字符组内的任一字符| /[Mm]y/  匹配包含My或my的行
|[^]| 匹配不在指定字符组内的任一字符| /[^Mm]y/  匹配包含y，但y之前的那个字符不是M或m的行
|`\(..\)`| 保存已匹配的字符|`1,20s/\(you\)self/\1r/ ` 标记元字符之间的模式，并将其保存为标签1，之后可以使用\1来引用它。最多可以定义9个标签，从左边开始编号.
|&| 保存查找串以便在替换串中引用| s/my/**&**/  符号&代表查找串。my将被替换为**my**
|<| 词首定位符| /\<my/  匹配包含以my开头的单词的行
|\>| 词尾定位符| /my\>/  匹配包含以my结尾的单词的行
|x\{m\}| 连续m个x| /9\{5\}/ 匹配包含连续5个9的行
|x\{m,\}| 至少m个x| /9\{5,\}/  匹配包含至少连续5个9的行
|x\{m,n\}| 至少m个，但不超过n个x| /9\{5,7\}/  匹配包含连续5到7个9的行

关于`\(..\), 1,20s/\(you\)self/\1r/ `
标记元字符之间的模式，并将其保存为标签1，之后可以使用\1来引用它。最多可以定义9个标签，从左边开始编号，最左边的是第一个。此例中，对第1到第20行进行处理，you被保存为标签1，如果发现youself，则替换为your。


##示例
### p命令
命令p用于显示模式空间的内容。默认情况下，sed把输入行打印在屏幕上，选项-n用于取消默认的打印操作。当选项-n和命令p同时出现时,sed可打印选定的内容。
```bash
#默认情况下，sed把所有输入行打印出来。这里进行匹配，p命令把印匹配到行再打印一次。
sed '/regex/p' file

#选项-n取消sed默认的打印，p命令把匹配模式regex的行打印一遍。
sed -n '/regex/p' file
```


### d命令

命令d用于删除输入行。sed先将输入行从文件复制到模式空间里，然后对该行执行sed命令，最后将模式空间里的内容显示在屏幕上。如果发出的是命令d，当前模式空间里的输入行会被删除，不被显示。
```bash
#删除最后一行，其余的都被显示
sed '$d' file

#删除包含regex的行，其余的都被显示
sed '/regex/d' file
```

### s命令

```bash
#命令末端的g表示在行内进行全局替换，也就是说如果某行出现多个one，所有的one被替换为two。
#这里one前加了'^',表达以one开头的才会被替换（都是正则)
sed 's/^one/two/g' file

#取消默认输出，处理1到20行里匹配以one结尾的行，把行内所有的one替换为two
sed -n '1,20s/one$/two/gp' file

#紧跟在s命令后的字符就是查找串和替换串之间的分隔符。分隔符默认为正斜杠，但可以改变。无论什么字符（换行符、反斜线除外），只要紧跟s命令，就成了新的串分隔符。
sed 's-one-two-g' file
```


### e选项
-e是编辑命令，用于sed执行多个编辑任务的情况下。在下一行开始编辑前，所有的编辑动作将应用到模式缓冲区中的行上。
```bash
#选项-e用于进行多重编辑。第一重编辑删除第1-3行。第二重编辑将出现的所有My替换为Your。因为是逐行进行这两项编辑（即这两个命令都在模式空间的当前行上执行），所以编辑命令的顺序会影响结果。
sed -e '1,10d' -e 's/one/two/g' file
```

### r命令

r命令是读命令。sed使用该命令将一个文本文件中的内容加到当前文件的特定位置上。
```bash
#如果在文件file的某一行匹配到模式one，就在该行后读入文件read.txt的内容。如果出现one的行不止一行，则在出现one的各行后都读入read.txt文件的内容。
sed '/one/r read.txt' file
```

### w命令
```bash
sed -n '/one/w one.txt' file
```


### a\ 命令
a\ 命令是追加命令，追加将添加新文本到文件中当前行（即读入模式缓冲区中的行）的后面。所追加的文本行位于sed命令的下方另起一行。如果要追加的内容超过一行，则每一行都必须以反斜线结束，最后一行除外。最后一行将以引号和文件名结束。
```bash
#如果在file文件中发现匹配以one开头的行，则在该行下面追two three
sed '/^one/a\
>two\
>three' file
```
### i\ 命令
i\ 命令是在当前行的前面插入新的文本。在有three的行前面插入两行。
```bash
sed '/three/i\
>one\
>two' file
```

### c\ 命令
sed使用该命令将已有文本修改成新的文本。
```bash
sed '/three/c\
>one\
>two' file

#上面这条命令跟s命令作用相似
sed 's/three/
>one\
>two/' file
```

### n命令
sed使用该命令获取输入文件的下一行，并将其读入到模式缓冲区中，任何sed命令都将应用到匹配行紧接着的下一行上。
```bash
#注：如果需要使用多条命令，或者需要在某个地址范围内嵌套地址，就必须用花括号将命令括起来，每行只写一条命令，或这用分号分割同一行中的多条命令。
#这里的意思是匹配到one的行，都对它的下一行执行将two替换成three
sed '/one/{n;s/two/three/;}' file
```
### y命令
该命令与UNIX/Linux中的tr命令类似，字符按照一对一的方式从左到右进行转换。例如，y/abc/ABC/将把所有小写的a转换成A，小写的b转换成B，小写的c转换成C。
```bash
#将1到20行内，所有的小写a转换成A,b转换成B，将1转换成^,将2转换成$。
#正则表达式元字符对y命令不起作用。与s命令的分隔符一样，斜线可以被替换成其它的字符。
sed '1,20y/ab12/AB^$/' file
```

### q命令

q命令将导致sed程序退出，不再进行其它的处理。
``` bash
sed '/hrwang/{s/one/two/;q;}' file
```

### h命令和g命令
```bash
#file 文件的内容
#cat file
1 ls ./
2 cat file
3 ls
4 more file

#通过下面两条命令，你会发现h会把原来暂存缓冲区的内容清除，只保存最近一次执行h时保存进去的模式空间的内容。而H命令则把每次匹配hrwnag的行都追加保存在暂存缓冲区。
sed -e '/ls/h' -e '$G' file
sed -e '/ls/H' -e '$G' file
#通过下面两条命令，你会发现g把暂存缓冲区中的内容替换掉了模式空间中当前行的内容，此处即替换了最后一行。而G命令则把暂存缓冲区的内容追加到了模式空间的当前行后。此处即追加到了末尾。
sed -e '/ls/H' -e '$g' file
sed -e '/ls/H' -e '$G' file
```