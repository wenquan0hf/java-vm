# Java 语法糖

语法糖（Syntactic Sugar），也称糖衣语法，是由英国计算机学家 Peter.J.Landin 发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。Java 中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构，这个过程成为解语法糖。

泛型是 JDK1.5 之后引入的一项新特性，Java 语言在还没有出现泛型时，只能通过 Object 是所有类型的父类和类型强制转换这两个特点的配合来实现泛型的功能，这样实现的泛型功能要在程序运行期才能知道 Object 真正的对象类型，在 javac 编译期，编译器无法检查这个 Object 的强制转型是否成功，这便将一些风险转接到了程序运行期中。

Java 语言在 JDK1.5 之后引入的泛型实际上只在程序源码中存在，在编译后的字节码文件中，就已经被替换为了原来的原生类型，并且在相应的地方插入了强制转型代码，因此对于运行期的Java语言来说，ArrayList<String>和 ArrayList<Integer>就是同一个类。所以泛型技术实际上是 Java 语言的一颗语法糖，Java 语言中的泛型实现方法称为类型擦除，基于这种方法实现的泛型被称为伪泛型。

下面是一段简单的 Java 泛型代码：

```
Map<Integer,String> map = new HashMap<Integer,String>();  
map.put(1,"No.1");  
map.put(2,"No.2");  
System.out.println(map.get(1));  
System.out.println(map.get(2));  
```

将这段 Java 代码编译成 Class 文件，然后再用字节码反编译工具进行反编译后，将会发现泛型都变回了原生类型，如下面的代码所示：

```
Map map = new HashMap();  
map.put(1,"No.1");  
map.put(2,"No.2");  
System.out.println((String)map.get(1));  
System.out.println((String)map.get(2));  
```

为了更详细地说明类型擦除，再看如下代码：

```
import java.util.List;  
public class FanxingTest{  
    public void method(List<String> list){  
        System.out.println("List String");  
    }  
    public void method(List<Integer> list){  
        System.out.println("List Int");  
    }  
}  
```

当用 javac 编译器编译这段代码时，报出了如下错误：

```
FanxingTest.java:3: 名称冲突：method(java.util.List<java.lang.String>) 和 method

(java.util.List<java.lang.Integer>) 具有相同疑符

        public void method(List<String> list){

                    ^

FanxingTest.java:6: 名称冲突：method(java.util.List<java.lang.Integer>) 和 metho

d(java.util.List<java.lang.String>) 具有相同疑符

        public void method(List<Integer> list){

                    ^

2 错误
```

是因为泛型 List<String> 和 List<Integer> 编译后都被擦除了，变成了一样的原生类型 List，擦除动作导致这两个方法的特征签名变得一模一样，在 Class 类文件结构一文中讲过，Class 文件中不能存在特征签名相同的方法。

把以上代码修改如下：

```
import java.util.List;  
public class FanxingTest{  
    public int method(List<String> list){  
        System.out.println("List String");  
        return 1;  
    }  
    public boolean method(List<Integer> list){  
        System.out.println("List Int");  
        return true;  
    }  
}  
```

发现这时编译可以通过了（注意：Java 语言中 true 和 1 没有关联，二者属于不同的类型，不能相互转换，不存在 C 语言中整数值非零即真的情况）。两个不同类型的返回值的加入，使得方法的重载成功了。这是为什么呢？

我们知道，Java 代码中的方法特征签名只包括了方法名称、参数顺序和参数类型，并不包括方法的返回值，因此方法的返回值并不参与重载方法的选择，这样看来为重载方法加入返回值貌似是多余的。对于重载方法的选择来说，这确实是多余的，但我们现在要解决的问题是让上述代码能通过编译，让两个重载方法能够合理地共存于同一个 Class 文件之中，这就要看字节码的方法特征签名，它不仅包括了 Java 代码中方法特征签名中所包含的那些信息，还包括方法返回值及受查异常表。为两个重载方法加入不同的返回值后，因为有了不同的字节码特征签名，它们便可以共存于一个 Class 文件之中。

自动拆装箱、变长参数等语法糖也都是在编译阶段就把它们该语法糖结构还原为了原生的语法结构，因此在 Class 文件中也只存在其对应的原生类型，这里不再一一说明。

