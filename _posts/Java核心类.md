---
title: Java核心类
date: 2021-04-24 09:35:57
tags: Java
---

本节介绍Java核心类，包括字符串，StringBuilder，StringJoiner，包装类型，JavaBean，枚举，常用工具类。<!--more-->

## 字符串和编码

### String

在Java中，String是一个引用类型，它本身也是一个class。但是Java对String有特殊处理，可以直接用`"..."`来表示一个字符串。实际上字符串在String内部是通过一个`char[]`数组表示的。因为String太常用了，所以Java提供了这种字符串字面量的表示方法。

Java字符串的一个重要特点是`字符串不可变`。

当我们想要比较两个字符串是否相同时，我们实际上想比较字符串的内容是否相同。必须用`equals()`方法，而不能使用`==`。

要忽略大小写比较，使用`equalsIgnoreCase()`方法。

### 常用字符串操作

提取字串、查找、替换、大小写转换等。

使用`trim()`方法可以移除字符串首尾空白字符。空白字符包括空格，`\t`，`\r`，`\n`。注意，trim()并没有改变字符串内容，而是返回了一个新字符串。另一个`strip()`方法也可以移除字符串首尾空白字符。它和`trim()`不同的是，类似中文的空格字符`\u3000`也会被移除。

 `isEmpty()`和`isBlank()`来判断字符串是否为空和空白字符串。

有几个占位符，后面就传入几个参数。参数类型要和占位符一致。我们经常用这个方法来格式化信息。常用的占位符有：

- `%s`：显示字符串；
- `%d`：显示整数；
- `%x`：显示十六进制整数；
- `%f`：显示浮点数。

### 类型转换

要把任意基本类型或引用类型转换为字符串，可以使用静态方法`valueOf()`。 这是一个重载方法，编译器会根据参数自动选择合适的方法。要把字符串转换为其他类型，就需要根据情况。 

`String`和`char[]`类型可以互相转换。

```java
char[] cs = "Hello".toCharArray(); // String -> char[]
String s = new String(cs); // char[] -> String
```

从`String`的不变性设计可以看出，如果传入的对象有可能改变，我们需要复制而不是直接引用。

### 字符编码

在早期的计算机系统中，为了给字符编码，美国国家标准学会（American National Standard Institute：ANSI）制定了一套英文字母、数字和常用符号的编码，它占用一个字节，编码范围从`0`到`127`，最高位始终为`0`，称为`ASCII`编码。

类似的， `GB2312`标准使用两个字节表示一个汉字，日文有`Shift_JIS`编码，韩文有`EUC-KR`编码，这些编码因为标准不统一，同时使用，就会产生冲突。 

为了统一全球所有语言的编码，全球统一码联盟发布了`Unicode`编码，它把世界上主要语言都纳入同一个编码，这样，中文、日文、韩文和其他语言就不会冲突。 

那我们经常使用的`UTF-8`又是什么编码呢？因为英文字符的`Unicode`编码高字节总是`00`，包含大量英文的文本会浪费空间，所以，出现了`UTF-8`编码，它是一种变长编码，用来把固定长度的`Unicode`编码变成1～4字节的变长编码。 `UTF-8`编码的另一个好处是容错能力强。如果传输过程中某些字符出错，不会影响后续字符，因为`UTF-8`编码依靠高字节位来确定一个字符究竟是几个字节，它经常用来作为传输编码。 

## StringBuilder

Java编译器对String做了特殊处理，使得我们可以直接用`+`拼接字符串。String在拼接时总会创建新的字符串对象，然后扔掉旧的字符串。这样，绝大多数字符串都是临时对象，不但浪费内存，还影响GC效率。

为了能高效拼接字符串，Java标准库提供了`StringBuilder`，它是一个可变对象，可以预分配缓冲区。这样，往StringBuilder中新增字符时，不会创建新的临时对象。

StringBuilder还可以进行链式操作。

注意：对于普通字符串`+`操作，并不需要我们将其改写为StringBuilder，因为Java编译器在编译时就自动把多个连续的`+`操作编码为`StringConcatFactory`的操作。在运行期，StringConcatFactory会自动把字符串连接优化为数组复制或者StringBuilder操作。

你可能还听说过`StringBuffer`，这是Java早期的一个StringBuilder的线程安全版本，它通过同步来保证多个线程操作StringBuffer是安全的，但是同步会带来执行速度的下降。

## StringJoiner

用分隔符拼接数组的需求很常见，Java标准库提供了一个`StringJoiner`来干这个事。

```java
String[] names = {"Bob", "Alice", "Grace"};
var sj = new StringJoiner(", ", "Hello ", "!");
for (String name : names) {
	sj.add(name);
}//hello和!分别是开头和结尾
```

String还提供了一个静态方法`join()`，这个方法在内部使用了StringJoiner来拼接字符串，在不需要指定开头和结尾时，用`String.join()`更方便。

```java
String[] names = {"Bob", "Alice", "Grace"};
var s = String.join(", ", names);
```

## 包装类型

我们已经知道，Java的数据类型分两种：

基本类型：byte，short，int，long，boolean，float，double，char

引用类型：所有class和interface类型

引用类型可以赋值为`null`表示空，但基本类型不能赋值为null。

那么如何把一个基本类型视为对象（引用类型）呢？

比如，我们想要把int基本类型变成一个引用类型，我们可以定义一个`Interger`，它只包含一个int型的实例字段，这样就把Integer视为int的包装类型（Wrapper Class）。定义好了Interger，就可以把int和Interger相互转换。

实际上，因为包装类型非常有用，Java核心库为每种基本类型都提供了对应的包装类型。

### 自动装箱

Java编译器可以帮助我们自动在int和Interger之间转型。

```java
Integer n = 100; // 编译器自动使用Integer.valueOf(int)
int x = n; // 编译器自动使用Integer.intValue()
```

这种直接把int变为Interger的赋值写法，成为自动装箱（Auto Boxing），反过来，把Interger变为int的赋值写法，成为自动拆箱（Auto Unboxing）。

注意：自动装箱和自动拆箱只发生在编译阶段，目的是为了少写代码。装箱和拆箱会影响代码的执行效率，因为编译后的class代码是严格区分基本类型和引用类型的。并且，自动拆箱执行时可能会报`NullPointerException`。

### 不变类

所有的包装类型都是不变类。因此一旦创建了Interger对象，该对象就是不变的。对两个Interger实例进行比较时要特别注意，绝不能用`==`比较，因为Interger是引用类型，必须用`equals()`比较。

在我们自己创建Interger的时候，有以下两种方法：

```java
Interger n = new Interger(100);//方法一
Interger n = interger.valueOf(100);//方法二
```

方法二更好，因为方法一总是创建新的Interger实例，方法二把内部优化交给Interger的实现者来做，即使在当前版本没有优化，也有可能在下一个版本进行优化。

我们把能创建”新“对象的静态方法称为静态工厂方法。`Interger.valueOf()`就是静态工厂方法，它尽可能的返回缓存的实例以节省内存。

### 进制转换

Interger类本身还提供了大量方法，例如，最常用的静态方法`parseint()`可以把字符串解析成一个整数。

```java
int x1 = Integer.parseInt("100"); // 100
int x2 = Integer.parseInt("100", 16); // 256,因为按16进制解析
```

Integer还可以把整数格式化为指定进制的字符串。

Java的包装类型还定义了一些有用的静态变量。

```java
// boolean只有两个值true/false，其包装类型只需要引用Boolean提供的静态字段:
Boolean t = Boolean.TRUE;
Boolean f = Boolean.FALSE;
// int可表示的最大/最小值:
int max = Integer.MAX_VALUE; // 2147483647
int min = Integer.MIN_VALUE; // -2147483648
// long类型占用的bit和byte数量:
int sizeOfLong = Long.SIZE; // 64 (bits)
int bytesOfLong = Long.BYTES; // 8 (bytes)
```

最后，所有的整数和浮点数的包装类型都继承自`Number`，因此可以非常方便的通过包装类型获取各种基本类型。

```java
// 向上转型为Number:
Number num = new Integer(999);
// 获取byte, int, long, float, double:
byte b = num.byteValue();
int n = num.intValue();
long ln = num.longValue();
float f = num.floatValue();
double d = num.doubleValue();
```

### 处理无符号整型

在Java中，并没有无符号整型（Unsigned）的基本数据类型。byte，short，int和long都是带符号整型，最高位是符号位。无符号整型和有符号整型的转换在Java中就需要借助包装类型的静态方法完成。例如：

```java
byte x = -1;
byte y = Byte.toUnsignedInt(x);
```

## JavaBean

在Java中，有很多class都符合这样的规范：

- 若干private实例字段
- 通过public方法来读写实例字段

如果读写方法符合以下这种命名规范：

```java
//读方法
public Type getXyz();
//写方法
public void setXyz(Type value);
```

那么这种class被称为`JavaBean`。

上面的字段是xyz，那么读写方法分别以get和set开头，并且后接大写字母开头的字段名Xyz，因此读写方法分别是getXyz()和setXyz()。boolean字段比较特殊，它的读方法一般命名为isXyz()。

我们通常把一组对应的读方法（`getter`）和写方法（`setter`）称为属性（`property`）。只有getter的属性称为只读属性。

属性只需要定义getter和setter方法，不一定需要对应字段。可以看出，getter和setter也是一种数据封装的方法。

### JavaBean的作用

JavaBean主要用来传递数据，即把一组数据组合一个JavaBean便于传输。此外，JavaBean可以方便地被IDE分析，生成读写属性的代码，主要用在图形界面的可视化设计中。

### 枚举JavaBean

要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的`Introspector`。

```java
BeanInfo info = Introspector.getBeanInfo(Person.class);
for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
    System.out.println(pd.getName());
    System.out.println("  " + pd.getReadMethod());
    System.out.println("  " + pd.getWriteMethod());
}
```

## 枚举类

为了让编译器能自动检查某个值在枚举的集合内，并且不同用途的枚举需要不同的类型来标记，不能混用，我们可以使用`enum`来定义枚举类。

```java
enum Weekday {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}
```

注意到定义枚举类是通过关键字enum实现的，我们只需要依次列举出枚举的常量名。和int定义的常量相比，使用enum定义枚举有如下好处：

首先，enum常量本身带有类型信息，即Weekday.SUN类型是Weekday，编译器会自动检查出类型错误。其次，不可能引用到非枚举的值，因为无法通过编译。最后，不同类型的枚举不能相互比较或者赋值，因为类型不符。这就使得编译器可以在编译器自动检查出所有可能的潜在错误。

### enum的比较

使用enum定义的枚举类型是引用类型。引用类型比较，要使用equals()方法，如果使用==比较，它比较的是两个引用类型的变量是否是同一个对象。因此，引用类型比较，要始终使用equals()方法，但enum类型可以例外。 

因为enum类型的每个常量在JVM中只有一个唯一实例，所以可以直接用==比较。

### enum类型

通过enum定义的枚举类，和其他class有什么区别？答案是没有任何区别。enum定义的类型就是class，只不过它有以及几个特点。

- 定义的enum类型总是继承自java.lang.Enum，且无法被继承
- 只能定义出enum实例，而无法通过new操作符创建enum实例
- 定义的每个实例都是引用类型的唯一实例
- 可以将enum类型用于switch语句

因为enum是一个class，每个枚举的值都是class实例，这些实例有一些方法。

- name() 返回常量名
- ordinal() 返回定义的常量的顺序，从0开始计数

对枚举常量调用toString()会返回和name()一样的字符串。但是，toString()可以被覆写，而name()则不行。覆写toString()的目的是在输出时更有可读性。

## 纪录类

使用String，Integer等类型的时候，这些类型都是不变类，一个不变类具有以下几个特点：

- 定义class时使用final，无法派生子类
- 每个字段使用final，保证创建实例后无法修改任何字段

为了保证不变类的比较，还需要正确覆写equals()和hashCode()，这样才能在集合类中正常使用。

### record

从Java 14开始，引入了新的Record类。我们定义Record类时，使用关键字`record`。

```java
public record Point(int x, int y) {}

//改写为class
public final class Point extends Record {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() {
        return this.x;
    }

    public int y() {
        return this.y;
    }

    public String toString() {
        return String.format("Point[x=%s, y=%s]", x, y);
    }

    public boolean equals(Object o) {
        ...
    }
    public int hashCode() {
        ...
    }
}
```

除了用final修饰class以及每个字段外，编译器还为我们创建了构造方法，和字段名同名的方法，以及覆写toString()，equals()和hashCode()方法。换句话说，使用record关键字可以一行写出一个不变类。

和enum类似，我们自己不能直接从Record派生，只能通过record关键字由编译器实现继承。

### 构造方法

编译器默认按照record声明的变量顺序自动创建一个构造方法，并在方法内给字段赋值。那么问题来了，如果我们要检查参数，应该怎么办？

假设Point类的x，y不允许负数，我们就给Point的构造函数加上检查逻辑。

```java
public record Point(int x, int y) {
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException();
        }
    }
}
```

注意到方法`public Point{...}`被称为Compact Constructor，它的目的是让我们编写出检查逻辑，编译器最终生成的构造方法如下：

```java
public final class Point extends Record {
    public Point(int x, int y) {
        // 这是我们编写的Compact Constructor:
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException();
        }
        // 这是编译器继续生成的赋值代码:
        this.x = x;
        this.y = y;
    }
    ...
}
```

作为record的Point仍然可以添加静态方法，一般常用的静态的`of()`方法，用来创建Point。

```java
public record Point(int x, int y) {
    public static Point of() {
        return new Point(0, 0);
    }
    public static Point of(int x, int y) {
        return new Point(x, y);
    }
}
```

这样我们可以写出更简洁的代码。

```java
var z = Point.of();
var p = Point.of(123, 456);
```

## BigInteger

在Java中，由CPU原生提供的整型最大范围是64位long型整数。使用long型整数可以直接通过CPU指令进行计算，速度非常快。

如果我们使用的整数范围超过了long型怎么办？这时，就只能用软件来模拟一个大整数。`java.math.BigInteger`就是用来表示任意大小的整数。BigInteger内部使用一个int[]数组来模拟一个非常大的整数。

和long型整数运算相比，BigInteger不会有范围限制，但缺点是速度比较慢。

Biginteger和Integer、Long一样，也是不变类，也继承自Number类，因为Number定义了几种转换为基本类型的几个方法：

- 转换为byte：byteValue()
- 转换为short：shortValue()
- 转换为int：intValue()
- 转换为long：longValue()
- 转换为float：floatValue()
- 转换为double：doubleValue()

因此，通过上述方法，可以把BigInteger转换为基本类型。如果BigInteger表示的范围超过了基本类型的范围，转换时将丢失高位信息，即结果不一定是准确的。如果需要准确的转换成基本类型，可以使用intValueExact()、longValueExact()等方法，在转换时如果超出范围，将直接抛出`ArithmeticException`异常。

## BigDecimal

和BigInteger类似，BigDecimal可以表示一个任意大小且精度完全准确的浮点数。

BigDecimal用scale()表示小数位数。如果一个BigDecimal的scale()返回负数，例如返回-2，表示这个数是整数，并且末尾有两个0。可以对一个BigDecimal设置scale，如果精度比原始值低，那么按照指定的方法进行四舍五入或者直接截断。

stripTrailingZeros()可以将一个BigDecimal格式化为一个相等的，但去掉了末尾0的BigDecimal。

对BigDecimal做加减乘，精度不会丢失，但是做除法时，存在除不尽的情况，这时必须指定精度以及如何截断。

比较BigDecimal

在比较两个BigDecimal的值是否相等时，要特别注意，使用equals()方法不但要求两个BigDecimal值相等，还要求他们的scale()相等。使用compareTo()方法来比较，它根据两个值的大小分别返回负数、正数和0，分别表示小于，大于和等于。

BigDecimal也是从Number继承的，也是不可变对象。

## 常用工具类

Java的核心库提供了大量的现成的类供我们使用。

### Math

Math类是用来进行数学计算的，它提供了大量的静态方法来便于我们实现数学计算。

求绝对值，最值，计算x^y次方，e^x，以e为低的对数，三角函数，几个数学常量，生成随机数等等。

### Random

`Random`用来创建伪随机数。所谓伪随机数，是指只要给定一个初始的种子，产生的随机数序列是完全一样的。

要生成一个随机数，可以使用`nextInt()`、`nextLong()`、`nextFloat()`、`nextDouble()`。

我们在创建Random实例时，如果不给定种子，就使用系统当前时间戳作为种子，因此每次运行时，种子不同得到的随机数序列也就不同。如果我们创建Random实例时指定一个种子，就会得到完全相同的随机数序列。

### SecureRandom

有伪随机数，就有真随机数。实际上真正的真随机数只能通过量子力学原理来获取，而我们想要的是一个不可预测的安全的随机数，SecureRandom就是用来创建安全的随机数的。

> `SecureRandom`无法指定种子，它使用RNG（random number generator）算法。JDK的`SecureRandom`实际上有多种不同的底层实现，有的使用安全随机种子加上伪随机数算法来产生安全的随机数，有的使用真正的随机数生成器。实际使用的时候，可以优先获取高强度的安全随机数生成器，如果没有提供，再使用普通等级的安全随机数生成器： 
>
> `SecureRandom`的安全性是通过操作系统提供的安全的随机种子来生成随机数。这个种子是通过CPU的热噪声、读写磁盘的字节、网络流量等各种随机事件产生的“熵”。
>
> 在密码学中，安全的随机数非常重要。如果使用不安全的伪随机数，所有加密体系都将被攻破。因此，时刻牢记必须使用`SecureRandom`来产生安全的随机数。

