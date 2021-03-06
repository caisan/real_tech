## Jc的sed简单入门/sed命令总结
>
1. [Sed概述](#sed概述)
1. [Sed执行的流程](#sed执行的流程)
1. [选项options](#选项options)
1. [命令commands](#命令commands)

这是我学习sed的一个总结,只限于自己和比我水平低（就是没接触过）的同学看。。。除了参考资料,讲解也包括很大成分的个人理解,如果发现错误,希望大家可以提醒我及时更正,谢谢~!

# **sed概述**
sed是一个流编辑器，更准确的说是一个行编辑器，就是sed处理文本处理命令用于逐行处理文本中的文字。

这是sed命令的格式： **sed [options] [commands] [inputfile]


就是说sed命令一般由“sed”、选项、命令和待处理文本文件组成。
举个例子：
对于文本test.txt:
~~~
this is the first line.
the 2nd line.
hello, this is the 3rd line.
~~~
命令`sed -n '/hello/p' test` 就会产生以下输出：
~~~
hello, this is the 3rd line.
~~~
其中`-n`选项表示屏蔽sed默认输出。单引号中`/hello/`表示匹配带有"hello"的行`p`代表打印所匹配的行。

# **sed执行的流程**
1. 读取一行到模式空间缓冲区
2. 按照命令对模式空间缓冲区内容进行处理
3. 打印处理后缓冲区中的文本
4. 循环到第一步读取下一行，直到文本处理完

# **选项options**

像其他软件的参数一样,sed选项紧跟在sed后边,用'-'开头,并且多个参数可以写在一起

### **-n** 屏蔽默认打印
* 静默模式，相当于不进行执行流程中的第3步"打印处理后缓冲区中的文本"

### **-e** 使用多条命令组合
* 用于连续执行多个命令,形式:`sed -e [command1] -e [commaned2]... [inputfile]`
* 这里有一个花括号的等价形式:`sed {command1;command2;... [inputfile]}`

### **-f** 使用sed脚本
* 用于用sed脚本代替命令,形式: `sed -f [sed_script_file] [inputfile]`
* 其中[sed_script_file]是sed脚本,我们可以将一组sed命令编写成一个脚本文件 *.sed 来代替输入命令
* 如果脚本第一行写有解释器路径`#!/bin/sed -f` 并具有可执行权限,可以直接使用`[sed_script_file] [options] [inputfile]` 处理文件

### **-i** 直接修改原文件
* sed默认不修改原输入文件,除非加了-i参数,个人认为加此参数需要很谨慎
* sed也提供了更保险一点的参数 `-ibak`,使用它在修改原文件前会生成一个备份文件'*bak'

# **命令commands**

* sed的命令接在选项之后,用`''`包围.
* 每条一般包括两个部分,格式为'[匹配范围][指令]'('[range][command]'),其中匹配范围可以省略,但是指令不可以省略,如果省略匹配范围,则表示全部文本都会执行命令.

### 匹配范围
* 命令前可以加匹配来限制所要执行命令的行
* 匹配大致分为两类: **行号匹配**和**模式匹配**
* **行号匹配**的几种形式: `m,n` 表示从m行到n行; `m,+n` 表示从m行到m+n行; `m~n` 表示从m行开始每n行匹配依次(如`1~2`表示只匹配奇数行,`2~2`表示只匹配偶数行)
* **模式匹配**两边为'/'符号, 格式为`/pattern/`,其中pattern为**正则表达式**(正则表达式博大精深,可能比sed本身还要复杂,本文不讨论)
* 模式匹配可以和行号匹配进行结合,比如`2,/hello/`表示匹配从第2行到第二行后第一次匹配到'hello'的行
* `$`表示最后一行;`0`表示第一行前,也就是最开始;匹配后加`!`表示不匹配以上条件的行(类似取反)


### **p** 打印指令

* 打印筛选和处理后的行,常和-n一同使用,例如 `sed -n '1,4p' test.txt`会打印出文件的1-4行.

### **d** 删除指令

* 删除筛选和处理后的行,常在不加-n的时候使用,例如`sed '5,$d' test.txt`文件第5行到结尾会被删除而在本行处理后不打印处理,所以也会打印出1-4行,和上一条命令等价. 

### **w** 写文件指令

* 将处理后的行输出到文件中,感觉和`p`命令很类似,只是`w`是打印到文件中,用法为`sed [options] '[range]w [outfilename]' [filenmae]`,例如:`sed -n '1,4w out.txt' test.txt` 是将test.txt中的1-4行输出到out.txt

### **s** 替换指令

* 替换命令比较复杂, **格式**为:
`sed [options] '[range]s/origin_string/replace_string/[flags]' [filename]`
其中's'就像其他参数一样紧跟在匹配范围之后,'s'之后的部分都属于替换命令's'的"附属",可以和's'看为一个整体.
* 其**功能**就是把每行中的"origin_string"替换为"replace_string",具体怎么替换还和后边的"flag"标志有关
* "origin_string"的匹配方式类似与匹配范围的匹配,也是用正则表达式.
* "origin_string"中可以进行"捕获",将"捕获"到的字符子串用于"replace_string",方式是"origin_string"中用`\(...\)`捕获,"replace_string"中用'\[n]'回溯重现,例如:
用`sed -n "/hello/s/hello\(.*\)$/hi\1haha/p" test.txt` 处理test.txt:
~~~
hello jaycee~~
haha hello zjc
hi zhangjc!
~~~
将打印出:
~~~
hi jaycee~~haha
haha hi zjchaha
~~~
如果要回溯全部匹配字符串,可以用`&`

* **标志g:** 替换默认只对每行的第一处匹配进行,而加上`g`之后,将对每行所有匹配的字符串进行替换
* **标志[n]:**  数字标志,匹配到的第n次才进行替换
* **标志p:** 类似`p`命令,这里用作替换命令`s`的标志,作用都是对处理后的行进行打印,常和`-n`选项一起用
* **标志w:** 类似`w`命令,只不过做为了命令`s`的标志,作用和用法都相同
* **标志i:** 忽略大小写标志,加上i标志之后"origin_string"匹配时不去分大小写字母
* **标志e:** 把替换后的行作为shell命令执行
* 替换命令的标志使可以**组合**使用的,例如:`sed -n 's/hello/hi/gpw output.txt' test.txt`
* **分割标志**`/`可以用任意字符替换,sed根据`s`后边紧跟的第一个字符判断你用了什么做分隔符
* 匹配或者替换字符串出现**分割标志**的时候要用'\'进行转义zhuanyi

### **y** 转换字符命令

* 格式同替换命令`s`类似,作用是像字典一样,将origin_string中对应的字符替换为replace_string

### **a** 追加命令

* `a`用于在指定行**后面**追加其他文本,使用方法: `sed [options] '[range]a [string]' [filename]`
* 如果追加多行,[string]中用`\n`分割,表示回车

### **r** 从文件读入数据

* 效果和`a`类似,都是追加新行,用法和`w`类似,都是涉及文件的操作.
* 作用就是把文本文件中的文字打印到指定行后

### **i** 插入命令

* 与`a`类似, `i`用于在指定行**前面**插入其他文本,使用方法: `sed [options] '[range]i [string]' [filename]`
* 如果插入多行,[string]中用`\n`分割,表示回车

### **c** 修改命令

* 与`a`与'i'类似, `c`用于对指定行进行修改,使用方法: `sed [options] '[range]c [string]' [filename]`
* 如果修改为多行,[string]中用`\n`分割,表示回车

### **l** 打印不可见字符

* 类似于`p`参数,但`l`能打印处不可见得字符,如制表符`/t` 结束符`$`等

### **=** 打印行号

* 可以在输出行后打印一行该行行号

### 命令组合

* **匹配范围的组合:** `''`中可以加`{}`来组合多个匹配范围,例如:
`sed -n '1,5{/hello/p} [filename]'` 表示对于每一行会进行用来筛选打印文件1-5行中匹配'hello'的行.
`sed -n '/hello/{1,5{/world/p}}'`表示对于每一行,筛选打印出同时符合在1-5行且包括'hello'且包括'world'的行.

* **多条命令的组合**`''`中可以加`;`来分割多个命令,类似-e选项,有别于匹配范围的组合,`;`或者`-e`分割的多条命令会分别对每一行依次执行,例如:
`sed -n '1,5p;/hello/p [filename]` 对于每一行会连续执行两条指令,如果一行既在1-5行又包含'hello'则会打印两次,如果符合一个条件就打印一次.