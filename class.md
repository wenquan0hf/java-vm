# Class 类文件结构

## 平台无关性

Java 是与平台无关的语言，这得益于 Java 源代码编译后生成的存储字节码的文件，即 Class 文件，以及 Java 虚拟机的实现。不仅使用 Java 编译器可以把 Java 代码编译成存储字节码的 Class 文件，使用 JRuby 等其他语言的编译器也可以把程序代码编译成 Class 文件，虚拟机并不关心 Class 的来源是什么语言，只要它符合一定的结构，就可以在 Java 中运行。Java 语言中的各种变量、关键字和运算符的语义最终都是由多条字节码命令组合而成的，因此字节码命令所能提供的语义描述能力肯定会比 Java 语言本身更强大，这便为其他语言实现一些有别于 Java 的语言特性提供了基础，而且这也正是在类加载时要进行安全验证的原因。

## 类文件结构

Class 文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在 Class 文件中，中间没有添加任何分隔符，这使得整个 Class 文件中存储的内容几乎全部都是程序运行的必要数据。根据 Java 虚拟机规范的规定，Class 文件格式采用一种类似于 C 语言结构体的伪结构来存储，这种伪结构中只有两种数据类型：无符号数和表。无符号数属于基本数据类型，以 u1、u2、u4、u8 来分别代表 1、2、4、8 个字节的无符号数。表是由多个无符号数或其他表作为数据项构成的符合数据类型，所有的表都习惯性地以“_info”结尾。

整个 Class 文件本质上就是一张表，它由如下所示的数据项构成。

从表中可以看出，无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的该数据项的形式，称这一系列连续的摸一个类型的数据为某一类型的集合，比如，fields_count 个 field_info 表数据构成了字段表集合。这里需要说明的是：Class 文件中的数据项，都是严格按照上表中的顺序和数量被严格限定的，每个字节代表的含义，长度，先后顺序等都不允许改变。

下表列出了 Class 文件中各个数据项的具体含义：

![](images/classtable.jpg)

从表中可以看出，无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会在其前面使用一个前置的容量计数器来记录其数量，而便跟着若干个连续的数据项，称这一系列连续的某一类型的数据为某一类型的集合，如：fields_count 个 field_info 表数据便组成了方法表集合。这里需要注意的是：Class 文件中各数据项是按照上表的顺序和数量被严格限定的，每个字节代表的含义、长度、先后顺序都不允许改变。

**magic 与 version**

每个 Class 文件的头 4 个字节称为魔数（magic），它的唯一作用是判断该文件是否为一个能被虚拟机接受的 Class 文件。它的值固定为 0xCAFEBABE。紧接着 magic 的 4 个字节存储的是 Class 文件的次版本号和主版本号，高版本的 JDK 能向下兼容低版本的 Class 文件，但不能运行更高版本的 Class 文件。

**constant_pool**

major_version 之后是常量池（constant_pool）的入口，它是 Class 文件中与其他项目关联最多的数据类型，也是占用 Class 文件空间最大的数据项目之一。

常量池中主要存放两大类常量：字面量和符号引用。字面量比较接近于 Java 层面的常量概念，如文本字符串、被声明为 final 的常量值等。而符号引用总结起来则包括了下面三类常量：

- 类和接口的全限定名（即带有包名的 Class 名，如：org.lxh.test.TestClass）
- 字段的名称和描述符（private、static 等描述符）
- 方法的名称和描述符（private、static 等描述符）

虚拟机在加载 Class 文件时才会进行动态连接，也就是说，Class 文件中不会保存各个方法和字段的最终内存布局信息，因此，这些字段和方法的符号引用不经过转换是无法直接被虚拟机使用的。当虚拟机运行时，需要从常量池中获得对应的符号引用，再在类加载过程中的解析阶段将其替换为直接引用，并翻译到具体的内存地址中。

这里说明下符号引用和直接引用的区别与关联：

- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到了内存中。
- 直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那说明引用的目标必定已经存在于内存之中了。

常量池中的每一项常量都是一个表，共有 11 种（JDK1.7 之前）结构各不相同的表结构数据，没中表开始的第一位是一个 u1 类型的标志位（1-12，缺少 2），代表当前这个常量属于的常量类型。11 种常量类型所代表的具体含义如下表所示：

![](images/constant_pool.jpg)


这 11 种常量类型各自均有自己的结构。在 CONSTANT_Class_info 型常量的结构中有一项 name_index 属性，该常属性中存放一个索引值，指向常量池中一个 CONSTANT_Utf8_info 类型的常量，该常量中即保存了该类的全限定名字符串。而 CONSTANT_Fieldref_info、CONSTANT_Methodref_info、CONSTANT_InterfaceMethodref_info 型常量的结构中都有一项index属性，存放该字段或方法所属的类或接口的描述符 CONSTANT_Class_info 的索引项。另外，最终保存的诸如 Class 名、字段名、方法名、修饰符等字符串都是一个 CONSTANT_Utf8_info 类型的常量，也因此，Java 中方法和字段名的最大长度也即是CONSTANT_Utf8_info 型常量的最大长度，在 CONSTANT_Utf8_info 型常量的结构中有一项 length 属性，它是 u2 类型的，即占用 2 个字节，那么它的最大的 length 即为 65535。因此，Java 程序中如果定义了超过 64KB 英文字符的变量或方法名，将会无法编译。

下表给出了常量池中 11 种数据类型的结构：
<table>
<tbody>
<tr>
<td valign="top" style="background:rgb(204,153,255)">
<p style="text-align:center">&nbsp; &nbsp; 常量</p>
</td>
<td valign="top" style="background:rgb(204,153,255)">
<p style="text-align:center">项目</p>
</td>
<td valign="top" style="background:rgb(204,153,255)">
<p style="text-align:center">&nbsp; 类型 &nbsp;</p>
</td>
<td valign="top" style="background:rgb(204,153,255)">
<p style="text-align:center">描述</p>
</td>
</tr>
<tr>
<td valign="top" rowspan="3" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Utf8_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p align="center" style="text-align:left">tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p style="text-align:left">u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">1</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>length</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>UF-8<span style="font-family:宋体">编码的字符串占用的字节数</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>bytes</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>长度为<span style="font-family:Times New Roman">length</span><span style="font-family:宋体">的</span><span style="font-family:Times New Roman">UTF-8</span><span style="font-family:宋体">编码的字符串</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Integer_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">3</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>bytes</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u4</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>按照高位在前存储的<span style="font-family:Times New Roman">int</span><span style="font-family:宋体">&#20540;</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Float_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">4</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>bytes</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u4</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>按照高位在前存储的<span style="font-family:Times New Roman">float</span><span style="font-family:宋体">&#20540;</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Long_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">5</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>bytes</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u8</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>按照高位在前存储的<span style="font-family:Times New Roman">long</span><span style="font-family:宋体">&#20540;</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Double_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">6</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>bytes</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u8</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>按照高位在前存储的<span style="font-family:Times New Roman">double</span><span style="font-family:宋体">&#20540;</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Class_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">7</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向全限定名常量项的索引</p>
</td>
</tr>
<tr>
<td valign="top" rowspan="2" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_String_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">8</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向字符串字面量的索引</p>
</td>
</tr>
<tr>
<td valign="top" rowspan="3" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Fieldref_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">9</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向声明字段的类或接口描述符<span style="font-family:Times New Roman">CONSTANT_Class_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向字段名称及类型描述符<span style="font-family:Times New Roman">CONSTANT_NameAndType_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="3" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_Methodref_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">10</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向声明方法的类描述符<span style="font-family:Times New Roman">CONSTANT_Class_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向方法名称及类型描述符<span style="font-family:Times New Roman">CONSTANT_NameAndType_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="3" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_InrerfaceMethodref_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">11</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向声明方法的接口描述符<span style="font-family:Times New Roman">CONSTANT_Class_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向方法名称及类型描述符<span style="font-family:Times New Roman">CONSTANT_NameAndType_info</span><span style="font-family:宋体">的索引项</span></p>
</td>
</tr>
<tr>
<td valign="top" rowspan="3" style="background:rgb(255,255,0)">
<p>&nbsp;</p>
<p>CONSTANT_NameAndType_info</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>tag</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u1</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>&#20540;为<span style="font-family:Times New Roman">12</span></p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向字段或方法名称常量项目的索引</p>
</td>
</tr>
<tr>
<td valign="top" style="background:rgb(255,255,0)">
<p>index</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>u2</p>
</td>
<td valign="top" style="background:rgb(255,255,0)">
<p>指向该字段或方法描述符常量项的索引</p>
</td>
</tr>
</tbody>
</table>

**access_flag**

在常量池结束之后，紧接着的 2 个字节代表访问标志（access_flag），这个标志用于识别一些类或接口层次的访问信息，包括：这个 Class 是类还是接口，是否定义为 public 类型，abstract 类型，如果是类的话，是否声明为 final，等等。每种访问信息都由一个十六进制的标志值表示，如果同时具有多种访问信息，则得到的标志值为这几种访问信息的标志值的逻辑或。

**this_class、super_class、interfaces**

类索引（this_class）和父类索引（super_class）都是一个 u2 类型的数据，而接口索引集合（interfaces）则是一组 u2 类型的数据集合，Class 文件中由这三项数据来确定这个类的继承关系。类索引、父类索引和接口索引集合都按照顺序排列在访问标志之后，类索引和父类索引两个 u2 类型的索引值表示，它们各自指向一个类型为 COMNSTANT_Class_info 的类描述符常量，通过该常量中的索引值找到定义在 COMNSTANT_Utf8_info 类型的常量中的全限定名字符串。而接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按 implements 语句（如果这个类本身是个接口，则应当是 extend 语句）后的接口顺序从左到右排列在接口的索引集合中。

**fields**

字段表（field_info）用于描述接口或类中声明的变量。字段包括了类级变量或实例级变量，但不包括在方法内声明的变量。字段的名字、数据类型、修饰符等都是无法固定的，只能引用常量池中的常量来描述。下面是字段表的最种格式：

![](images/fields.jpg)

其中的 access_flags 与类中的 access_flagsfei 类似，是表示数据类型的修饰符，如 public、static、volatile 等。后面的 name_index 和 descriptor_index 都是对常量池的引用，分别代表字段的简单名称及字段和方法的描述符。这里简单解释下“简单名称”、“描述符”和“全限定名”这三种特殊字符串的概念。

前面有所提及，全限定名即指一个事物的完整的名称，如在 org.lxh.test 包下的 TestClass 类的全限定名为：`org/lxh/test/TestClass`，即把包名中的“.”改为“/”，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“，”来表示全限定名结束。简单名称则是指没有类型或参数修饰的方法或字段名称，如果一个类中有这样一个方法 boolean  get(int name)和一个变量 private final static int m，则他们的简单名称则分别为 get()和 m。

而描述符的作用则是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序等）和返回值的。根据描述符规则，详细的描述符标示字的含义如下表所示：

![](images/fields1.jpg)

对于数组类型，每一维度将使用一个前置的 “[” 字符来描述，如一个整数数组 “int [][]” 将为记录为 “[[I” ，而一个 String 类型的数组 “String[]” 将被记录为 “[Ljava/lang/String” 。

用方法描述符描述方法时，按照先参数后返回值的顺序描述，参数要按照严格的顺序放在一组小括号内，如方法  int getIndex(String name,char[] tgc,int start,int end,char target) 的描述符为 “（Ljava/lang/String[CIIC）I”。

字段表包含的固定数据项目到 descriptor_index 为止就结束了，但是在它之后还紧跟着一个属性表集合用于存储一些额外的信息。比如，如果在类中有如下字段的声明：staticfinalint m = 2；那就可能会存在一项名为ConstantValue 的属性，它指向常量 2。关于 attribute_info 的详细内容，在后面关于属性表的项目中会有详细介绍。

最后需要注意一点：字段表集合中不会列出从父类或接口中继承而来的字段，但有可能列出原本 Java 代码中不存在的字段。比如在内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段。

**methods**

方法表（method_info）的结构与属性表的结构相同，不过多赘述。方法里的 Java 代码，经过编译器编译成字节码指令后，存放在方法属性表集合中一个名为“Code”的属性里，关于属性表的项目，同样会在后面详细介绍。

与字段表集合相对应，如果父类方法在子类中没有被覆写，方法表集合中就不会出现来自父类的方法信息。但同样，有可能会出现由编译器自动添加的方法，最典型的便是类构造器 “<clinit>” 方法和实例构造器 “<init>” 方法。

在 Java 语言中，要重载一个方法，除了要与原方法具有相同的简单名称外，还要求必须拥有一个与原方法不同的特征签名，特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合，也就是因为返回值不会包含在特征签名之中，因此 Java 语言里无法仅仅依靠返回值的不同来对一个已有方法进行重载。

**ttributes**

属性表（attribute_info）在前面已经出现过多系，在 Class 文件、字段表、方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。

属性表集合的限制没有那么严格，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，但 Java 虚拟机运行时会忽略掉它不认识的属性。Java 虚拟机规范中预定义了 9 项虚拟机应当能识别的属性（JDK1.5 后又增加了一些新的特性，因此不止下面 9 项，但下面 9 项是最基本也是必要，出现频率最高的），如下表所示：

![](images/attributes.jpg)

对于每个属性，它的名称都需要从常量池中引用一个 CONSTANT_Utf8_info 类型的常量来表示，每个属性值的结构是完全可以自定义的，只需说明属性值所占用的位数长度即可。一个符合规则的属性表至少应具有 “attribute_name_info”、“attribute_length” 和至少一项信息属性。

***Code 属性***

前面已经说过，Java 程序方法体中的代码讲过 javac 编译后，生成的字节码指令便会存储在 Code 属性中，但并非所有的方法表都必须存在这个属性，比如接口或抽象类中的方法就不存在 Code 属性。如果方法表有 Code 属性存在，那么它的结构将如下表所示：

![](images/code.jpg)

attribute_name_index 是一项指向 CONSTANT_Utf8_info 型常量的索引，常量值固定为 “Code”，它代表了该属性的名称。attribute_length 指示了属性值的长度，由于属性名称索引与属性长度一共是 6 个字节，所以属性值的长度固定为整个属性表的长度减去 6 个字节。

max_stack 代表了操作数栈深度的最大值，max_locals 代表了局部变量表所需的存储空间，它的单位是Slot，并不是在方法中用到了多少个局部变量，就把这些局部变量所占 Slot 之和作为 max_locals 的值，原因是局部变量表中的 Slot 可以重用。

code_length 和 code 用来存储 Java 源程序编译后生成的字节码指令。code 用于存储字节码指令的一系列字节流，它是 u1 类型的单字节，因此取值范围为 0x00 到 0xFF，那么一共可以表达 256 条指令，目前，Java 虚拟机规范已经定义了其中 200 条编码值对应的指令含义。code_length 虽然是一个 u4 类型的长度值，理论上可以达到 2^32-1，但是虚拟机规范中限制了一个方法不允许超过 65535 条字节码指令，如果超过了这个限制，Javac 编译器将会拒绝编译。

字节码指令之后是这个方法的显式异常处理表集合（exception_table），它对于 Code 属性来说并不是必须存在的。它的格式如下表所示：

![](images/exception_table.jpg)

它包含四个字段，这些字段的含义为：如果字节码从第 start_pc 行到第 end_pc 行之间（不含 end_pc 行）出现了类型为 catch_type 或其子类的异常（catch_type为指向一个 CONSTANT_Class_info 型常量的索引），则转到第 handler_pc 行继续处理，当 catch_pc 的值为 0 时，代表人和的异常情况都要转到 handler_pc 处进行处理。异常表实际上是 Java 代码的一部分，编译器使用异常表而不是简单的跳转命令来实现 Java 异常即 finally 处理机制，也因此，finally 中的内容会在 try 或 catch 中的 return 语句之前执行，并且在 try 或 catch 跳转到 finally 之前，会将其内部需要返回的变量的值复制一份副本到最后一个本地表量表的 Slot中，也因此便有了[http://blog.csdn.net/ns_code/article/details/17485221](http://blog.csdn.net/ns_code/article/details/17485221)这篇文章中出现的情况。

Code 属性是 Class 文件中最重要的一个属性，如果把一个 Java 程序中的信息分为代码和元数据两部分，那么在整个 Class 文件里，Code 属性用于描述代码，所有的其他数据项目都用于描述元数据。

***Exception 属性***

这里的 Exception 属性的作用是列举出方法中可能抛出的受查异常，也就是方法描述时在 throws 关键字后面列举的异常。它的结构很简单，只有 attribute_name_index、attribute_length、number_of_exceptions、exception_index_table 四项，从字面上便很容易理解，这里不再详述。


***LineNumberTable 属性***

它用于描述 Java 源码行号与字节码行号之间的对应关系。


***LocalVariableTable 属性***

它用于描述栈帧中局部变量表中的变量与 Java 源码中定义的变量之间的对应关系。


***SourceFile 属性***

它用于记录生成这个 Class 文件的源码文件名称。


***ConstantValue 属性***

ConstantValue 属性的作用是通知虚拟机自动为静态变量赋值，只有被 static 修饰的变量才可以使用这项属性。在 Java 中，对非 static 类型的变量（也就是实例变量）的赋值是在实例构造器<init>方法中进行的；而对于类变量（static 变量），则有两种方式可以选择：在类构造其中赋值，或使用 ConstantValue 属性赋值。

目前 Sun Javac 编译器的选择是：如果同时使用 final 和 static 修饰一个变量（即全局常量），并且这个变量的数据类型是基本类型或 String 的话，就生成 ConstantValue 属性来进行初始化（编译时 Javac 将会为该常量生成 ConstantValue 属性，在类加载的准备阶段虚拟机便会根据 ConstantValue 为常量设置相应的值），如果该变量没有被 final 修饰，或者并非基本类型及字符串，则选择在<clinit>方法中进行初始化。

虽然有 final 关键字才更符合”ConstantValue“的含义，但在虚拟机规范中并没有强制要求字段必须用 final 修饰，只要求了字段必须用 static 修饰，对 final 关键字的要求是 Javac 编译器自己加入的限制。因此，在实际的程序中，只有同时被 final 和 static 修饰的字段才有 ConstantValue 属性。而且 ConstantValue 的属性值只限于基本类型和 String，很明显这是因为它从常量池中也只能够引用到基本类型和 String 类型的字面量。

ConstantValue 属性的作用是通知虚拟机自动为静态变量赋值，只有被 static 修饰的变量才可以使用这项属性。在 Java 中，对非 static 类型的变量（也就是实例变量）的赋值是在实例构造器<init>方法中进行的；而对于类变量（static 变量），则有两种方式可以选择：在类构造其中赋值，或使用 ConstantValue 属性赋值。

目前 Sun Javac 编译器的选择是：如果同时使用 final 和 static 修饰一个变量（即全局常量），并且这个变量的数据类型是基本类型或 String 的话，就生成 ConstantValue 属性来进行初始化（编译时 Javac 将会为该常量生成 ConstantValue 属性，在类加载的准备阶段虚拟机便会根据 ConstantValue 为常量设置相应的值），如果该变量没有被 final 修饰，或者并非基本类型及字符串，则选择在<clinit>方法中进行初始化。

虽然有 final 关键字才更符合”ConstantValue“的含义，但在虚拟机规范中并没有强制要求字段必须用 final 修饰，只要求了字段必须用 static 修饰，对 final 关键字的要求是 Javac 编译器自己加入的限制。因此，在实际的程序中，只有同时被 final 和 static 修饰的字段才有 ConstantValue 属性。而且 ConstantValue 的属性值只限于基本类型和 String，很明显这是因为它从常量池中也只能够引用到基本类型和 String 类型的字面量。

下面简要说明下 final、static、static final 修饰的字段赋值的区别：

- static 修饰的字段在类加载过程中的准备阶段被初始化为 0 或 null 等默认值，而后在初始化阶段（触发类构造器<clinit>）才会被赋予代码中设定的值，如果没有设定值，那么它的值就为默认值。
- final 修饰的字段在运行时被初始化（可以直接赋值，也可以在实例构造器中赋值），一旦赋值便不可更改；
- static final 修饰的字段在 Javac 时生成 ConstantValue 属性，在类加载的准备阶段根据ConstantValue的值为该字段赋值，它没有默认值，必须显式地赋值，否则 Javac 时会报错。可以理解为在编译期即把结果放入了常量池中。

**InnerClasses 属性**

该属性用于记录内部类与宿主类之间的关联。如果一个类中定义了内部类，那么编译器将会为它及它所包含的内部类生成 InnerClasses 属性。

**Deprecated 属性和 Synthetic 属性**

该属性用于表示某个类、字段和方法，已经被程序作者定为不再推荐使用，它可以通过在代码中使用 @Deprecated 注释进行设置。

**Synthetic 属性**

该属性代表此字段或方法并不是 Java 源代码直接生成的，而是由编译器自行添加的，如 this 字段和实例构造器、类构造器等。





