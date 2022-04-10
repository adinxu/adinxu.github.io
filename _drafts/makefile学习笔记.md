makefile文件，见了不少，但自己不会写，在这里学习一下，看的是一位程序员前辈的[博客](https://blog.csdn.net/haoel/article/details/2886)，好像是2004年写的了，不过对我这种以入门为目的的人来说应该已经足够了。另外贴一下他的[新站的网址](https://coolshell.cn/articles/20977.html#%E4%B8%96%E7%95%8C%E5%8F%91%E5%B1%95%E8%B6%8B%E5%8A%BF)。github上有人以他的内容为基础整理了一番，[项目地址](https://github.com/seisman/how-to-write-makefile)，release里面有完成的pdf，也有在线版网站，方便阅读。  
另外，最原始版本的gnu make网址[在这](http://www.gnu.org/software/make/manual/make.html)。  
这里针对阅读内容做下记录。
# 程序编译顺序回顾
预处理->编译->汇编->链接  
预处理主要处理头文件包含，宏定义替换，条件编译等，输出.i文件。  
编译阶段将文本编译为汇编代码或中间码，到这为止是编译器的前端，输出.s文件。  
汇编阶段将汇编代码转换为二进制代码，这步已经与平台强相关，输出.o文件。  
链接阶段则是符号表与重定位处理，将其他.o或库文件链接,生成最终的可执行程序。  
# 关于编译工具
## cc,gcc,g++
可以看以下资料：
1. [Linux 下 的 cc 和 gcc](https://www.cnblogs.com/zhouyinhui/archive/2010/02/01/1661078.html)
2. [g++ gcc 的区别](https://www.cnblogs.com/xiedan/archive/2009/10/25/1589462.html)  

简单来说，cc(c compiler)是一个编译器，而gcc(GNU compiler collection)是编译器集合。如果在linux下gcc命令与cc是等效的，因为cc是软链接到gcc的。
## pkg-config
man手册里介绍说，pkg-config可以返回已安装库的元信息。所以可以用于makefile脚本里面。
## cmake
CMake(cross platform make)是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。
## automake
automake能够根据`Makefile.am`的配置来生成`Makefile.in`.后续执行./configure即可生成Makefile文件。
# makefile概览
makefile是make命令的执行对象，它告诉make命令需要怎么样的去编译和链接程序。
## 基本规则
1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
2. 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。
## 核心语法
```makefile
target ... : prerequisites ...
    command
    ...
    ...
```
target  
可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。对于标签这种特性，在后续的“伪目标”章节中会有叙述。  
prerequisites  
生成该target所依赖的文件和/或target
command
该target要执行的命令（任意的shell命令）  

这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:  
prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。
## 一个示例makefile
```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```
### 使用方法
把上述文件保存至"makefile"或"Makefile"文件中，然后在该目录下直接输入命令 make 就可以生成执行文件edit。如果要删除执行文件和所有的中间目标文件，那么，只要简单地执行一下 make clean 就可以了。  
### 文件释义 
反斜杠（ \ ）和c语言中一样，是换行符的意思。  
在这个makefile中，目标文件（target）包含：执行文件edit和中间目标文件（ *.o ），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h 文件。  
每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是执行文件 edit 的依赖文件。依赖关系的实质就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。  
在定义好依赖关系后，后续的那一行定义了如何生成目标文件的操作系统命令，一定要以一个 `Tab` 键作为开头。  
记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。
### makefile动作
这里要说明一点的是， clean 不是一个文件，它只不过是一个动作名字，有点像c语言中的label一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个label的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。
### make编译流程
以上面的Makefile为例：
1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 .o 文件的文件修改时间要比 edit 这个文件新，那么，他就会执行后面所定义的命令来生成 edit 这个文件。
4. 如果 edit 所依赖的 .o 文件也不存在，那么make会在当前文件中找目标为 .o 文件的依赖性，如果找到则再根据那一个规则生成 .o 文件。（这有点像一个堆栈的过程）
5. 当然，你的C文件和H文件是存在的啦，于是make会生成 .o 文件，然后再用 .o 文件生成make的终极任务，也就是执行文件 edit 了。
找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。  
像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即命令—— `make clean` ，以此来清除所有的目标文件，以便重编译。  
### makefile变量
为了makefile的易维护，在makefile中我们可以使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏可能会更好。
比如在刚刚的头文件开头加入：
```makefile
objects = main.o kbd.o command.o display.o \
     insert.o search.o files.o utils.o
```
那么后续在所有使用到这些文件的地方，就可以使用`$(objects)`来使用这个变量。  
新的makefile:
```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit $(objects)
```
### makefile自动推导
GNU的make很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个 .o 文件后都写上类似的命令（比如依赖的c文件和编译命令），因为，我们的make会自动识别，并自己推导命令。
makefile可以再次被简化为以下版本：
```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    rm edit $(objects)
```
`.PHONY` 表示 clean 是个伪目标文件。
更稳健的写法是：
```makefile
.PHONY : clean
clean :
    -rm edit $(objects)
```
在 rm 命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。当然， clean 的规则不要放在文件的开头，不然，这就会变成make的默认目标，相信谁也不愿意这样。不成文的规矩是——“clean从来都是放在文件的最后”。
### 另一种风格的makefile
上面好多.o的依赖头文件其实是一样的，makefile文件可以写成这样：
```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```
看自己喜好选择。
### makefile基本要素
Makefile里主要包含了五个东西：显式规则、隐晦规则、变量定义、文件指示和注释。
1. 显式规则。显式规则说明了如何生成一个或多个目标文件。这是由Makefile的书写者明显指出要生成的文件、文件的依赖文件和生成的命令。
2. 隐晦规则。由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较简略地书写 Makefile，这是由make所支持的。
3. 变量的定义。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
4. 文件指示。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。
5. 注释。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用 # 字符，这个就像C/C++中的 // 一样。如果你要在你的Makefile中使用 # 字符，可以用反斜杠进行转义，如： `\#` 。
最后，还值得一提的是，在Makefile中的命令，必须要以 Tab 键开始。
### 显式指定makefile
大多数的make都支持“makefile”和“Makefile”这两种默认文件名。如果你的makefile不是使用这两种默认的文件名，可以使用make的 `-f` 和 `--file` 参数，如： `make -f Make.Linux` 或 `make --file Make.AIX` 。
### makefile引用
在Makefile使用 include 关键字可以把别的Makefile包含进来，这很像C语言的 #include ，被包含的文件会原模原样的放在当前文件的包含位置。 include 的语法是：
```makefile
include <filename>
```
在 include 前面可以有一些空字符，但是绝不能是 Tab 键开始。  
filename 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）。 如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找： 
1. 如果make执行时，有 -I 或 --include-dir 参数，那么make就会在这个参数所指定的目录下去寻找。
2. 如果目录 <prefix>/include （一般是： /usr/local/bin 或 /usr/include ）存在的话，make也会去找。
   
注意：不会搜索VPATH或vpath指定的路径
如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”。如：
```makefile
-include <filename>
```
### make工作方式
1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐晦规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

## 书写规则
### 在规则中的通配符
通配符有三种：`~`,`*`,`?`.
需要注意的是，如果通配符用于变量，不会自动展开，所以可能不会按你的预期进行工作。若想在变量中使用通配，需要使用makefile函数。
### 文件搜寻
当项目规模较小时，指定源文件还不算太麻烦，但当在一些大的工程中有大量的源文件时，我们往往会根据功能把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make在自动去找。
方法一：VPATH变量
Makefile中的特殊变量 VPATH 可以完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件了。以下是使用实例：
```makefile
VPATH = src:../headers
```
上述语句代表make会按照“src”和“../headers”的顺序来搜寻，目录由“冒号”分隔。（需要注意，当前目录永远是最高优先搜索的地方）。
2.vpath关键字
vpath使用方法如下：
```
vpath <pattern> <directories>'
```
为符合模式<pattern>的文件指定搜索目录<directories>。
```
vpath <pattern>
```
清除符合模式<pattern>的文件的搜索目录。
```
vpath
```
清除所有已被设置好了的文件搜索目录。

vapth使用方法中的<pattern>需要包含 % 字符。 % 的意思是匹配零或若干字符，（需引用 % ，使用 \ ）例如， %.h 表示所有以 .h 结尾的文件。<pattern>指定了要搜索的文件集，而<directories>则指定了< pattern>的文件集的搜索的目录。

下面是使用实例：
```
 #VPATH=src:public
 ```
 ```
vpath %.c src
vpath %.h public
```
上面两段的作用是相同的，指定搜索目录为src和public，但是很明显使用vpath更加灵活。如果你同种文件的目录有多个，可以采用省略写法。
```
vpath %.c foo:zoo
vpath %   blish
vpath %.c bar
```
其表示 .c 结尾的文件，先在“foo”目录，"zoo"目录，然后是“blish”，最后是“bar”目录。
### 伪目标
前例中的clean就是伪目标，它并不是一个文件，只是一个标签，由于“伪目标”不是文件，所以make无法生成它的依赖关系和决定它是否要执行。我们只有通过显式地指明这个“目标”才能让其生效，我们并不生成“clean”这个文件。
“伪目标”的取名不能和文件名重名，不然其就失去了“伪目标”的意义了。
通过.PHONY标记显式的指明这是一个伪目标：
```
.PHONY : clean
clean :
    rm *.o temp
```
查看下面这个实例：
```bash
#开始时使用了.PHONY
$ ll
total 40K
-rw-r--r-- 1 sss sss  184 Apr  7 00:33 Makefile
-rw-r--r-- 1 sss sss 1.6K Apr  7 00:33 func.o
-rwxr-xr-x 1 sss sss  17K Apr  7 00:33 helloworld
-rw-r--r-- 1 sss sss 1.7K Apr  7 00:33 main.o
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:21 public
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:23 src
$ touch clean
$ make clean
rm helloworld main.o func.o
#可以正常删除
$ ll
total 12K
-rw-r--r-- 1 sss sss  184 Apr  7 00:33 Makefile
-rw-r--r-- 1 sss sss    0 Apr  7 00:33 clean
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:21 public
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:23 src
$ vim Makefile
#此处删除.PHONY
$ make
cc    -c -o main.o src/main.c
cc    -c -o func.o src/func.c
cc -o helloworld main.o func.o
$ make clean
make: 'clean' is up to date.
$ ll
total 40K
-rw-r--r-- 1 sss sss  185 Apr  7 00:34 Makefile
-rw-r--r-- 1 sss sss    0 Apr  7 00:33 clean
-rw-r--r-- 1 sss sss 1.6K Apr  7 00:34 func.o
-rwxr-xr-x 1 sss sss  17K Apr  7 00:34 helloworld
-rw-r--r-- 1 sss sss 1.7K Apr  7 00:34 main.o
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:21 public
drwxr-xr-x 2 sss sss 4.0K Apr  6 23:23 src
#不能正常删除
 ```
伪目标一般没有依赖的文件。但是，我们也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个。一个示例就是，如果你的Makefile需要一口气生成若干个可执行文件，但你只想简单地敲一个make完事，并且，所有的目标文件都写在一个Makefile中，那么你可以使用“伪目标”这个特性：
```
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o

prog2 : prog2.o
    cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```
从上面的例子我们可以看出，目标也可以成为依赖。所以，伪目标同样也可成为依赖。看下面的例子：
```
.PHONY : cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
    rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```
“make cleanall”将清除所有要被清除的文件。“cleanobj”和“cleandiff”这两个伪目标有点像“子程序”的意思。我们可以输入“make cleanall”和“make cleanobj”和“make cleandiff”命令来达到清除不同种类文件的目的。
### 静态模式
语法如下：
```
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
```
targets定义了一系列的目标文件，可以有通配符。是目标的一个集合。  
target-pattern是指明了targets的模式，也就是的目标集模式。  
prereq-patterns是目标的依赖模式，它对target-pattern形成的模式再进行一次依赖目标的定义。  
使用实例：
```
objects=main.o func.o
helloworld:$(objects)
        cc -o helloworld $(objects)

$(objects):%.o:%.c
        cc -c $<
```
上面的语句等同以下语句：
```
objects=main.o func.o
helloworld:$(objects)
        cc -o helloworld $(objects)

main.o:main.c
        cc -c $<

func.o:func.c
        cc -c $<
```
### 自我生成依赖性
大型的工程中，用上述提到的那些方式编写makefile时，你必需清楚哪些C文件包含了哪些头文件，并且，你在加入或删除头文件时，也需要小心地修改Makefile，这是一个很没有维护性的工作。为了避免这种繁重而又容易出错的事情，我们可以使用C/C++编译的一个功能。大多数的C/C++编译器都支持一个“-M”的选项，即自动找寻源文件中包含的头文件，并生成一个依赖关系。
```
$cc -M src/main.c
main.o: src/main.c /usr/include/stdc-predef.h /usr/include/stdlib.h \
 /usr/include/x86_64-linux-gnu/bits/libc-header-start.h \
 ...
 ...
 ```
 这样就可以由编译器自动生成的依赖关系，你就不必再手动书写若干文件的依赖关系，而由编译器自动生成了。  
 需要提醒一句的是，如果你使用GNU的C/C++编译器，你得用 -MM 参数，不然， -M 参数会把一些标准库的头文件也包含进来，就像上述其实会输出很多行。
 ```
$  cc -MM src/main.c
main.o: src/main.c src/../public/func.h
```
那么如何利用这些命令来进行编译工作呢？  
GNU组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个文件中，为每一个 name.c 的文件都生成一个 name.d 的Makefile文件， .d 文件中就存放对应 .c 文件的依赖关系。  
于是，我们可以写出 .c 文件和 .d 文件的依赖关系，并让make自动更新或生成 .d 文件，并把其包含在我们的主Makefile中，这样，我们就可以自动化地生成每个文件的依赖关系了。
```
%.d: %.c
    @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```
上面是一个模式规则来产生 .d 文件，比如对于main.c做实例讲解，set -e表示有错误立即退出。首先删除原来的main.d文件，然后生成main.d.4398，$$$$代表四位随机数，然后将新生成的main.d.4398用sed来替换，main.d.4398内容如下：
```
main.o: main.c ../public/func.h
```
sed正则的意思是将main.d加入依赖关系，因为$*是%及之前的部分，即main，正则保留main.o的同时，加入目标main.d。
转换后的mian.d如下：
```
main.o main.d : main.c ../public/func.h
```
于是，我们的 .d 文件也会自动更新了，并会自动生成了，当然，你还可以在这个 .d 文件中加入的不只是依赖关系，包括生成的命令也可一并加入，让每个 .d 文件都包含一个完赖的规则。一旦我们完成这个工作，接下来，我们就要把这些自动生成的规则放进我们的主Makefile中。我们可以使用Makefile的“include”命令，来引入别的Makefile文件，例如：
```
sources = main.c func.c
include $(sources:.c=.d)
```
上述语句中的 $(sources:.c=.d) 中的 .c=.d 的意思是做一个替换，把变量 $(sources) 所有 .c 的字串都替换成 .d ，即为include了main.d和func.d。当然，你得注意次序，因为include是按次序来载入文件，最先载入的 .d 文件中的目标会成为默认目标。
下面又是实例环节：
```
#vpath %.c app:cliplugin:lib:resource
#vpath %.h include
#vpath %.d app:cliplugin:lib:resource
VPATH=app:cliplugin:lib:resource:include

OBJS=main.o cli.o lib.o resource.o
TARGETS=makefile_test

$(TARGETS):$(OBJS)
        cc -o $@ $(OBJS)

include $(OBJS:.o=.d)

%.d:%.c
        cc -MM $< > $@.$$$$; \
        sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
        rm -f $@.$$$$

clean:
        @rm -rf $(TARGETS) $(OBJS) $(OBJS:.o=.d)
```
其中include那一行引用了所有.d文件，如果不存在会通过下面的模式规则来生成并包含进主makefile中来。因为include是在检查依赖前进行的，所以当检查到TARGETS需要OBJS时，关于OBJS的依赖规则此时已经被包含进来了。
## 书写命令
每条规则中的命令和操作系统Shell的命令行是一致的。make会一按顺序一条一条的执行命令，每条命令的开头必须以 Tab 键开头，除非，命令是紧跟在依赖规则后面的分号后的。在命令行之间中的空格或是空行会被忽略，但是如果该空格或空行是以Tab键开头的，那么make会认为其是一个空命令。
我们在UNIX下可能会使用不同的Shell，但是make的命令默认是被 /bin/sh ——UNIX的标准Shell 解释执行的。除非你特别指定一个其它的Shell。Makefile中， # 是注释符，很像C/C++中的 // ，其后的本行字符都被注释。
### 显示命令
make执行时的命令会被打印，比如echo，那么执行make时会将命令本身也显示出来：
```
echo 正在编译XXX模块......
正在编译XXX模块......
```
如果使用@加在命令前，则不会打印执行的命令本身。  
如果make执行时，带入make参数 -n 或 --just-print ，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。  
而make参数 -s 或 --silent 或 --quiet 则是全面禁止命令的显示。
### 命令执行
如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令。最典型的就是依赖于路径的命令。
下面两个makefile的输出是不同的：
```
exec:
    cd /home/xwd
    pwd
```
其实输出当前目录
```
exec:
    cd /home/xwd; pwd
```
这个会输出/home/xwd  
make一般是使用环境变量SHELL中所定义的系统Shell来执行命令，默认情况下使用UNIX的标准Shell——/bin/sh来执行命令。但在MS-DOS下有点特殊，因为MS-DOS下没有SHELL环境变量，当然你也可以指定。如果你指定了UNIX风格的目录形式，首先，make会在SHELL所指定的路径中找寻命令解释器，如果找不到，其会在当前盘符中的当前目录中寻找，如果再找不到，其会在PATH环境变量中所定义的所有路径中寻找。MS-DOS中，如果你定义的命令解释器没有找到，其会给你的命令解释器加上诸如 .exe 、 .com 、 .bat 、 .sh 等后缀。
### 命令出错
如果一个规则中的某个命令出错了（命令退出码非零），那么make就会终止执行当前规则，这将有可能终止所有规则的执行。有些时候，命令的出错并不表示就是错误的。如果想忽略命令的出错，我们可以在Makefile的命令行前加一个减号 -。
```
clean:
    -rm -f *.o
```
这样不管命令出不出错都认为是成功的。
还有一个全局的办法是，给make加上 -i 或是 --ignore-errors 参数，那么，Makefile中所有命令都会忽略错误。而如果一个规则是以 .IGNORE 作为目标的，那么这个规则中的所有命令将会忽略错误。这些是不同级别的防止命令出错的方法，你可以根据你的不同喜欢设置。  
还有一个要提一下的make的参数的是 -k 或是 --keep-going ，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则。
### 嵌套执行make
```
subsystem:
    cd subdir && $(MAKE)
```
其等价于：
```
subsystem:
    $(MAKE) -C subdir
```
定义$(MAKE)宏变量的意思是，也许我们的make需要一些参数，所以定义成一个变量比较利于维护。这两个例子的意思都是先进入“subdir”目录，然后执行make命令。 
我们把这个Makefile叫做“总控Makefile”，总控Makefile的变量可以传递到下级的Makefile中（如果你显示的声明），但是不会覆盖下层的Makefile中所定义的变量，除非指定了 -e 参数。  
传递变量的声明方式如下：
```
export <variable ...>;
````
如果你不想让某些变量传递到下级Makefile中，那么你可以这样声明:
```
unexport <variable ...>;=
```
如果你要传递所有的变量，那么，只要一个export就行了。后面什么也不用跟，表示传递所有的变量。  
需要注意的是，有两个变量，一个是 SHELL ，一个是 MAKEFLAGS ，这两个变量不管你是否export，其总是要传递到下层 Makefile中，特别是 MAKEFLAGS 变量，其中包含了make的参数信息，如果我们执行“总控Makefile”时有make参数或是在上层 Makefile中定义了这个变量，那么 MAKEFLAGS 变量将会是这些参数，并会传递到下层Makefile中，这是一个系统级的环境变量。  
但是make命令中的有几个参数并不往下传递，它们是 -C , -f , -h, -o 和 -W （有关Makefile参数的细节将在后面说明），如果你不想往下层传递参数，那么，你可以这样来写：
```
subsystem:
    cd subdir && $(MAKE) MAKEFLAGS=
```
如果你定义了环境变量 MAKEFLAGS ，那么你得确信其中的选项是大家都会用到的，如果其中有 -t , -n 和 -q 参数，那么将会有让你意想不到的结果，或许会让你异常地恐慌。  
还有一个在“嵌套执行”中比较有用的参数， -w 或是 --print-directory 会在make的过程中输出一些信息，让你看到目前的工作目录。  
当你使用 -C 参数来指定make下层Makefile时， -w 会被自动打开。如果参数中有 -s （ --slient ）或是 --no-print-directory ，那么， -w 总是失效的。  

### 定义命令包
变量不仅可以替代一系列目标或字符串，还能定义为一系列命令序列：
```
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```
上面的语句把run-yacc这个变量定义为下面的两个命令。  
这里，“run-yacc”是这个命令包的名字，其不要和Makefile中的变量重名。在 define 和 endef 中的两行就是命令序列。这个命令包中的第一个命令是运行Yacc程序，因为Yacc程序总是生成“y.tab.c”的文件，所以第二行的命令就是把这个文件改改名字。  
使用实例：
```
foo.c : foo.y
    $(run-yacc)
```
我们可以看见，要使用这个命令包，我们就好像使用变量一样。在这个命令包的使用中，命令包“run-yacc”中的 $^ 就是 foo.y ， $@ 就是 foo.c （有关这种以 $ 开头的特殊变量，我们会在后面介绍），make在执行命令包时，命令包中的每个命令会被依次独立执行。

## 使用变量
在Makefile中的定义的变量，就像是C/C++语言中的宏一样，他代表了一个文本字串，在Makefile中执行的时候其会自动原模原样地展开在所使用的地方。其与C/C++所不同的是，你可以在Makefile中改变其值。在Makefile中，变量可以使用在“目标”，“依赖目标”， “命令”或是Makefile的其它部分中。  
变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有 : 、 # 、 = 或是空字符（空格、回车等）。变量是大小写敏感的，“foo”、“Foo”和“FOO”是三个不同的变量名。传统的Makefile的变量名是全大写的命名方式，但我推荐使用大小写搭配的变量名，如：MakeFlags。这样可以避免和系统的变量冲突，而发生意外的事情。
### 变量中的变量
1.`=`
在 = 左侧是变量，右侧是变量的值，右侧变量的值可以定义在文件的任何一处，也就是说，右侧中的变量不一定非要是已定义好的值，其也可以使用后面定义的值。如：
```
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:
    echo $(foo)
```
执行“make all”将会打出变量 $(foo) 的值是 Huh?  
这种形式也有不好的地方，那就是递归定义，如：  
```
A = $(B)
B = $(A)
```
2.`:=`
为了避免上面的这种方法，我们可以使用make中的另一种用变量来定义变量的方法。这种方法使用的是 := 操作符，如：  
这种方法，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量。如果是这样：  
```
y := $(x) bar
x := foo
```
那么，y的值是“bar”，而不是“foo bar”。  
看下面的例子：  
```
ifeq (0,${MAKELEVEL})
cur-dir   := $(shell pwd)
whoami    := $(shell whoami)
host-type := $(shell arch)
MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```
对于系统变量“MAKELEVEL”，其意思是，如果我们的make有一个嵌套执行的动作（参见前面的“嵌套使用make”），那么，这个变量会记录了我们的当前Makefile的调用层数。  
定义空格变量  
```
nullstring :=
space := $(nullstring) # end of the line
```
nullstring是一个Empty变量，其中什么也没有，而我们的space的值是一个空格。因为在操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止，这样，我们可以定义出其值是一个空格的变量。  
请注意这里关于“#”的使用，注释符“#”的这种特性值得我们注意，如果我们这样定义一个变量：  
```
dir := /foo/bar    # directory to put the frobs in
```
dir这个变量的值是“/foo/bar”，后面还跟了4个空格，如果我们这样使用这样变量来指定别的目录——“$(dir)/file”那么就完蛋了。  
3.`?=`
`?=`含义是，如果FOO没有被定义过，那么变量FOO的值就是“bar”，如果FOO先前被定义过，那么这条语将什么也不做，
```
FOO ?= bar
```
其等价于：
```
ifeq ($(origin FOO), undefined)
    FOO = bar
endif
```
## 变量的高级用法
### 变量值替换
```
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```
这个示例中，我们先定义了一个 $(foo) 变量，而第二行的意思是把 $(foo) 中所有以 .o 字串“结尾”全部替换成 .c ，所以我们的 $(bar) 的值就是“a.c b.c c.c”。  
另外一种变量替换的技术是以“静态模式”（参见前面章节）定义的，如：  
```
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```
这个示例的效果和上面是相同的。
### 用变量定义变量
```
x = y
y = z
a := $($(x))
```
最后$(a)的值为z。
在这种方式中，可以使用多个变量来组成一个变量的名字，然后再取其值：
```
first_second = Hello
a = first
b = second
all = $($a_$b)
```
这里的 $a_$b 组成了“first_second”，于是， $(all) 的值就是“Hello”。
两种结合则为:
```
a_objects := a.o b.o c.o
1_objects := 1.o 2.o 3.o

sources := $($(a1)_objects:.o=.c)
```
这个例子中，如果 $(a1) 的值是“a”的话，那么， $(sources) 的值就是“a.c b.c c.c”；如果 $(a1) 的值是“1”，那么 $(sources) 的值是“1.c 2.c 3.c”。
下面这种方法类似与函数指针：
```
ifdef do_sort
    func := sort
else
    func := strip
endif

bar := a d b g q c

foo := $($(func) $(bar))
```
下面这个例子中定义了三个变量：“dir”，“foo_sources”和“foo_print”：
```
dir = foo
$(dir)_sources := $(wildcard $(dir)/*.c)
define $(dir)_print
lpr $($(dir)_sources)
endef
```
### 追加变量值
使用 += 操作符给变量追加值，如：
```
objects = main.o foo.o bar.o utils.o
objects += another.o
```
$(objects) 值变成：“main.o foo.o bar.o utils.o another.o”（another.o被追加进去了）  
如果变量之前没有定义过，那么， += 会自动变成 = ，如果前面有变量定义，那么 += 会继承于前次操作的赋值符。如果前一次的是 := ，那么 += 会以 := 作为其赋值符
### override 指示符
如果有变量是通常make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。如果你想在Makefile中设置这类参数的值，那么，你可以使用“override”指示符。其语法是:  
```
override <variable>; = <value>;

override <variable>; := <value>;
```
也可以追加：
```
override <variable>; += <more text>;
```
对于多行的变量定义，我们用define指示符，在define指示符前，也同样可以使用override指示符，如:  
```
override define foo
bar
endef
```
### 多行变量
还有一种设置变量值的方法是使用define关键字。使用define关键字设置变量的值可以有换行，这有利于定义一系列的命令（前面的“命令包”的技术就是利用这个关键字）。  
define指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以endef 关键字结束。其工作方式和“=”操作符一样。变量的值可以包含函数、命令、文字，或是其它变量。
### 环境变量
make运行时的系统环境变量可以在make开始运行时被载入到Makefile文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了“-e”参数，那么，系统环境变量将覆盖Makefile中定义的变量）  
因此，如果我们在环境变量中设置了 CFLAGS 环境变量，那么我们就可以在所有的Makefile中使用这个变量了。这对于我们使用统一的编译参数有比较大的好处。如果Makefile中定义了CFLAGS，那么则会使用Makefile中的这个变量，如果没有定义则使用系统环境变量的值，一个共性和个性的统一，很像“全局变量”和“局部变量”的特性。  
当make嵌套调用时（参见前面的“嵌套调用”章节），上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile 中。当然，默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层Makefile传递，则需要使用export关键字来声明。  
并不推荐把许多的变量都定义在系统环境中，这样，在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。
### 目标变量
可以为某个目标设置局部变量，这种变量被称为“Target-specific Variable”，它可以和“全局变量”同名，因为它的作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值。  
```
<target ...> : <variable-assignment>;
<target ...> : overide <variable-assignment>
```
<variable-assignment>;可以是前面讲过的各种赋值表达式，如 = 、 := 、 += `` 或是 ``?= 。第二个语法是针对于make命令行带入的变量，或是系统环境变量。  
```
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
    $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o : prog.c
    $(CC) $(CFLAGS) prog.c

foo.o : foo.c
    $(CC) $(CFLAGS) foo.c

bar.o : bar.c
    $(CC) $(CFLAGS) bar.c
```
在这个示例中，不管全局的 $(CFLAGS) 的值是什么，在prog目标，以及其所引发的所有规则中（prog.o foo.o bar.o的规则）， $(CFLAGS) 的值都是 -g
### 模式变量
给定一种“模式”，可以把变量定义在符合这种模式的所有目标上。  
可以以如下方式给所有以 .o 结尾的目标定义目标变量
```
%.o : CFLAGS = -O
```
他的语法同目标变量的相同：
```
<pattern ...>; : <variable-assignment>;

<pattern ...>; : override <variable-assignment>;
```
## 使用条件判断
使用条件判断，可以让make根据运行时的不同情况选择不同的执行分支。条件表达式可以是比较变量的值，或是比较变量和常量的值。
```
libs_for_gcc = -lgnu
normal_libs =

ifeq ($(CC),gcc)
    libs=$(libs_for_gcc)
else
    libs=$(normal_libs)
endif

foo: $(objects)
    $(CC) -o foo $(objects) $(libs)
```
使用示例如上。  
条件表达式的语法如下：
```
<conditional-directive>
<text-if-true>
endif
```
和
```
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```
<conditional-directive> 表示条件关键字,这个关键字有四个。分别为`ifeq`,`ifneq`,代表两个值相同与不同的判断。剩下两个为`ifdef`和`ifndef`，代表判断值是否为空。  
注意， ifdef 只是测试一个变量是否有值，其并不会把变量扩展到当前位置。  
```
bar =
foo = $(bar)
ifdef foo
    frobozz = yes
else
    frobozz = no
endif
```
此处$(frobozz) 值是 yes .
```
foo =
ifdef foo
    frobozz = yes
else
    frobozz = no
endif
```
此处的值则为no.所以他只会判断判断值有无定义，而不管最后是否有值。  
特别注意的是，make是在读取Makefile时就计算条件表达式的值，并根据条件表达式的值来选择语句，所以，你最好不要把自动化变量（如 $@ 等）放入条件表达式中，因为自动化变量是在运行时才有的。

而且为了避免混乱，make不允许把整个条件语句分成两部分放在不同的文件中。
## makefile函数
### 函数调用语法
```
$(<function> <arguments>)
#这种也可以
${<function> <arguments>}
```
<function> 就是函数名，make支持的函数不多。 <arguments> 为函数的参数，参数间以逗号 , 分隔，而函数名和参数之间以“空格”分隔。函数调用以 $ 开头，以圆括号或花括号把函数名和参数括起。
下方具体的函数及介绍可参考[此处](https://seisman.github.io/how-to-write-makefile/functions.html#id3)
### 字符串处理函数
### 文件名操作函数
### foreach函数
### if函数
### call函数
### origin函数
### shell函数
### 控制make的函数
## make的运行
### make的退出码
0表示成功执行。  
1如果make运行时出现任何错误，其返回1。  
2如果你使用了make的“-q”选项，并且make使得一些目标不需要更新，那么返回2。  
### 显式指定makefile
要显式指定makefile，可以使用-f 或是 --file 参数（ --makefile 参数也行）。例如，我们有个makefile的名字是“hchen.mk”，那么，我们可以这样来让make来执行这个文件：
```
make –f hchen.mk
```
### 指定目标
一般来说，make的最终目标是makefile中的第一个目标，而其它目标一般是由这个目标连带出来的。这是make的默认行为。任何在makefile中的目标都可以被指定成终极目标，只需在make命令后直接跟目标的名字就可以完成（如前面提到的“make clean”形式），但是除了以 - 打头，或是包含了 = 的目标，因为有这些字符的目标，会被解析成命令行参数或是变量。甚至没有被我们明确写出来的目标也可以成为make的终极目标，也就是说，只要make可以找到其隐含规则推导规则，那么这个隐含目标同样可以被指定成终极目标。  
有一个make的环境变量叫 MAKECMDGOALS ，这个变量中会存放你所指定的终极目标的列表，如果在命令行上，你没有指定目标，那么，这个变量是空值。这个变量可以让你使用在一些比较特殊的情形下。比如下面的例子：
```
sources = foo.c bar.c
ifneq ( $(MAKECMDGOALS),clean)
    include $(sources:.c=.d)
endif
```
基于上面的这个例子，只要我们输入的命令不是“make clean”，那么makefile会自动包含“foo.d”和“bar.d”这两个makefile。  
使用指定终极目标的方法可以很方便地让我们编译我们的程序，例如下面这个例子：
```
.PHONY: all
all: prog1 prog2 prog3 prog4
```
这个makefile中有四个需要编译的程序——“prog1”， “prog2”，“prog3”和 “prog4”，我们可以使用“make all”命令来编译所有的目标（如果把all置成第一个目标，那么只需执行“make”），我们也可以使用 “make prog2”来单独编译目标“prog2”。
### 约定的目标名称
all:这个伪目标是所有目标的目标，其功能一般是编译所有的目标。  
clean:这个伪目标功能是删除所有被make创建的文件。  
install:这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。  
print:这个伪目标的功能是例出改变过的源文件。  
tar:这个伪目标功能是把源程序打包备份。也就是一个tar文件。  
dist:这个伪目标功能是创建一个压缩文件，一般是把tar文件压成Z文件。或是gz文件。  
TAGS:这个伪目标功能是更新所有的目标，以备完整地重编译使用。  
check和test:这两个伪目标一般用来测试makefile的流程。  
### 检查规则
有时候，我们不想让我们的makefile中的规则执行起来，我们只想检查一下我们的命令，或是执行的序列。于是我们可以使用make命令的下述参数：  
-n 不执行，只打印  
-t 只把目标文件日期更新，但不更改目标文件。也就是说，make假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。  
-q 如果目标存在，那么其什么也不会输出，当然也不会执行编译，如果目标不存在，其会打印出一条出错信息。
-W 这个参数需要指定一个文件。一般是是源文件（或依赖文件），Make会根据规则推导来运行依赖于这个文件的命令，一般来说，可以和“-n”参数一同使用，来查看这个依赖文件所发生的规则命令。  
另外一个很有意思的用法是结合 -p 和 -v 来输出makefile被执行时的信息
### make参数
参见[make参数](https://seisman.github.io/how-to-write-makefile/invoke.html#id4)
这里就提几个有用的
-B 总是重编译所有目标  
-i 执行时忽略所有错误  
-k 出错也不停止运行  
-s 不输出命令的输出  
-t 把所有目标的修改日期变为最新，即阻止生成目标的命令运行。  
-w 输出运行makefile之前和之后的信息。这个参数对于跟踪嵌套式调用make时很有用。  
## 隐含规则
## 使用make更新库函数文件
这两个以后有需要再整吧。
