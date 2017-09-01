# The Go Programming Language Specification

***Version Of June 28, 2017, 由set_daemon@126.com翻译，2017-09-01，部分语句使用谷歌翻译，用于学习***



##介绍

这是Go编程语言的参考手册，要获取更多信息和其它文档，请访问golang.org。

Go被设计成用于系统编程的多用途语言。它是强类型的，具备垃圾回收机制，明确支持并发编程。程序是由包构造而成的，这个特点获得了高效的依赖管理。当前的实现使用传统的编译/链接模型来产生可执行二进制。

Go的语法是紧凑而规则的，这使得一些类似集成开发环境的自动化工具更容易分析。

##Notation

句法规则由Extended Backus-Naur Form(EBNF)来表示：

​	Production = production_name "=" [ Expression ] "." .

​	Expression = Alternative { "|" Alternative } .

​	Alternative = Term { Term }

​	Term = production_name | token [ "···" token ] | Group | Option | Repetition .

​	Group = "(" Expression ")"

​	Option = "[" Expression "]"

​	Repetition = "{" Expression "}"

Production是由term和以下操作符构造而成的表达式，其优先级递增：

​	| alternation

​	() grouping

​	[] option (0或1次)

​	{} repetition (0到n次)

小写的production名字用于标识词法记号，非终止符(Non-terminals)用驼峰写法(CamelCase)。词法记号包括在双引号""或反引号``中。

a ··· b的形式表示从a到b的字符集是可选的，<u>水平省略号···在这份手册的其它地方非正式地表示各种未详细指定的枚举或代码片段</u>。这个字符···（与这三个字符...相反）不是Go语言的记号。

## 源代码表示

源代码是UTF-8编码的Unicode文本。<u>文本不是规范化的，所以单个重音代码点与组合口音和字母的同一个字符不同，这会被处理成两个代码点。为简化起见，本文档将用不合格的术语character指向源文本中的Unicode代码点</u>。（？）

每个代码点是不同的，例如，大写和小写字母是不同的字符。

实施限制：为了兼容其它工具，编译器也许会不允许在源文本中出现NUL（U+0000）字符。

实施限制：为兼容其它工具，编译器也许会忽略UTF-8编码字节序记号（U+FEFF），如果它是源文件中的第一个Unicode代码点。字节序标识不允许出现在源文本中的任何地方。

### 字符

以下术语用于代表特定的Unicode字符类：

​	newline = /\* Unicode代码点 U+000A \*/ .

​	unicode_char = /\* 除newline以外的任意Unicode代码点 \*/

​	unicode_letter = /\* 分类成"字母"的Unicode代码点 \*/

​	unicode_digit = /\* 分类成“数，十进制数字“的Unicode代码点 \*/

在Unicode标准8.0，第4.5节”通用类别“定义了一套字符类别。Go将任意字符类别Lu、Ll、Lt、Lm或Lo处理成Unicode字符，数类别Nd处理成Unicode数字。

#### 字符和数字

下划线字符_（U+005F）作为一个字母。

​	letter                = unicode_letter | "_" .

​	decimal_digit  = ”0“ ··· ”9“ .

​	octal_digit       = "0" ··· "7" .

​	hex_digit         = "0" ··· ”9“ | ”A" ··· “F" | "a" ··· ”f"

## 词法元素(Lexical elements)

#### 注释

注释作为程序文档，有两种形式：

​	1 行注释以字符序列//开始并在本行尾停止

​	2 通常的注释以字符序列/\*开始，出现第一个字符序列\*/结束

注释不能出现在一个rune或string literal中，或在一个注释中。不包含新行的一般注释就像一个空格。任何其它的注释就像新行。

#### 记号tokens

记号组成了Go语言的词汇表。有四类记号：标识符、关键字、操作符和标点符号、literals。空白符由空格（U+0020）组成，水平制表符tab（U+0009），回车（U+000D），以及换行（U+000A），这些字符在用于分隔记号防止粘合成单一记号时被保留，其它情况被忽略。换行或文件尾也许会触发分号的插入。当将输入拆成记号时，下一个记号是由最长的字符序列组成的有效记号。

#### 分号

之前的语法使用分号“;"作为一些production的终止符。Go程序会在以下两种规则下忽略大部分分号：

​	1 当输入被拆成记号，一个分号将在记号流中的一行最后一个记号后自动插入，如果最后的记号是

​		一个标识符

​		一个整型、浮点数、虚数、rune或字符串literal

​		以下关键字：break、continue、fallthrough或return

​		以下操作符或标点符：++、--、)、]或}

​	2 为允许复杂语句占据一个单行，在")"或"}"之前的分号会被省略。

为了反映习惯用法，本文档中的代码样例将使用这些规则来删除分号。

#### 标识符

标志符命名了程序实体，例如变量和类型。一个标识符是一个或多个字符和数字的序列。标识符的第一个字符必须是字母。

​	identifier = letter { letter | unicode_digit } .



​	a

​	_x9

​	ThisVariableIsExported

​	α β

一些标识符是预声明的。

#### 关键字

以下关键字是保留的，不能被用作标识符。

| break    | default | func   | interface | select | case  | defer       | go   | map   | struct |
| -------- | ------- | ------ | --------- | ------ | ----- | ----------- | ---- | ----- | ------ |
| chan     | else    | goto   | package   | switch | const | fallthrough | if   | range | type   |
| continue | for     | import | return    | var    |       |             |      |       |        |



#### 操作符和标点

以下的字符序列表示操作符（包括赋值操作符）以及标点：

| +    | &    | +=   | &=   | &&   | ==   | !=   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| (    | )    | -    | \|   | -=   | \|=  | \|\| |
| <    | <=   | [    | ]    | *    | ^    | *=   |
| ^=   | <-   | >    | >=   | {    | }    | /    |
| <<   | /=   | <<=  | ++   | =    | :=   | ,    |
| ;    | %    | >>   | %=   | >>=  | --   | !    |
| ...  | .    | :    | &^   | &^=  |      |      |



#### 整型literals

一个整型literal是一个用数字序列表示的整型常量。









## 参考

https://golang.org/ref/spec

https://translate.google.cn





















