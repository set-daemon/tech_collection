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

一个整型literal是一个用数字序列表示的整型常量。一个可选前缀设置非十进制基底：0是八进制，0x或0X是十六进制。在十六进制literal里，字母a-f以及A-F表示10到15。

​	int_lit = decimal_lit | octal_lit | hex_lit .

​	decimal_lit = ("1" ... "9") {decimal_digit} .

​	octal_lit = "0" {octal_digit} .

​	hex_lit = "0" ("x" | "X") hex_digit {hex_digit} .

例如：

​	42

​	0600

​	0xBadFace

​	170141183460469231731687303715884105727

#### 浮点literals

浮点literal是浮点常数的十进制表示。它有整数部分，一个十进制点，一个分数部分，以及指数部分。整数和分数部分组成十进制数字；指数部分是一个e或E，其后有一个可选的符号十进制指数。整数部分或分数部分中的一个可以被省略；十进制点或指数中的一个也可以被省略。

float_lit = decimals "." [decimals]\[exponent] | 

​		decimals exponent |

​		"." decimals [exponent] .

decimals = decimal_digit {decimal_digit} .

exponent = ("e" | "E") ["+" | "-"] decimals .

例如：

​	0.

​	72.40

​	072.40 // == 72.40

​	2.71828

​	1.e+0

​	6.67428e-11

​	1E6

​	.25

​	.12345E+5

#### 虚数literal

虚数是复数的虚数部分的十进制表示。它由一个带小写字母i结尾的浮点literal或十进制整数组成。

​	imaginary_lit = (decimals | float_lit) "i"

例子：

​	0i

​	011i // == 11i

​	0.i

​	2.71828i

​	1.e+0i

​	6.67428e-11i

​	1E6i

​	.25i

​	.12345E+5i

#### Rune字面量

一个rune字面量代表一个rune常数，一个整数值标识一个Unicode代码点。一个rune字面量被表达成一个或多个字符被包括在单引号里，例如‘x'或'\n'。除换行和未转义的单引号外，任何其它字符都可出现在引号中。一个单引号字符表示该字符本身的Unicode值，即以反斜杠开始的多种格式的多字符序列编码值。

最简单的形式表示单引号中的单个字符（？）；由于Go源文件是由UTF-8编码的Unicode字符，多个UTF-8编码字节也许表示一个单整数值。例如，字面量'a'占一个单字节，但表示一个字面量a，Unicode值为U+0061，值为0x61，而'ä'占2个字节（0xc3 0xa4）表示一个字面量a-dieresis，U+00E4，值为0xe4。

许多反斜杠转义允许任何值被编码成ASCII文本。这里有4种方式表示整数值为数值常量：\x后面接两个完整十六进制数字；\u后面接4个完整十六进制数字；\U后面接8个完整的十六进制数字；以及一个反斜杠后接3个完整的八进制数字；每种情况下这个字面量的值是由对应基数的数字所表示的值。

在一个反斜杠后，某些单字符转义表示特殊值：

​	\a U+0007 alert或bell

​	\b U+0008 退格

​	\f U+000C form feed

​	\n U+000A 换行

​	\r U+000D 回车

​	\t U+0009 水平制表符

​	\v U+000b 垂直制表符

​	\\\\ U+005c 反斜杠

​	\\' U+0027 单引号 （仅在rune字面量中转义有效）

​	\\" U+0022 双引号（仅在字符串字面量中转义有效）

所有其它的以反斜杠开始的序列在rune字面量中都是非法的。

​	rune_lit = "''" (unicode_value | byte_value) "''" .

​	unicode_value = unicode_char | little_u__value | big_u_\_value | escaped_char .

​	byte_value = octal_byte_value | hex_byte_value .

​	octal_byte_value = \`\\\` octal_digit octal_digit octal_digit .

​	hex_byte_value = \`\\\` ”x" hex_digit hex_digit .

​	little_u_value = \`\\\` "u" hex_digit hex_digit hex_digit hex_digit .

​	big_u_value = \`\\\` "U" hex_digit hex_digit hex_digit hex_digit hex_digit hex_digit hex_digit hex_digit .

​	escaped_char = \`\\\` ("a" | "b" | "f" | "n" | "r" | "t" | "v" | \`\\\` | "''" | \`"\`) .

例子：

​	'a'

​	'ä'

​	'本'

​	'\t'

​	'\000'

​	'\007'

​	'\377'

​	'\x07'

​	'\xff'

​	'\u12e4'

​	'\U00101234'

​	'\''         // rune literal containing single quote character

​	'aa'         // illegal: too many characters

​	'\xa'        // illegal: too few hexadecimal digits

​	'\0'         // illegal: too few octal digits'\uDFFF'     // illegal: surrogate half

​	'\U00110000' // illegal: invalid Unicode code point

#### 字符串字面量

一个字符串字面量表示从字符序列拼接而来的字符串常量。这里有两种形式：原始字符串字面量和解释的字符串字面量。

原始字符串字面量是在反引号之间的字符序列，例如\`foo\`。除反引号外，其它任何字符都可以出现在引号中。一个原始字符串字面量的值是引号中未解释的（隐含UTF-8编码）字符组成；特别地，反斜杠没有特殊意义，字符串中也许包含换行；在原始字符串字面量中的回车符('\r')将会从原始字符串值被抛弃。

解释的字符串字面量是在双引号之间的字符序列，例如"bar"。除换行和未转义的双引号外，其它任何字符都可以出现在引号中。引号中的文本组成了字面量的值，同样的限制下，反斜杠转义被解释成在rune字面量的值（例外情况，\\’是非法的，而\\"是合法的）。三个数字的八进制(\\nnn)和两个数字的十六进制(\\xnn)转义表示结果字符串的单个字节，所有其它转义代表（可能多字节）UTF-8编码的单个字符。因此在一个字符串字面量中\377和\XFF代表一个单字节值为 0XFF=255，而ÿ、\u00FF、\U000000FF和\xc3\xbf代表了UTF-8编码的字符U+00FF的两个字节0xc3 0xbf。

​	string_lit = raw_string_lit | interpreted_string_lit .

​	raw_string_lit = "\`" {unicode_char | newline } "\`" .

​	interpreted_string_lit = \`"\` {unicode_value | byte_value} \`"\` .

例子：

​	\`abc\`  // 等同于"abc"

​	\`\\n

​	  \\n\` // 等同于"\\\\n\\n\\n"

​	"\n"

​	"\""   // 等同于\`"\`

​	"Hello,world!\\n"

​	"中国語"

​	"\u65e5本\U00008a9e"	

​	"\xff\u00FF"

​	"\uD800"  //非法：仅表示了一半

​	"\U00110000" //非法：无效的Unicode代码点

下面的例子都代表同样的字符串：

​	"日本語"                                 // UTF-8 input text

​	\`日本語\`                                 // UTF-8 input text as a raw 

​         "\u65e5\u672c\u8a9e"          // the explicit Unicode code

​         "\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code                      

​          "\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes

如果源代码用两个代码点表示一个字符，比如一个声调和字母的合并形式，如果放在一个rune字面量中会产生错误（这不是一个单代码点），而放在一个字符串字面量则会显示成两个代码点。

#### 常量

Go语言有布尔常量、rune常量、整数常量、浮点常量、复数常量以及字符串常量。Rune、整数、浮点数以及复数常量统一称为数值常量。

常量值可以用以下数据来表示：rune、整数、浮点数、虚数或字符串字面量、指示常量的标识符、常量表达式、常量结果的转换、一些类似应用于任何值的unsafe.Sizeof的内置函数的结果值、应用于一些表达式的cap或len、应用于复数常量的real和imag，以及应用于数值常量的complex。布尔真值由预声明常量true和false来表示。预声明标识iota指示一个整数常量。

通常，复数常量是常量表达式的一个形式，将会在那一节中讨论。

数值常量表示任意精度的确切值，且不溢出。因此，这儿没有常量能指示IEEE-754的负0、无穷值以及非数值值。

常量可以是有类型或无类型的。字面常量、true、false、iota以及某些常量表达式，仅包含无类型常量操作数时是无类型的。（？）

一个常量可以被用常量声明或转换来明确给定类型，或者当在一个变量声明或赋值或作为表达式的一个操作数中使用时被隐性赋予类型。如果常量值不能表示成特定类型的值，将会产生错误。例如，3.0可以赋值给任何整数或任意浮点类型，而2147483648.0(等于1<<31)则只能赋给类型float32、float64、或uint32，但不能给int32或string。

一个无类型常量有一个默认类型，该类型取决于上下文有什么类型需求，并实现隐性转换，例如，一个短变量声明i := 0，这里没有明确的类型，对于这个无类型的常量，默认的类型可以分别是bool、rune、int、float64、complex128或string，取决于i是一个boolean、rune、integer、floating-point、complex或string常量。

实施限制：尽管在该语言中数值常量有任意精度，编译器将会内部表示的限制精度来实现。这就是说，每个实施必须：

​	用至少256bits来表示一个整数常量。

​	对包含复数常量部分的浮点常量，至少需要256bits来表示其尾部，至少16bits表示带符号指数。

​	如果不能精确地表示一个整数常量，报错。

​	如果因为溢出不能表示一个浮点或复数常量，报错。

​	如果因为精度限制不能表示一个浮点或复数常量，将其舍入到最接近表达的常量。

这些要求适用于字面常量和常量表达式的计算值。

#### 变量

变量是存放值的一个存储位置。可允许的值集合由变量类型决定。

一个变量声明，或对于函数参数和结果，一个函数声明签名或函数字面为一个有名变量保留存储空间（？）。调用内置函数new或获取一个复合字面量（composite literal?）的地址可以在运行时为变量分配存储空间。那种匿名变量是通过一个指针间接量所引用（可能是隐性的）。

array、slice和struct类型的结构化变量具有可能可以单独寻址的元素和域，其中每个元素犹如一个变量。（那么域呢？）

一个变量的静态类型（或者仅类型）是在声明时给出的类型，或由new调用或复合字面量提供的类型，或一个结构体变量的一个元素的类型。接口类型的变量也有一个不同的动态类型，这是在运行时可赋给变量的值的具体类型（除非这个值是预定义为无类型的标识符nil）。动态类型在执行时可能会发生改变，但存储在接口变量中的值始终可以赋给变量的静态类型。

var x interface {} // x为nil，有一个静态类型interface{}

var v \*T   // v值为nil，静态类型\*T

x = 42    // x值为42,动态类型int

x = v       // x值为(\*T)(nil)和动态类型\*T

变量的值通过引用在表达式中的这个变量获取到的；也是最近赋给该变量的值。如果一个变量还没有赋值，它的值默认为该值类型的0值。

## 类型

一个类型决定了值的集合以及伴随这些值的具体操作和方法。类型可以是由有名称的类型表示，也可以用从已有类型组合成的类型字面来表示。

​	Type = TypeName | TypeLit | "(" Type ")" .

​	TypeName = identifier | QualifiedIdent .

​	TypeLit = ArrayType | StructType | FunctionType | InterfaceType | SliceType |MapType |ChannelType

布尔、数值和字符串类型的有名实例是预声明的（编译系统内置的概念？）。其它的有名类型是由类型声明表示的（有名类型是具体声明的一个引用）。组合类型，如数组、结构体、指针、函数、接口、slice、map和channel类型，是使用类型字面来构成的（基础类型和其它组合类型再组合）。

每个类型T有一个底层类型：如果T是预声明的布尔、数值或字符串类型，或一个类型字面，对应的底层类型是T本身，否则，T的底层类型，是T在引用它的类型声明时的类型的底层类型。

​	type (

​		A1 = string

​		A2 = A1

​	)

​	type (

​		B1 string

​		B2 B1

​		B3 []B1

​		B4 B3

​	)

字符串、A1、A2、B1和B2的底层类型是字符串。而[]B1、B3和B4的底层类型是[]B1。

#### 方法集合

一个类型有一个与它关联的方法集。一个接口类型的方法集是它的接口。任何其它类型T的方法集是由与接收类型T声明的所有方法组成。对应的指针类型\*T的方法集是接收者\*T或T所声明的方法集（即，也包括T的方法集）。应用于包含嵌入域的结构体的规则，可以在***结构体类型***小节中找到描述。任何其它类型的方法集合是空的。在方法集合中，每个方法必须有唯一且非空方法名。

#### 布尔类型

布尔类型代表了一个由预声明常量true和false表示的布尔真值的集合。预声明布尔类型是bool。

#### 数值类型

一个数值类型代表了整数或浮点值的集合。预定义的架构无关的数值类型是：

​	uint8   所有无符号8bit整数的集合（0到255)

​	uint16 所有无符号16bit整数的集合（0到65535）

​	uint32 所有无符号32bit整数的集合（0到4294967295）

​	uint64 所有无符号64bit整数的集合（0到18446744073709551615）



​	int8 所有有符号8bit整数的集合（-128到127）

​	int16 所有有符号16bit整数的集合（-32768到32767）

​	int32 所有有符号32bit整数的集合（-2147483648到2147483647）

​	int64 所有有符号64bit整数的集合（-9223372036854775808到9223372036854775807）



​	float32 所有IEEE-754 32bit浮点数集合

​	float64 所有IEEE-754 64bit浮点数集合



​	complex64 所有复数集合，实数和虚数部分都为float32

​	complex128 所有复数集合，实数和虚数部分都为float64	



​	byte uint8的别名

​	rune int32的别名



一个n-bit的整数值是n bit宽，并用二进制补码算术表示。

还有一个预声明的特定大小的数值类型集合：

​	uint 32或64位

​	int 与uint相同大小

​	uintptr 一个无符号整数足够存储一个未解释的指针值的位数

为避免兼容问题，除了uint8的别名byte和int32的别名rune外，其余所有的数值类型都是不同的。当不同的数值类型混在一个表达式或赋值时，转换是必然的。例如，int32和int是不同的类型，尽管它们在某些特定的架构上有相同大小。

#### 字符串类型

string类型表示字符串值的集合。一个字符串值是一个字节（可能为空）的序列。字符串是不可修改的(immutable)：一旦创建，就不可能改变一个string的内容。这个预声明的字符串类型是string。

一个字符串s（大小用byte表示）的长度能用内置函数len求得。如果字符串是一个常量，那么其长度是编译时的常量。一个字符串的字节可以从整数索引0到len(s)-1访问，获取这种元素的地址是非法的，如s[i]是一个字符串的第i个byte，&s[i]是无效的。















































































































































































































































































































































































































































































































































































































































































































































## 参考

https://golang.org/ref/spec

https://translate.google.cn





















