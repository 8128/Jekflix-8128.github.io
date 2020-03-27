---
layout: post
title:  Bash Shell编程入门
date:   2020-03-19 12:00:00
description: 程序员必备基础知识，不会的不配找到工作
subtitle: 
comments: true
image: https://raw.githubusercontent.com/8128/PicGo/master/20200321004047.png
optimized_image: 
category: tutorial
tags:
  - tutorial
  - code
author: echoworlding
paginate: false
---

**作者：echoworlding**

**链接：https://www.jianshu.com/p/e1c8e5bfa45e**

**著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。**

## Shell是什么？

shell本身是一个命令解释器，介于操作系统的内核（kernel）态和用户态之间，可以执行系统调用及系统命令等，让用户以此来与操作系统实现互动。同时，它也用来指一种计算机程序语言（类似于C、Python等）。一个shell程序一般被称为一个脚本。

### Shell语言的流派

目前，shell主要有两大流派：

- sh:
  - burne shell (sh)
  - burne again shell (bash)
- csh:
  - c shell (csh)
  - tc shell (tcsh)
  - korn shell (ksh)

目前，大部分Linux系统预设的shell都是bash。

> Ubuntu16.04提供的shell环境（登录成功后默认使用bash）：

```bash
slot@slot-ubt:~$ cat /etc/shells 
# /etc/shells: valid login shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash
slot@slot-ubt:~$ 
```

> Mac OS提供的shell环境：

```bash
$ cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.
>
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh  # zsh系本文作者自己安装 
```

> 注意：
>
> 在Catalina之后mac就不是使用bash的了，而是使用zsh的

### 一个极简的bash demo: hello_world.sh

```bash
    #!/bin/bash
    
    # Here is comment 
    echo "Hello World!"
```

**执行**  

方法1: 直接使用bash解释器来解释执行：

```bash
    bash hello_world.sh
```

或者：

```bash
    sh hello_world.sh
```

方法2: 先将文件属性改为可执行状态：



```bash
    chmod +x hello_world.sh
```

或者：



```bash
    chmod 777 hello_world.sh  
```

再直接执行：



```bash
    ./hello_world.sh
```

**输出**



```bash
    Hello World!
```

**解释**



#!用来指定执行该脚本的解释器，后面的/bin/bash表明指定/bin目录下的bash程序来解释执行该脚本文件。

#开头的是注释行(#!除外)，shell中只有单行注释。  

```bash
    echo "Hello World!" 即用echo命令输出字符串"Hello World!"到终端显示器。
```

> **补充知识: 文件的属性**
>  通过`ls -l`命令可以查看文件的属性，例如查看新建文件test.sh的属性：

```ruby
slot@slot-ubt:~$ touch test.sh
slot@slot-ubt:~$ ls -l test.sh 
-rw-rw-r-- 1 slot slot 0 12月 21 15:40 test.sh
```

> 可以看到，一般新建文件的默认属性是`-rw-rw-r--`，即644，不具有可执行属性`x`，可使用`chmod`命令来改变文件属性（修改默认属性则使用`umask`命令），例如将文件test.sh的属性改为可读可写可执行(rwx: 4 + 2 + 1 = 7)：

```ruby
slot@slot-ubt:~$ chmod 777 test.sh 
slot@slot-ubt:~$ ls -l test.sh 
-rwxrwxrwx 1 slot slot 0 12月 21 15:40 test.sh
```

## Bash中的变量

### 变量的定义与赋值

不像C、Java等静态语言需要先声明然后才能使用，而是和Python等动态语言类似，Bash变量在使用时直接定义，例如：

```bash
my_bash_var="this is my bash var"
```

**注意**:

- **`＝`两边不能有空格！**否则就是语法错误了。
- Bash变量命名只能使用字母，下划线和数字，并且不能以数字开头。

### 变量的引用

使用已定义的变量时，只要在变量名前面加`$`符号即可:

```bash
echo $my_bash_var
```

或者使用`${var_name}`的形式，`{}`是可选的，主要是帮助解释器更好地识别变量的边界(推荐)：

```bash
echo ${my_bash_var}
```

注意`''`和`""`的区别：

- `''` ：单引号里的任何字符都会原样输出，单引号中对变量引用是无效的，且单引号中不能出现单引号（对单引号使用转义符也不行）；
- `""`：双引号里可以引用变量，可以出现转义字符。

实例：

```bash
a="hello"
echo 'a is : $a'
echo "a is : $a"
```

Output:

```csharp
a is : $a
a is : hello
```

#### 只读变量

使用 **`readonly`** 命令可以将变量限定为只读变量，这与 C 语言中的 const 常量类型的情况相同.

```bash
a_var="hello"
readonly a_var
a_var="world"  # Output: bash: read-only variable: a_var
```

#### 删除变量

使用 **`unset`** 命令可以删除变量，但是不能删除只读变量。
 变量被删除后不能再次使用。

```bash
my_var="haha"
unset my_var
echo ${my_var} # 变量my_var已被删除，没有任何输出
```

### 变量的类型

诸如C、Java、Python等这些高级语言中的变量是有类型的，例如数字类型（整型、浮点型等）、字符串类型、布尔类型，面向对象语言中还有引用类型等。但是，在Bash中，并不对变量区分**类型**。

本质上来说，Bash变量都是字符串。但是依赖于上下文，Bash也允许比较操作和算术操作。决定这些的关键因素是**变量中的值是否只有数字**，只有当变量是纯数字时，该变量才是“数字类型的”，否则就是字符串类型的。

另外，注意Bash中的数字默认的是十进制，八进制需要以`0`开头，十六进制以`0x`开头。

纯数字变量是“数字变量”：

```bash
a=1234
let "a+=1"
echo ${a}  # Output: 1235
```

数字＋字符串：字符串变量，字符串变量不能进行数学运算

```bash
b=${a/12/BB} # 将12替换为BB
echo ${b}    # Output: BB35

let "b+=1"
echo ${b}    # Output:1
```

将变量中的非数字字符替换为数字，得到数字变量

```bash
c=BB34
echo ${c}     # Output: BB34

d=${c/BB/12}  # 将BB替换为12
echo ${d}     # Output: 1234

let "d+=1"
echo ${d}     # Output: 1235
```

空变量+数字：数字变量

```bash
# 变量e定义为空值
e=""
echo ${e}  # Output: 没有任何输出

let "e+=1" # 空值 + 1
echo ${e}  # Output: 1
```

未定义的变量+数字：数字变量

```bash
# 变量f未定义
echo "f = $f"   # Output: f =

let "f+=1" 
echo "f = ${f}" # Output: f = 1
```

### 变量的作用域

- 局部变量(local variables)：这种变量只有在变量所在的代码块或者函数中才可见，需要使用`local`声明；
- 全局变量：Bash中用户自定义的普通变量默认是全局变量，可以在本文件中的其它位置引用；
- 环境变量(environmental variables)：所有的程序（包括shell启动的程序）都能访问环境变量。
   如果一个shell脚本设置了环境变量,需要用 export 命令来通知脚本的环境。

> 更多关于环境变量的知识可以参考以下文章：
>
> - [Linux环境变量总结](https://www.jianshu.com/p/ac2bc0ad3d74)
> - [linux入门之环境变量与文件查找](https://www.jianshu.com/p/f7d4a821d292)

## Bash中的运算符

Bash支持的运算符有：

- 数学运算符
- 关系运算符
- 布尔运算符
- 逻辑运算符
- 字符串运算符
- 文件测试运算符

### 数学运算符

| 数学运算符 | 说明                         |
| ---------- | ---------------------------- |
| +          | 加法                         |
| -          | 减法                         |
| *          | 乘法                         |
| /          | 除法                         |
| %          | 取余                         |
| =          | 赋值                         |
| ==         | 相等测试，相等则返回true     |
| !=         | 不相等测试，不相等则返回true |

**注意：**  乘号`*`前边必须加反斜杠 \ 才能实现乘法运算

### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字

| 关系运算符 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| -eq        | (equal)    检测两个数是否相等，相等则返回 true               |
| -ne        | (not equal）检测两个数是否相等，不相等则返回 true            |
| -gt        | (greater than）检测左边的数是否大于右边的，如果是，则返回 true |
| -lt        | (lower than) 检测左边的数是否小于右边的，如果是，则返回 true |
| -ge        | (greater equal）检测左边的数是否大于等于右边的，如果是，则返回 true |
| -le        | (lower equal) 检测左边的数是否小于等于右边的，如果是，则返回 true |

### 布尔运算符

| 布尔运算符 | 说明                                              |
| ---------- | ------------------------------------------------- |
| -a         | 与运算，两个表达式都为 true 才返回 true           |
| -o         | 或运算，有一个表达式为 true 则返回 true           |
| !          | 非运算，表达式为 true 则返回 false，否则返回 true |

### 逻辑运算符

| 逻辑运算符 | 说明   |
| ---------- | ------ |
| &&         | 逻辑与 |
| II         | 逻辑或 |

### 字符串运算符

| 字符串运算符 | 说明                                    | 举例  (a="abc" b="def") |
| ------------ | --------------------------------------- | ----------------------- |
| =            | 检测两个字符串是否相等，相等返回 true   | [ $a = $b ] 返回 false  |
| !=           | 检测两个字符串是否相等，不相等返回 true | [ $a != $b ] 返回 true  |
| -z           | 检测字符串长度是否为0，为0返回 true     | [ -z $a ] 返回 false    |
| -n           | 检测字符串长度是否为0，不为0返回 true   | [ -n $a ] 返回 true     |
| str_name     | 检测字符串是否为空，不为空返回 true     | [ $a ] 返回 true        |

### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性

| 文件测试运算符 | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| -b file        | 检测文件是否是块设备文件，如果是，则返回 true                |
| -c file        | 检测文件是否是字符设备文件，如果是，则返回 true              |
| -d file        | 检测文件是否是目录，如果是，则返回 true                      |
| -f file        | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true |
| -g file        | 检测文件是否设置了 SGID 位，如果是，则返回 true              |
| -k file        | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true    |
| -p fill        | 检测文件是否是命名管道，如果是，则返回 true                  |
| -u file        | 检测文件是否设置了 SUID 位，如果是，则返回 true              |
| -r file        | 检测文件是否可读，如果是，则返回 true                        |
| -w file        | 检测文件是否可写，如果是，则返回 true                        |
| -x file        | 检测文件是否可执行，如果是，则返回 true                      |
| -s file        | 检测文件是否为空（文件大小是否大于0），不为空返回 true       |
| -e file        | 检测文件（包括目录）是否存在，如果是，则返回 true            |

## Bash的控制流

### 条件语句

#### if语句

- if

```bash
if [ condition ]
then
    command
fi
```

- if-else

```bash
if [ condition ]
then
    command
else
    command
fi
```

- if-elif-else

```bash
if [ condition1 ]
then
    command1
elif [ condition2 ]
    command2
else
    commandN
fi
```

**注意：**
 勿忘最后的`fi`（`if`的反向拼写）!

#### case语句

```bash
case "${var}" in
    "$condition1" )
        command1
    ;;

    "$condition2" )
        command2
    ;;

    * )  # 这里相当于C中case语句的default 
esac
```

**注意：**

- 对变量使用`""`并不是强制的，因为不会发生单词分离;
- 每句测试行,都以右小括号`)`结尾;
- 每个条件块都以两个分号`;;`结尾（作用类似C中的break）;
- case 块的结束以 `esac`(case 的反向拼写)结尾.

### 循环语句

#### for语句

```bash
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

**注意：**  
 在循环的每次执行中，var将顺序的存取 list （i.e. item1 ... itemN）中列出的变量。

C风格的for循环:

```bash
for (( EXP1; EXP2; EXP3 ))
do
    command1
    command2
    command3
done
```

**注意:**  `(())`中对变量的引用可以不加`$`

#### while语句

```bash
while [ condition ]
do
    command
done
```

C风格的while循环:

```bash
while (( condition ))
do
    command
done
```

**注意:**  `(())`中对变量的引用同样可以不加`$`

#### until语句

```bash
until [ condition-is-true ]
do
    command
done
```

**注意：**  
 util结构在循环的顶部判断条件,并且如果条件一直为 false 那就一直循环下去，直到条件为真才结束循环(与 while 相反)。

## Bash支持的编程模型

Bash只支持**过程式**的编程模型，不支持面向对象和函数式等高级编程模型，更不支持对高级设计模式的实现。所以，如果要实现比较复杂的功能，还是使用Perl、Python或者Ruby等高级语言吧。

## Bash的传参机制

shell通过**位置参数**（positional parameters）来给脚本文件传递参数，就是从命令行中传进来的参数,$0, $1, $2, $3...  其中：

- $0 是该脚本文件的名字
- $1 是第一个参数, $2 是第 2 个参数...

**注意：**

- $9 以后就需要大括号了,如 ${10}, ${11}, ${12}...

另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。                     |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $-       | 显示Shell使用的当前选项，与set命令功能相同                   |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

`$*` 与 `$@` 区别:

- 相同点：都表示引用所有的位置参数;
- 不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

## Bash函数

### 函数的定义

```php
function func_name()
{
    # 函数体
}
```

**注意：**

- 关键字function是可选的;
- 函数定义必须在第一次调用前完成，shell没有像 C 中的函数“声明”;
- 在一个函数内嵌套另一个函数也是可以的，但是不常用.

### 函数的参数传递

函数以**位置**来引用传递过来的参数(就好像他们是位置参数一样), 例如$1, $2, ...

### 函数的调用

函数被调用或被触发, 只需要简单地用函数名来调用，有参数的话将参数依次置于函数名之后。

#### 无参函数的调用



```php
function func1()
{
    echo "Hello World!"
    echo "This is func1"
}

# Calling func1
func1
```

输出：



```swift
Hello World!
This is func1
```

#### 有参函数的调用



```bash
function func2()
{
    echo "This is func2"
    
    a=$1
    b=$2
    echo "a is : $a"
    echo "b is : $b"
}

# Calling func2,and pass two parameters
func2 "aaa" "bbb"
```

输出：



```csharp
This is func2
a is : aaa
b is : bbb
```

### 函数中参数的作用域

在函数调用之前，所有在函数内声明且没有明确声明为 local 的变量都可在函数体外可见（默认为全局变量）。

如果变量用`local` 来声明,那么它只能在该变量声明的代码块中可见。这个代码块就是局部"范围"。在一个函数内,局部变量意味着只能在函数代码块内它才有意义。



```bash
  1 #!/bin/bash
  2
  3 func()
  4 {
  5     global_var="I am global_var difined in function func."
  6     echo $global_var
  7
  8     local loc_var="I am local_var defined in function func"
  9     echo $loc_var
 10 }
 11
 12 func
 13 echo
 14 echo $global_var
 15 echo $loc_var
 16
 17 exit
```

输出：



```go
I am global_var difined in function func.
I am local_var defined in function func

I am global_var difined in function func.
```

## Bash数组

Bash 只支持一维数组，用圆括号`()`来表示，数组元素之间用"空格"符号来分割（不同于C、C++、Java等语言中用逗号分割）。

### 数组的定义与初始化

初始化时不需要指定数组的大小。和其它大部分语言一样，bash数组元素的下标从`0`开始。

### 初始化方式1：直接初始化



```undefined
arr_name=(value1 value2 ... valueN) 
```

### 初始化方式2: 用下标初始化



```bash
arr_name[0]=value1
arr_name[1]=value2
arr_name[223]=value3 # 不连续初始化
```

**注意：**

- 数组成员不必一定要连续，空缺元素是允许的;
- 数组的一部分成员允许不被初始化, 没有被初始化的元素将打印空(NULL)值;

### 访问数组元素

访问数组元素的一般格式：



```bash
${arr_name[index]}
```

### 遍历数组

使用`*` 或`@` 可以获取数组中的所有元素
 实例：



```bash
my_arr=(A B C "ddd")

# 遍历数组格式1
echo "my_arr: ${my_arr[*]}

# 遍历数组格式2
echo "my_arr: ${my_arr[@]}
```

输出：



```undefined
my_arr: A B C ddd
my_arr: A B C ddd
```

### 获取数组长度

获取数组长度（即数组中的元素个数），和遍历数组语法很相似，只是在数组名前加了`#`符号，格式：



```ruby
${#arr_name[*]}
```

或者



```ruby
${#my_arr[@]}
```

实例：



```bash
# 获取数组my_arr的长度（元素个数）
echo "my_arr length: ${#my_arr[*]}

# 获取数组my_arr的长度,
echo "my_arr length: ${#my_arr[@]}
```

输出：



```undefined
my_arr length: 4
my_arr length: 4
```

## Bash字符串操作

在Bash中，字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别如前文所述。
 定义字符串实例：



```bash
str1=hello
str2='hello'
str3="hello"

echo $str1
echo $str2
echo $str3
```

输出：



```undefined
hello
hello
hello
```

### 获取字符串长度

格式：



```ruby
${#string_name}
```

实例：



```bash
my_str="hello world"
echo "my_str length: ${#my_str}
```

输出：



```undefined
my_str length: 11
```

### 字符串拼接

和Java Python等语言类似，bash允许直接将字符串拼接在一起以获得新的字符串。
 实例：



```bash
str1="hello"
str2="world"

str3=${str1}" "${str2}
echo ${str3}
```

输出：



```undefined
hello world
```

### 提取子串

| 表达式                    | 含义                                                 |
| ------------------------- | ---------------------------------------------------- |
| ${string:position}        | 在string中, 从位置position开始提取子串               |
| ${string:position:length} | 在string中, 从位置position开始提取长度为length的子串 |

实例1：从位置1开始提取子串



```bash
str="hello world"

sub_str=${str:1}
echo "sub_str: ${sub_str}"
```

输出：



```undefined
sub_str: ello world
```

实例2：从位置1开始提取长度为3的子串



```bash
str="hello world"

sub_str=${str:1:3}
echo "sub_str: ${sub_str}"
```

输出：



```undefined
sub_str: ell
```

### 删除子串

| 表达式               | 含义                                        |
| -------------------- | ------------------------------------------- |
| ${string#substring}  | 从string的开头, 删除最短匹配substring的子串 |
| ${string##substring} | 从string的开头, 删除最长匹配substring的子串 |
| ${string%substring}  | 从string的结尾, 删除最短匹配substring的子串 |
| ${string%%substring} | 从string的结尾, 删除最长匹配substring的子串 |

记忆：

- `#`表示从头匹配，`%`表示从尾匹配
- 一个符号（`#`或者`%`）表示最短匹配，两个符号（`##`或者`%%`）表示最长匹配

**注意：**substring可以是正则表达式。

实例：



```bash
str="abcabcdefabcabc"

# 从str的开头,删除最短匹配的以a开头c结尾的子串
# 将删除最左端的abc
# 输出：abcdefabcabc
echo "${str#a*c}"  

# 从str的开头,删除最长匹配以a开头b结尾的子串
# 将删除abcabcdefabcab
# 输出：c
echo "${str##a*b}" 

# 从str的结尾,删除最短匹配以a开头c结尾的子串
# 将删除最右端的abc
# 输出：abcabcdefabc
echo "${str%a*c}"  

# 从str的结尾,删除最长匹配以a开头c结尾的子串
# 将删除整个字符串
# 输出：空
echo "${str%%a*b}" 
```

### 替换子串

| 表达式                           | 含义                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| ${string/substring/replacement}  | 使用$replacement 来代替**第一个**匹配的$substring            |
| ${string//substring/replacement} | 使用$replacement 代替**所有**匹配的$substring                |
| ${string/#substring/replacement} | 如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |
| ${string/%substring/replacement} | 如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |

实例：



```ruby
str="abcdefabc"

# 用hello替换第一个abc
echo ${str/"abc"/"hello"}    # 输出：hellodefabc

# 用hello替换第一个abc
echo ${str//"abc"/"hello"}   # 输出：hellodefhello

# 前缀匹配替换
echo ${str/#"abc"/"world"}   # 输出：worlddefabc

# 后缀匹配替换
echo ${str/％"abc"/"world"}   # 输出：abcdefworld
```

## References

[Advanced Bash-Scripting Guide](https://link.jianshu.com?t=http://tldp.org/LDP/abs/html/)
 这本书内容翔实，实例丰富，既可做精钻细研的教材，又可作为手册时时查阅，真是居家旅行必备良品;-)

另外，感兴趣的同学也可翻阅《Linux Shell Scripting Cookbook》(中文译名《Linux Shell脚本攻略》)，里面介绍了很多Shell编程的奇技淫巧，甚是有用有趣。