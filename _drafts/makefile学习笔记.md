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
## cmake
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
