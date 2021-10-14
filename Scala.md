# Scala

##### HelloWorld

###### HelloWorld.scala

```scala
/*
   object: 关键字，声明一个单例对象（伴生对象）
 */
object HelloWorld {
  /*
    main 方法：从外部可以直接调用执行的方法
    def 方法名称(参数名称: 参数类型): 返回值类型 = { 方法体 }
   */
  def main(args: Array[String]): Unit = {
    println("hello world")
    System.out.println("hello scala from java")
  }
```

> 产生两个.class字节码文件，伴生类（HelloWorld.class），伴生对象的所属类（HelloWorld$.class）

###### **Student.java**

```java
public class Student {
    private String name;
    private Integer age;
    private static String school = "CQUPT";

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public void printInfo() {
        System.out.println(this.name + " " + this.age + " " + Student.school);
    }

    public static void main(String[] args) {
        Student alice = new Student("alice", 20);
        Student bob = new Student("bob", 23);
        alice.printInfo();
        bob.printInfo();
    }
}
```

###### **Student.scala**

```scala
class Student(name: String, var age: Int) {
  def printInfo(): Unit = {
    println(name + " " + age + " " + Student.school)
  }
}

// 引入伴生对象
object Student{
  val school: String = "CQUPT"

  def main(args: Array[String]): Unit = {
    val alice = new Student("alice", 20)
    val bob = new Student("bob", 23)

    alice.printInfo()
    bob.printInfo()
  }
}
```

> Java中通过类名Student调用了静态的属性school，也即通过类来调用了其属性，而这似乎看起来并不是真正的面向对象，若是真正的面向对象，就应该也是通过对象来调用所有属性。
>
> 于是，为了解决这个问题，在Scala中就出现了伴生对象，即Scala中的`object Student`是`class Student`的伴生对象。
>
> 如果再将全局只有一份的属性，定义为伴生对象里的一个属性，那么就可以通过伴生对象的实例来调用这个属性，从而避免了Java中的类名.属性的方式来调用属性。
>
> 目前有种不知是否正确的看法——其实就很像是把原Java类`XXX`中的静态属性、静态方法单独拧出来将其放入伴生对象`object XXX`中，而自身就成为了伴生类？

##### 变量和数据类型

###### Test01_Comment.scala

> 【注释】Scala 注释使用和  Java 完全一样

```scala
/*
   这是一个简单的测试程序
   测试注释
 */
object Test01_Comment {
  /**
   * 程序的入口方法
   * @param args 外部传入的参数
   */
  def main(args: Array[String]): Unit = {
    // 打印输出
    println("hello")
  }
}
```

###### Test02_Variable.scala

> 【变量和常量】

```scala
object Test02_Variable {
  def main(args: Array[String]): Unit = {
    // 声明一个变量的通用语法
    var a: Int = 10

    //（1）声明变量时，类型可以省略，编译器自动推导，即类型推导
    var a1 = 10
    val b1 = 23

    //（2）类型确定后，就不能修改，说明Scala是强数据类型语言
    var a2 = 15    // a2类型为Int
//    a2 = "hello" // 不允许

    //（3）变量声明时，必须要有初始值
//    var a3: Int  // 不允许

    //（4）在声明/定义一个变量时，可以使用var或者val来修饰，var修饰的变量可改变，val修饰的变量不可改
    a1 = 12
//    b1 = 25 // 在（1）中已经定义了b1为常量23，于是不能再赋值，即便再赋值23也不允许

    var alice = new Student("alice", 20)
    alice = new Student("Alice", 20)
    alice = null
    val bob = new Student("bob", 23)
    bob.age = 24 // class Student(name: String, var age: Int) {}
    bob.printInfo()
//    bob = new Student("bob", 24) // 可对属性进行更改，但是自身不能改变
  }
}
```

> 常量：在程序执行的过程中，其值不会被改变的变量
> 0）回顾 Java 变量 和常量语法
> 变量类型 变量名称 = 初始值           int a = 10
> final 常量类型 常量名称 = 初始值  final int b = 20
> 1）基本语法
> **var** 变量名 [: 变量类型] = 初始值     var i:Int = 10
> **val** 常量名 [: 常量类型]  = 初始值     val j:Int = 20
>
> 注意：能用常量的地方不用变量
>
> var 修饰的对象引用可以改变，val 修饰的对象则不可改变（内存地址不能变），但对象的状态（值）却是可以改变的。（比如：自定义对象、数组、集合等等）

###### Test03_Identifier

> 【标识符】

```scala
object Test03_Identifier {
  def main(args: Array[String]): Unit = {
    //（1）以字母或者下划线开头，后接字母、数字、下划线
    val hello: String = ""
    var Hello123 = ""
    val _abc = 123

//    val h-b = "" // 不允许
//    val 123abc = 234 // 不允许

    //（2）以操作符开头，且只包含操作符（+ - * / # !等）
    val -+/% = "hello"
    println(-+/%)

    //（3）用反引号`....`包括的任意字符串，即使是Scala关键字（39个）也可以
    val `if` = "if"
    println(`if`)
  }
}
```

> Scala（ 39 个）关键字
>
> • package, import, class, **object** , **trait** , extends, **with** , type, for
> • private, protected, abstract, **sealed** , final, **implicit** , lazy, override
> • try, catch, finally, throw
> • if, else, **match** , case, do, while, for, return, **yield**
> • **def**, **val**, **var**
> • this, super
> • new
> • true, false, null

###### Test04_String.scala

> 【字符串输出】

```scala
object Test04_String {
  def main(args: Array[String]): Unit = {
    //（1）字符串，通过+号连接
    val name: String = "alice"
    val age: Int = 18
    println(age + "岁的" + name + "学习Scala")

    // *用于将一个字符串复制多次并拼接
    println(name * 3)

    //（2）printf用法：字符串，通过%传值
    printf("%d岁的%s学习Scala", age, name)
    println()

    //（3）字符串模板（插值字符串）：通过$获取变量值
    println(s"${age}岁的${name}学习Scala")

    val num: Double = 2.3456
    println(f"The num is ${num}%2.2f")    // 格式化模板字符串//**is 2.35
    println(raw"The num is ${num}%2.2f")	// The num is 2.3456%2.2f

    // 三引号表示字符串，保持多行字符串的原格式输出
    val sql = s"""
       |select *
       |from
       |  student
       |where
       |  name = ${name}
       |and
       |  age > ${age}
       |""".stripMargin
    println(sql)
  }
}
```

###### Test05_StdIn.scala

> 【键盘输入】

```scala
import scala.io.StdIn

object Test05_StdIn {
  def main(args: Array[String]): Unit = {
    // 输入信息
    println("请输入您的大名：")
    val name: String = StdIn.readLine()
    println("请输入您的芳龄：")
    val age: Int = StdIn.readInt()

    // 控制台打印输出
    println(s"${age}岁的${name}学习Scala")
  }
}
```

###### Test06_FileIO.scala

> 【读取写入文件】

```scala
import java.io.{File, PrintWriter}
import scala.io.Source

object Test06_FileIO {
  def main(args: Array[String]): Unit = {
    // 1. 从文件中读取数据
    Source.fromFile("src/main/resources/test.txt").foreach(print)

    // 2. 将数据写入文件
    val writer = new PrintWriter(new File("src/main/resources/output.txt"))
    writer.write("hello scala from java writer")
    writer.close()
  }
}
```

###### Test07_DataType.scala

> 【数据类型】整数类型、浮点类型、字符类型、布尔类型、Unit类型、Null类型和Noting类型

| 数据类型     | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| Byte [1]     | 8位有符号补码整数。数值区间为 -128 到 127                    |
| Short [2]    | 16位有符号补码整数。数值区间为 -32768 到 32767               |
| Int [4]      | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
| Long [8]     | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 = 2的(64-1)次方-1 |
| Float [4]    | 32位，IEEE 754标准的单精度浮点数                             |
| Double [8]   | 64位，IEEE 754标准的双精度浮点数                             |
| Unit[空值]   | 表示无值，和其他语言中void等同，用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成() |
| Null[空引用] | null , Null类型只有一个实例值null                            |
| Noting       | Nothing类型在Scala的类层级最低端，它是任何其他类型的子类型。 当一个函数，我们确定没有正常的返回值，可以用Nothing来指定返回类型，这样有一个好处，就是我们可以把返回的值（异常）赋给其它的函数或者变量（兼容性） |

```scala
import chapter01.Student

object Test07_DataType {
  def main(args: Array[String]): Unit = {
    // 1. 整数类型
    val a1: Byte = 127
    val a2: Byte = -128

//    val a2: Byte = 128    // error

    val a3 = 12    // 整数默认类型为Int
    val a4: Long = 1324135436436L	// 长整型数值定义
//    val a4 = 1324135436436	// error
//    val a4: Long = 1324135436436	// error

    val b1: Byte = 10
    val b2: Byte = 10 + 20
    println(b2)

//    val b3: Byte = b1 + 20
    val b3: Byte = (b1 + 20).toByte
    println(b3)

    // 2. 浮点类型
    val f1: Float = 1.2345f
    val d1 = 34.2245	//默认为double类型

    // 3. 字符类型
    val c1: Char = 'a'
    println(c1)

    val c2: Char = '9'
    println(c2)

    // 控制字符
    val c3: Char = '\t'    // 制表符
    val c4: Char = '\n'    // 换行符
    println("abc" + c3 + "def")
    println("abc" + c4 + "def")

    // 转义字符
    val c5 = '\\'    // 表示\自身
    val c6 = '\"'    // 表示"
    println("abc" + c5 + "def")
    println("abc" + c6 + "def")

    // 字符变量底层保存ASCII码
    val i1: Int = c1
    println("i1: " + i1)	// 97
    val i2: Int = c2
    println("i2: " + i2)	// 57

    val c7: Char = (i1 + 1).toChar
    println(c7)	// b
    val c8: Char = (i2 - 1).toChar
    println(c8)	// 8

    // 4. 布尔类型 占1个字节
    val isTrue: Boolean = true
    println(isTrue)

    // 5. 空类型
    // 5.1 空值Unit
    def m1(): Unit = {
      println("m1被调用执行")
    }

    val a: Unit = m1()
    println("a: " + a)
    // 输出
    // m1被调用执行
    // a: ()

    // 5.2 空引用Null
//    val n: Int = null    // error
    var student: Student = new Student("alice", 20)
    student = null
    println(student)	// null

    // 5.3 Nothing  //Nothing是Int类（任何类）的子类
    def m2(n: Int): Int = {
      if (n == 0)
        throw new NullPointerException
      else
        return n
    }

    val b: Int = m2(2)
    println("b: " + b)  // b: 2
  }
}
```

###### Test08_DataTypeConversion.scala

> 【类型转换】自动类型转换、强制类型转换、数值类型和String类型的转换

```scala
object Test08_DataTypeConversion {
  def main(args: Array[String]): Unit = {

    // 1. 自动类型转换
    //（1）自动提升原则：有多种类型的数据混合运算时，系统首先自动将所有数据转换成精度大的那种数据类型，然后再进行计算。
    val a1: Byte = 10
    val b1: Long = 2353
    val result1: Long = a1 + b1
    val result11: Int = (a1 + b1.toInt) // 强转

    //（2）把精度大的数值类型赋值给精度小的数值类型时，就会报错，反之就会进行自动类型转换。
    val a2: Byte = 10
    val b2: Int = a2
    //    val c2: Byte = b2    // error

    //（3）（byte，short）和char之间不会相互自动转换。
    val a3: Byte = 10
    val b3: Char = 'b'
    //    val c3: Byte = b3    // error
    val c3: Int = b3
    println(c3)

    //（4）byte，short，char他们三者可以计算，在计算时首先转换为int类型。
    val a4: Byte = 12
    val b4: Short = 25
    val c4: Char = 'c'
    val result4: Int = a4 + b4
    val result44: Int = a4 + b4 + c4
    println(result44)

    // 2. 强制类型转换
    //（1）将数据由高精度转换为低精度，就需要使用到强制转换
    val n1: Int = -2.9.toInt
    println("n1: " + n1)	// 2.9.toInt -> n1: 2	// -2.9.toInt -> n1: -2

    //（2）强转符号只针对于最近的操作数有效，往往会使用小括号提升优先级
    val n2: Int = 2.6.toInt + 3.7.toInt
    val n3: Int = (2.6 + 3.7).toInt
    println("n2: " + n2)	// n2: 5
    println("n3: " + n3)	// n3: 6

    // 3. 数值类型和String类型的转换
    //（1）数值转String
    val n: Int = 27
    val s: String = n + ""
    println(s)	// 27

    //（2）String转数值
    val m: Int = "12".toInt
    val f: Float = "12.3".toFloat
    val f2: Int = "12.3".toDouble.toInt
    // val f2: Int = "12.3".toInt会出现NumberFormatException异常
    println(f2)	// 12
  }
}
```

###### Test09_Problem_DataTypeConversion.scala

> 【一个题目】

```scala
/*
128: Int类型，占据4个字节，32位
原码 0000 0000 0000 0000 0000 0000 1000 0000
补码 0000 0000 0000 0000 0000 0000 1000 0000

截取最后一个字节，Byte
得到补码 1000 0000
表示最大负数 -128

130: Int类型，占据4个字节，32位
原码 0000 0000 0000 0000 0000 0000 1000 0010
补码 0000 0000 0000 0000 0000 0000 1000 0010

截取最后一个字节，Byte
得到补码 1000 0010
对应原码 1111 1110
-126
 */

object Test09_Problem_DataTypeConversion {
  def main(args: Array[String]): Unit = {
    var n: Int = 130
    val b: Byte = n.toByte
    println(b)
  }
}
```

##### 运算符

###### TestOperator.java

> 【Java中的运算符】

```java
public class TestOperator {
    public static void main(String[] args) {
        // 比较运算符
        String s1 = "hello";
        String s2 = new String("hello");

        Boolean isEqual = s1 == s2;
        System.out.println(isEqual);	// false
        System.out.println(s1.equals(s2));	// true

        System.out.println("========================");

        // 赋值运算符
        byte b = 10;
        b = (byte)(b + 1);
        b += 1;    // 默认会做强转
        System.out.println(b);	// 12

        // 自增自减
        int x = 15;
        int y = x++;
        System.out.println("x = " + x + ", y = " + y);	// 16 15

        x = 15;
        y = ++x;
        System.out.println("x = " + x + ", y = " + y);	// 16 16

        x = 23;
        x = x++;
        System.out.println(x);	// 23	// temp = x++; x = temp;
    }
}
```

###### Test01_TestOperator.scala

【运算符】算术运算符、比较运算符、逻辑运算符、赋值运算符、位运算符、运算符的本质

```scala
object Test01_TestOperator {
  def main(args: Array[String]): Unit = {
    // 1. 算术运算符
    val result1: Int = 10 / 3
    println(result1)	// 3

    val result2: Double = 10 / 3
    println(result2)	// 3.0

    val result3: Double = 10.0 / 3
    println(result3.formatted("%5.2f"))	// _3.33	// 此处至少要有5位，少于5位要在前添加空格

    val result4: Int = 10 % 3
    println(result4)	// 1

    // 2. 比较运算符
    val s1: String = "hello"
    val s2: String = new String("hello")

    println(s1 == s2)	// true
    println(s1.equals(s2))	// true
    println(s1.eq(s2))	// false	// 判断s1,s2的引用是否相等

    println("===================")

    // 3. 逻辑运算符
    def m(n: Int): Int = {
      println("m被调用")
      return n
    }

    val n = 1
    println((4 > 5) && m(n) > 0)	// false	// 不会输出"m被调用"

    // 判断一个字符串是否为空
    def isNotEmpty(str: String): Boolean = {
      return str != null && !("".equals(str.trim))
    }

    println(isNotEmpty(null))	// false

    // 4. 赋值运算符
//    var b: Byte = 10
//    b += 1	// error
    var i: Int = 12
    i += 1
    println(i)

//    i++

    // 5. 位运算符
    val a: Byte = 60
    println(a << 3)	// 480
    println(a >> 2)	// 15

    val b: Short = -13
    println(b << 2)	// -52
    println(b >> 2)	// -4
    println(b >>> 2)	// 1073741820

    // 6. 运算符的本质
    val n1: Int = 12
    val n2: Int = 37

    println(n1.+(n2))	// 49
    println(n1 + n2)	// 49

    println(1.34.*(25))	// 33.5
    println(1.34 * 25)	// 33.5

//    println(7.5 toInt toString)	// 这个要报错
    println(7.5 toInt)	// 7
    println(7.5.toInt.toFloat.toString)	// 7.0
  }
}
```

> Scala删掉了Java中的自增自减，即Scala中没有++、--操作符，可以通过+=、-=来实现同样的效果
>
> val b: Short = -13
>
> -13原码：1000 0000 0000 1101
>
> -13补码：1111 1111 1111 0011
>
> -13 << 2后补码：1111 1111 1100 1100 （左移右边补0）
>
> 再还原为原码：1000 0000 0011 0100 表示 -52
>
> -13 >> 2后补码：1111 1111 1111 1100 （右移左边补1）
>
> 再还原为原码：1000 0000 0000 0100 表示 -4
>
> -13 >>> 2后补码：00111111 11111111 11111111 11111100 不知道这里为什么是32位
>
> 再还原为原码：00111111 11111111 11111111 11111100 表示2 ^ 31 - 4 = 1073741820

##### 流程控制

###### Test01_IfElse.scala

【分支控制if-else】单分支、双分支、多分支、分支语句的返回值、嵌套分支

```scala
import scala.io.StdIn

object Test01_IfElse {
  def main(args: Array[String]): Unit = {
    println("请输入您的年龄：")
    val age: Int = StdIn.readInt()

    // 1. 单分支
    if (age >= 18) {
      println("成年")
    }
    println("===================")

    // 2. 双分支
    if (age >= 18) {
      println("成年")
    } else {
      println("未成年")
    }
    println("===================")

    // 3. 多分支
    if (age <= 6) {
      println("童年")
    } else if (age < 18) {
      println("青少年")
    } else if (age < 35) {
      println("青年")
    } else if (age < 60) {
      println("中年")
    } else {
      println("老年")
    }
    println("===================")

    // 4. 分支语句的返回值
    val result: Any = if (age <= 6) {
      println("童年")
      "童年"
    } else if (age < 18) {
      println("青少年")
      "青少年"
    } else if (age < 35) {
      println("青年")
      age
    } else if (age < 60) {
      println("中年")
      age
    } else {
      println("老年")
      age
    }
    println("result: " + result)

    // java中三元运算符 String res = (age >= 18)?"成年":"未成年"
    val res: String = if (age >= 18) {
      "成年"
    } else {
      "未成年"
    }

    val res2 = if (age >= 18) "成年" else "未成年"
    println("===================")

    // 5. 嵌套分支
    if (age >= 18) {
      println("成年")
      if (age >= 35) {
        if (age >= 60) {
          println("老年")
        } else {
          println("中年")
        }
      } else {
        println("青年")
      }
    } else {
      println("未成年")
      if (age <= 6) {
        println("童年")
      } else {
        println("青少年")
      }
    }
  }
}
```

###### Test02_ForLoop.scala

> 【循环控制for】范围遍历（to、until）、循环守卫、循环步长（by）、嵌套循环、循环引入变量、循环返回值

```scala
import scala.collection.immutable

object Test02_ForLoop {
  def main(args: Array[String]): Unit = {
    // java for语法： for(int i = 0; i < 10; i++){ System.out.println(i + ". hello world") }

    // 1. 范围遍历
    for (i <- 1 to 10) {
      println(i + ". hello world")
    }
    for (i: Int <- 1.to(10)) {
      println(i + ". hello world")
    }

    println("======================")

    //    for (i <- Range(1, 10)){
    //      println(i + ". hello world")	// 输出9次
    //    }
    for (i <- 1 until 10) {
      println(i + ". hello world")	// 输出9次
    }

    println("======================")

    // 2. 集合遍历
    for (i <- Array(12, 34, 53)) {
      println(i)
    }
    for (i <- List(12, 34, 53)) {
      println(i)
    }
    for (i <- Set(12, 34, 53)) {
      println(i)
    }

    println("======================")

    // 3. 循环守卫
    for (i <- 1 to 10) {
      if (i != 5) {
        println(i)
      }
    }

    for (i <- 1 to 10 if i != 5) {
      println(i)
    }

    println("======================")

    // 4. 循环步长
    for (i <- 1 to 10 by 2) {
      println(i)	// 1 3 5 7 9
    }
    println("-------------------")
    for (i <- 13 to 30 by 3) {
      println(i)
    }

    println("-------------------")
    for (i <- 30 to 13 by -2) {
      println(i)
    }
    for (i <- 10 to 1 by -1) {
      println(i)
    }
    println("-------------------")
    for (i <- 1 to 10 reverse) {
      println(i)
    }
    println("-------------------")
    //    for (i <- 30 to 13 by 0){
    //      println(i)
    //    }    // error，step不能为0

    for (data <- 1.0 to 10.0 by 0.3) {	// to 过期了，有可能精度缺失
      println(data)
    }

    println("======================")

    // 5. 循环嵌套
    for (i <- 1 to 3) {
      for (j <- 1 to 3) {
        println("i = " + i + ", j = " + j)
      }
    }
    println("-------------------")
    for (i <- 1 to 4; j <- 1 to 5) {
      println("i = " + i + ", j = " + j)
    }

    println("======================")

    // 6. 循环引入变量
    for (i <- 1 to 10) {
      val j = 10 - i
      println("i = " + i + ", j = " + j)
    }

    for (i <- 1 to 10; j = 10 - i) {
      println("i = " + i + ", j = " + j)
    }

    for {
      i <- 1 to 10
      j = 10 - i
    } {
      println("i = " + i + ", j = " + j)
    }

    println("======================")

    // 7. 循环返回值 默认返回空() 开发中很少使用
    val a = for (i <- 1 to 10) {
      i
    }
    println("a = " + a)	// a = ()

    val b: immutable.IndexedSeq[Int] = for (i <- 1 to 10) yield i * i
    println("b = " + b)	// b = Vector(1, 4, 9, 16, 25, 36, 49, 64, 81, 100)
  }
}
```

###### Test03_Practice_MulTable.scala

> 【练习】九九乘法表

```scala
// 输出九九乘法表
object Test03_Practice_MulTable {
  def main(args: Array[String]): Unit = {
    for (i <- 1 to 9) {
      for (j <- 1 to i) {
        print(s"$j * $i = ${i * j} \t")
      }
      println()
    }

    // 简写
    for (i <- 1 to 9; j <- 1 to i) {
      print(s"$j * $i = ${i * j} \t")
      if (j == i) println()
    }
  }
}
```

###### Test04_Practice_Pyramid.scala

> 【练习】九层妖塔

```scala
// 打印输出一个九层妖塔
object Test04_Practice_Pyramid {
  def main(args: Array[String]): Unit = {
    for (i <- 1 to 9) {
      val stars = 2 * i - 1
      val spaces = 9 - i
      println(" " * spaces + "*" * stars)
    }

    for (i <- 1 to 9; stars = 2 * i - 1; spaces = 9 - i) {
      println(" " * spaces + "*" * stars)
    }

    for (stars <- 1 to 17 by 2; spaces = (17 - stars) / 2) {
      println(" " * spaces + "*" * stars)
    }
  }
}
```

###### Test05_WhileLoop.scala

> 【循环控制while，do...while】不推荐使用

```scala
object Test05_WhileLoop {
  def main(args: Array[String]): Unit = {
    // while
    var a: Int = 10
    while (a >= 1) {
      println("this is a while loop: " + a)
      a -= 1
    }

    var b: Int = 0
    do {
      println("this is a do-while loop: " + b)
      b -= 1
    } while (b > 0)
  }
}
```

> 说明：
> （1）循环条件是返回一个布尔值的表达式
> （2）while 循环是先判断再执行语句
> （3）与 for 语句不同，while 语句没有返回值 ，即整个 while 语句的结果是 Unit 类型()
> （4）因为 while 中没有返回值，所以当要用该语句来计算并返回结果时，就不可避免的使用变量，而变量需要声明在 while 循环的外部，那么就等同于循环的内部对外部的变量造成了影响，所以不推荐使用，而是推荐使用 for 循环 。

###### Test06_Break.scala

> 【循环中断】

TestBreak.java

```java
public class TestBreak {
    public static void main(String[] args) {
        try {
            for (int i = 0; i < 5; i++) {
                if (i == 3)
//                    break;
                    throw new RuntimeException();
                System.out.println(i);
            }
        } catch (Exception e) {
            // 什么都不做，只是退出循环
        }
        System.out.println("这是循环外的代码");
    }
}
```

```scala
import scala.util.control.Breaks
import scala.util.control.Breaks._

object Test06_Break {
  def main(args: Array[String]): Unit = {
    // 1. 采用抛出异常的方式，退出循环
    try {
      for (i <- 0 until 5) {
        if (i == 3)
          throw new RuntimeException
        println(i)
      }
    } catch {
      case e: Exception => // 什么都不做，只是退出循环
    }

    // 2. 使用Scala中的Breaks类的break方法，实现异常的抛出和捕捉
    Breaks.breakable(
      for (i <- 0 until 5) {
        if (i == 3)
          Breaks.break()
        println(i)
      }
    )

    // import scala.util.control.Breaks._
    breakable(
      for (i <- 0 until 5) {
        if (i == 3)
          break()
        println(i)
      }
    )

    println("这是循环外的代码")
  }
}
```

> Scala 内置控制结构特地去掉了 Java 中的 break 和 continue 关键字，是为了更好的适应函数式编程，推荐使用函数式的风格实现 break 和 continue 的功能，而不是一个关键字。Scala 中使用 breakable
> 控制结构来实现 break 和 continue 功能。

##### 函数式编程

> 1）面向对象编程
> 解决问题，分解对象，行为，属性，然后通过对象的关系以及行为的调用来解决问题。
> 对象：用户
> 行为：登录 、连接 JDBC 、读取数据库
> 属性：用户名、密码
>
> Scala 语言是一个**完全面向对象编程**语言，万物皆对象。
> 对象的本质：对数据和行为的一个封装
>
> 2）函数式编程
> 解决问题时，将问题分解成一个一个的步骤，将每个步骤进行封装（函数），通过调用
> 这些封装好的步骤，解决问题。
> 例如：请求 -> 用户名、密码 -> 连接 JDBC -> 读取数据库
>
> Scala 语言是一个**完全函数式编程**语言，万物皆函数。
> 函数的本质：函数可以当做一个值进行传递
>
> 3）在 Scala 中函数式编程和面向对象编程完美融合在一起了。

###### Test01_FunctionAndMethod.scala

> 【函数和方法的区别】
>
> （1）为完成某一功能的程序语句的集合，称为函数
> （2）类中的函数称之方法

```scala
object Test01_FunctionAndMethod {
  def main(args: Array[String]): Unit = {
    // 定义函数
    def sayHi(name: String): Unit = {
      println("hi, " + name)
    }

    // 调用函数
    sayHi("alice")	// hi, alice

    // 调用对象方法
    Test01_FunctionAndMethod.sayHi("bob")	// Hi, bob

    // 获取方法返回值
    val result = Test01_FunctionAndMethod.sayHello("cary")
    println(result)
  }

  // 定义对象的方法
  def sayHi(name: String): Unit = {
    println("Hi, " + name)
  }

  def sayHello(name: String): String = {
    println("Hello, " + name)
    return "Hello"
  }
}
```

> ```scala
> hi, alice
> Hi, bob
> Hello, cary
> Hello
> ```
>
> （1）Scala 语言可以在任何的语法结构中声明任何的语法
> （2）函数没有重载和重写的概念；方法可以进行重载和重写
> （3）Scala 中函数可以嵌套定义

###### Test02_FunctionDefine.scala

> 【函数定义】
>
> （1）函数 1 ：无参，无返回值
>
> （2）函数 2 ：无参，有返回值
>
> （3）函数 3 ：有参，无返回值
>
> （4）函数 4 ：有参，有返回值
>
> （5）函数 5 ：多参，无返回值
>
> （6）函数 6 ：多参，有返回值

```scala
object Test02_FunctionDefine {
  def main(args: Array[String]): Unit = {
    //    （1）函数1：无参，无返回值
    def f1(): Unit = {
      println("1. 无参，无返回值")
    }

    f1()
    println(f1())

    println("=========================")

    //    （2）函数2：无参，有返回值
    def f2(): Int = {
      println("2. 无参，有返回值")
      return 12
    }

    println(f2())

    println("=========================")

    //    （3）函数3：有参，无返回值
    def f3(name: String): Unit = {
      println("3：有参，无返回值 " + name)
    }

    println(f3("alice"))

    println("=========================")

    //    （4）函数4：有参，有返回值
    def f4(name: String): String = {
      println("4：有参，有返回值 " + name)
      return "hi, " + name
    }

    println(f4("alice"))

    println("=========================")

    //    （5）函数5：多参，无返回值
    def f5(name1: String, name2: String): Unit = {
      println("5：多参，无返回值")
      println(s"${name1}和${name2}都是我的好朋友")
    }

    f5("alice", "bob")

    println("=========================")

    //    （6）函数6：多参，有返回值
    def f6(a: Int, b: Int): Int = {
      println("6：多参，有返回值")
      return a + b
    }

    println(f6(12, 37))
  }
}
```

> ```scala
> 1. 无参，无返回值
> 1. 无参，无返回值
> ()
> =========================
> 2. 无参，有返回值
> 12
> =========================
> 3：有参，无返回值 alice
> ()
> =========================
> 4：有参，有返回值 alice
> hi, alice
> =========================
> 5：多参，无返回值
> alice和bob都是我的好朋友
> =========================
> 6：多参，有返回值
> 49
> ```

###### Test03_FunctionParameter.scala

> 【函数参数】
>
> （1）可变参数
> （2）如果参数列表中存在多个参数，那么可变参数一般放置在最后
> （3）参数默认值，一般将有默认值的参数放置在参数列表的后面
> （4）带名参数

```scala
object Test03_FunctionParameter {
  def main(args: Array[String]): Unit = {
    //    （1）可变参数
    def f1(str: String*): Unit = {
      println(str)
    }

    f1("alice")
    f1("aaa", "bbb", "ccc")

    //    （2）如果参数列表中存在多个参数，那么可变参数一般放置在最后
    def f2(str1: String, str2: String*): Unit = {
      println("str1: " + str1 + " str2: " + str2)
    }

    f2("alice")
    f2("aaa", "bbb", "ccc")

    //    （3）参数默认值，一般将有默认值的参数放置在参数列表的后面
    def f3(name: String = "CQUPT"): Unit = {
      println("My school is " + name)
    }

    f3("school")
    f3()

    //    （4）带名参数
    def f4(name: String = "ahua", age: Int): Unit = {
      println(s"${age}岁的${name}学习Scala")
    }

    f4("alice", 20)
    f4(age = 21, name = "bob")
    f4(age = 23)
  }
}
```

> ```scala
> WrappedArray(alice)
> WrappedArray(aaa, bbb, ccc)
> str1: alice str2: WrappedArray()
> str1: aaa str2: WrappedArray(bbb, ccc)
> My school is school
> My school is CQUPT
> 20岁的alice学习Scala
> 21岁的bob学习Scala
> 23岁的ahua学习Scala
> ```

###### Test04_Simplify.scala

> 【函数至简原则（重点）】
>
> 函数至简原则：能省则省
> （1）return 可以省略， Scala 会使用函数体的最后一行代码作为返回值
> （2）如果函数体只有一行代码，可以省略花括号
> （3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略）
> （4）如果有 return ，则不能省略返回值类型，必须指定
> （5）如果函数明确声明 unit ，那么即使函数体中使用 return 关键字也不起作用
> （6）Scala 如果期望是无返回值类型，可以省略等号
> （7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加
> （8）如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略
> （9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略

```scala
// 函数至简原则
object Test04_Simplify {
  def main(args: Array[String]): Unit = {

    def f0(name: String): String = {
      return name
    }

    println(f0("ahua"))

    println("==========================")

    //    （1）return可以省略，Scala会使用函数体的最后一行代码作为返回值
    def f1(name: String): String = {
      name
    }

    println(f1("ahua"))

    println("==========================")

    //    （2）如果函数体只有一行代码，可以省略花括号
    def f2(name: String): String = name

    println(f2("ahua"))

    println("==========================")

    //    （3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略）
    def f3(name: String) = name

    println(f3("ahua"))

    println("==========================")

    //    （4）如果有return，则不能省略返回值类型，必须指定
    //    def f4(name: String) = {
    //      return name
    //    }
    //
    //    println(f4("ahua"))

    println("==========================")

    //    （5）如果函数明确声明unit，那么即使函数体中使用return关键字也不起作用
    def f5(name: String): Unit = {
      return name
    }

    println(f5("ahua"))	// ()

    println("==========================")

    //    （6）Scala如果期望是无返回值类型，可以省略等号
    def f6(name: String) {
      println(name)
    }

    println(f6("ahua"))
    // 输出
    // ahua
    // ()

    println("==========================")

    //    （7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加
    def f7(): Unit = {
      println("ahua")
    }

    f7()
    f7
    // 输出
    // ahua
    // ahua

    println("==========================")

    //    （8）如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略
    def f8: Unit = {
      println("atguigu")
    }

    //    f8()	// error
    f8

    println("==========================")

    //    （9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略
    def f9(name: String): Unit = {
      println(name)
    }

    // 匿名函数，lambda表达式
    (name: String) => {
      println(name)
    }

      println("==========================")

  }
}
```

###### Test05_Lambda.scala

> 【匿名函数】

```scala
object Test05_Lambda {
  def main(args: Array[String]): Unit = {
    val fun = (name: String) => {
      println(name)
    }
    fun("ahua")

    println("========================")

    // 定义一个函数，以函数作为参数输入
    def f(func: String => Unit): Unit = {
      func("ahua")
    }

    f(fun)
    f((name: String) => {
      println(name)
    })

    println("========================")

    // 匿名函数的简化原则
    //    （1）参数的类型可以省略，会根据形参进行自动的推导
    f((name) => {
      println(name)
    })

    //    （2）类型省略之后，发现只有一个参数，则圆括号可以省略；其他情况：没有参数和参数超过1的永远不能省略圆括号
    f(name => {
      println(name)
    })

    //    （3）匿名函数如果只有一行，则大括号也可以省略
    f(name => println(name))

    //    （4）如果参数只出现一次，则参数省略且后面参数可以用_代替
    f(println(_))

    //     (5) 如果可以推断出，当前传入的println是一个函数体，而不是调用语句，可以直接省略下划线
    f(println)

    println("=========================")

    // 实际示例，定义一个“二元运算”函数，只操作1和2两个数，但是具体运算通过参数传入
    def dualFunctionOneAndTwo(fun: (Int, Int) => Int): Int = {
      fun(1, 2)
    }

    val add = (a: Int, b: Int) => a + b
    val minus = (a: Int, b: Int) => a - b

    println(dualFunctionOneAndTwo(add))
    println(dualFunctionOneAndTwo(minus))

    // 匿名函数简化
    println(dualFunctionOneAndTwo((a: Int, b: Int) => a + b))
    println(dualFunctionOneAndTwo((a: Int, b: Int) => a - b))

    println(dualFunctionOneAndTwo((a, b) => a + b))
    println(dualFunctionOneAndTwo(_ + _))
    println(dualFunctionOneAndTwo(_ - _))

    println(dualFunctionOneAndTwo((a, b) => b - a))
    println(dualFunctionOneAndTwo(-_ + _))
  }
}
```

###### Test06_HighOrderFunction.scala

> 【高阶函数】
>
> （1）函数可以作为值进行传递
>
> （1）函数可以作为参数进行传递
>
> （3）函数可以作为函数返回值返回

```scala
object Test06_HighOrderFunction {
  def main(args: Array[String]): Unit = {
    def f(n: Int): Int = {
      println("f调用")
      n + 1
    }

    def fun(): Int = {
      println("fun调用")
      1
    }

    val result: Int = f(123)
    println(result)

    println("=========================")

    // 1. 函数作为值进行传递
    val f1: Int => Int = f
    val f2 = f _

    println(f1)
    println(f1(12))
    println(f2)
    println(f2(35))
    println("-------------------------")

    val f3: () => Int = fun	// val f3 = fun	println(f3)	// 输出 1
    val f4 = fun _
    println(f3)
    println(f4)

    println("=========================")

    // 2. 函数作为参数进行传递
    // 定义二元计算函数
    def dualEval(op: (Int, Int) => Int, a: Int, b: Int): Int = {
      op(a, b)
    }

    def add(a: Int, b: Int): Int = {
      a + b
    }

    println(dualEval(add, 12, 35))
    println(dualEval((a, b) => a + b, 12, 35))
    println(dualEval(_ + _, 12, 35))

	println("=========================")

    // 3. 函数作为函数的返回值返回
    def f5(): Int => Unit = {
      def f6(a: Int): Unit = {
        println("f6调用 " + a)
      }

      f6 // 将函数直接返回
    }

    //    val f6 = f5()
    //    println(f6)
    //    println(f6(25))

    println(f5()(25))
  }
}
```

> ```scala
> f调用
> 124
> =========================
> chapter05.Test06_HighOrderFunction$$$Lambda$5/2096171631@6debcae2
> f调用
> 13
> chapter05.Test06_HighOrderFunction$$$Lambda$6/2114694065@5ba23b66
> f调用
> 36
> -------------------------
> chapter05.Test06_HighOrderFunction$$$Lambda$7/2101440631@35bbe5e8
> chapter05.Test06_HighOrderFunction$$$Lambda$8/2109957412@2c8d66b2
> =========================
> 47
> 47
> 47
> =========================
> f6调用 25
> ()
> ```

###### Test07_Practice_CollectionOperation.scala

> 【练习】

```scala
object Test07_Practice_CollectionOperation {
  def main(args: Array[String]): Unit = {
    val arr: Array[Int] = Array(12, 45, 75, 98)

    // 对数组进行处理，将操作抽象出来，处理完毕之后的结果返回一个新的数组
    def arrayOperation(array: Array[Int], op: Int => Int): Array[Int] = {
      for (elem <- array) yield op(elem)
    }

    // 定义一个加一操作
    def addOne(elem: Int): Int = {
      elem + 1
    }

    // 调用函数
    val newArray: Array[Int] = arrayOperation(arr, addOne)

    println(newArray.mkString(", "))

    // 传入匿名函数，实现元素翻倍
    val newArray2 = arrayOperation(arr, _ * 2)
    println(newArray2.mkString(", "))
  }
}
```

> ```
> 13,46,76,99
> 24,90,150,196
> ```

###### Test08_Practice.scala

> 【练习】
>
> 练习1：
>
> 定义一个匿名函数，并将它作为值赋给变量 fun 。
>
> 函数有三个参数，类型分别为Int，String，Char；返回值类型为Boolean。
>
> 要求调用函数fun(0, “”, ‘0’)得到返回值为false，其它情况均返回true。
>
> 练习2：
>
> 定义一个函数func，它接收一个Int类型的参数，返回一个函数（记作f1）。
>
> 它返回的函数f1，接收一个String类型的参数，同样返回一个函数（记作f2）。
>
> 函数f2接收一个Char类型的参数，返回一个Boolean的值。
>
> 要求调用函数func(0) (“”) (‘0’)得到返回值为false，其它情况均返回true。

```scala
object Test08_Practice {
  def main(args: Array[String]): Unit = {
    // 1. 练习1
    val fun = (i: Int, s: String, c: Char) => {
      if (i == 0 && s == "" && c == '0') false else true
    }

    println(fun(0, "", '0'))
    println(fun(0, "", '1'))
    println(fun(23, "", '0'))
    println(fun(0, "hello", '0'))

    println("===========================")

    // 2. 练习2
    def func(i: Int): String => (Char => Boolean) = {
      def f1(s: String): Char => Boolean = {
        def f2(c: Char): Boolean = {
          if (i == 0 && s == "" && c == '0') false else true
        }

        f2
      }

      f1
    }

    println(func(0)("")('0'))
    println(func(0)("")('1'))
    println(func(23)("")('0'))
    println(func(0)("hello")('0'))

    // 匿名函数简写
    def func1(i: Int): String => (Char => Boolean) = {
      s => c => if (i == 0 && s == "" && c == '0') false else true
    }

    println(func1(0)("")('0'))
    println(func1(0)("")('1'))
    println(func1(23)("")('0'))
    println(func1(0)("hello")('0'))

    // 柯里化
    def func2(i: Int)(s: String)(c: Char): Boolean = {
      if (i == 0 && s == "" && c == '0') false else true
    }

    println(func2(0)("")('0'))
    println(func2(0)("")('1'))
    println(func2(23)("")('0'))
    println(func2(0)("hello")('0'))
  }
}
```

> ```scala
> false
> true
> true
> true
> ===========================
> false
> true
> true
> true
> false
> true
> true
> true
> false
> true
> true
> true
> ```

###### Test09_ClosureAndCurrying.scala

> 【函数柯里化&闭包】
>
> 闭包：函数式编程的标配
>
> 闭包：如果一个函数，访问到了它的外部（局部）变量的值，那么这个函数和他所处的环境，称为闭包
>
> 函数柯里化：把一个参数列表的多个参数，变成多个参数列表

```scala
object Test09_ClosureAndCurrying {
  def main(args: Array[String]): Unit = {
    def add(a: Int, b: Int): Int = {
      a + b
    }

    // 1. 考虑固定一个加数的场景
    def addByFour(b: Int): Int = {
      4 + b
    }

    // 2. 扩展固定加数改变的情况
    def addByFive(b: Int): Int = {
      5 + b
    }

    // 3. 将固定加数作为另一个参数传入，但是是作为“第一层参数”传入
    def addByFour1(): Int => Int = {
      val a = 4

      def addB(b: Int): Int = {
        a + b
      }

      addB
    }

    def addByA(a: Int): Int => Int = {
      def addB(b: Int): Int = {
        a + b
      }

      addB
    }

    println(addByA(35)(24))

    val addByFour2 = addByA(4)
    val addByFive2 = addByA(5)

    println(addByFour2(13))
    println(addByFive2(25))

    // 4. lambda表达式简写
    def addByA1(a: Int): Int => Int = {
      (b: Int) => {
        a + b
      }
    }

    def addByA2(a: Int): Int => Int = {
      b => a + b
    }

    def addByA3(a: Int): Int => Int = a + _

    val addByFour3 = addByA3(4)
    val addByFive3 = addByA3(5)

    println(addByFour3(13))
    println(addByFive3(25))

    // 5. 柯里化
    def addCurrying(a: Int)(b: Int): Int = {
      a + b
    }

    println(addCurrying(35)(24))
  }
}
```

###### Test10_Recursion.scala

> 【递归】阶乘、尾递归
>
> 一个函数/方法在函数/方法体内又调用了本身，我们称之为递归调用
>
> 递归算法
>
> （1）方法调用自身
>
> （2）方法必须要有跳出的逻辑
>
> （3）方法调用自身时，传递的参数应该有规律
>
> （4）scala 中的递归必须声明函数返回值类型

TestRecursion.java

```java
public class TestRecursion {
    public static void main(String[] args) {
        // 计算阶乘
        System.out.println(factorial(5));
        System.out.println(fact(5));
    }

    // 1. 循环实现
    public static int factorial(int n) {
        int result = 1;
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    // 2. 递归实现
    public static int fact(int n) {
        // 基准情形 0! = 1
        if (n == 0) return 1;
        return fact(n - 1) * n;
    }
}
```

```scala
import scala.annotation.tailrec

object Test10_Recursion {
  def main(args: Array[String]): Unit = {
    println(fact(5))
    println(tailFact(5))
  }

  // 递归实现计算阶乘
  def fact(n: Int): Int = {
    if (n == 0) return 1
    fact(n - 1) * n
  }

  // 尾递归实现
  def tailFact(n: Int): Int = {
    @tailrec
    def loop(n: Int, currRes: Int): Int = {
      if (n == 0) return currRes
      loop(n - 1, currRes * n)
    }

    loop(n, 1)
  }
}
```

###### Test11_ControlAbstraction.scala

> 【控制抽象】传值参数、传名参数
>
> 值调用： 把计算后的值传递过去
>
> 名调用： 把代码传递过去

```scala
object Test11_ControlAbstraction {
  def main(args: Array[String]): Unit = {
    // 1. 传值参数
    def f0(a: Int): Unit = {
      println("a: " + a)
      println("a: " + a)
    }

    f0(23)

    def f1(): Int = {
      println("f1调用")
      12
    }

    f0(f1())

    println("========================")

    // 2. 传名参数，传递的不再是具体的值，而是代码块
    def f2(a: => Int): Unit = {
      println("a: " + a)
      println("a: " + a)
    }

    f2(23)
    f2(f1())

    f2({
      println("这是一个代码块")
      29
    })

  }
}
```

> ```scala
> a: 23
> a: 23
> f1调用
> a: 12
> a: 12
> ========================
> a: 23
> a: 23
> f1调用
> a: 12
> f1调用
> a: 12
> 这是一个代码块
> a: 29
> 这是一个代码块
> a: 29
> ```
>
> 是不是将 a: => Int 认为是返回值为Int类型的一段代码，在函数体中a出现一次，这段代码执行一次。
>
> 而函数也是相应的一段代码，可以直接传入，如：f2(f1())。
>
> 其实，再从数学的角度考虑，将代码块视为一个多项式。这个名传递，不就是将一个多项式（可以计算出结果）整体带入到另一个式子里计算吗？并且没出现一次，都要重新计算一次这个多项式；而对于值传递，就是先计算出这个多项式的准确数值，然后再把此结果带入另一个式子里计算。这样的话，这个多项式其实只计算一次哎。
>
> 那么名传递岂不是要多进行一些计算吗？
>
> 比如：
>
> 值传递的结果
>
> ```
> f1调用
> a: 12
> a: 12
> ```
>
> 名传递的结果
>
> ```
> f1调用
> a: 12
> f1调用
> a: 12
> ```
>
> 很明显名传递多执行了一部分代码啊。

###### Test12_MyWhile.scala

> 【自定义一个 While 循环】

```scala
object Test12_MyWhile {
  def main(args: Array[String]): Unit = {
    var n = 10

    // 1. 常规的while循环
    while (n >= 1) {
      println(n)
      n -= 1
    }

    // 2. 用闭包实现一个函数，将代码块作为参数传入，递归调用
    def myWhile(condition: => Boolean): (=> Unit) => Unit = {
      // 内层函数需要递归调用，参数就是循环体
      def doLoop(op: => Unit): Unit = {
        if (condition) {
          op
          myWhile(condition)(op)
        }
      }

      doLoop _
    }

    println("=================")
    n = 10
    myWhile(n >= 1) {
      println(n)
      n -= 1
    }

    // 3. 用匿名函数实现
    def myWhile2(condition: => Boolean): (=> Unit) => Unit = {
      // 内层函数需要递归调用，参数就是循环体
      op => {
        if (condition) {
          op
          myWhile2(condition)(op)
        }
      }
    }

    println("=================")
    n = 10
    myWhile2(n >= 1) {
      println(n)
      n -= 1
    }

    // 3. 用柯里化实现
    def myWhile3(condition: => Boolean)(op: => Unit): Unit = {
      if (condition) {
        op
        myWhile3(condition)(op)
      }
    }

    println("=================")
    n = 10
    myWhile3(n >= 1) {
      println(n)
      n -= 1
    }
  }
}
```

###### Test13_Lazy.scala

> 【惰性加载】
>
> 当函数返回值被声明为 lazy 时 ，函数的执行将被推迟 ，直到我们首次对此取值，该函数才会执行。这种函数我们称之为惰性函数。
>
> 注意：lazy 不能修饰 var 类型的变量。

```scala
object Test13_Lazy {
  def main(args: Array[String]): Unit = {
    lazy val result: Int = sum(13, 47)

    println("1. 函数调用")
    println("2. result = " + result)
    println("4. result = " + result)
  }

  def sum(a: Int, b: Int): Int = {
    println("3. sum调用")
    a + b
  }
}
```

> ```scala
> 1. 函数调用
> 3. sum调用
> 2. result = 60
> 4. result = 60
> ```

##### 面向对象

> Scala 的面向对象思想和 Java 的面向对象思想和概念是一致的。
> Scala 中语法和 Java 不同 ，补充了更多的功能。

###### Test01_Package.scala

> 【包语句】
>
> （1）一个源文件中可以声明多个 package
> （2）子包中的类可以直接访问父包中的内容，而无需导包

```scala
// 用嵌套风格定义包
package com {

  import com.ahua.scala.Inner

  // 在外层包中定义单例对象
  object Outer {
    var out: String = "out"

    // 外层访问内层要导内层包
    def main(args: Array[String]): Unit = {
      println(Inner.in)	// in
    }
  }
  package ahua {
    package scala {

      // 内层包中定义单例对象
      object Inner {
        var in: String = "in"

        // 内层访问外层不需要导包
        def main(args: Array[String]): Unit = {
          println(Outer.out)	// out
          Outer.out = "outer"
          println(Outer.out)	// outer
        }
      }

    }

  }

}

// 在同一文件中定义不同的包
package aaa {
  package bbb {

    object Test01_Package {
      def main(args: Array[String]): Unit = {
        import com.atguigu.scala.Inner
        println(Inner.in)	// in
      }
    }

  }

}
```

###### package

> 【包对象】
>
> 在 Scala 中可以为每个包定义一个同名的包对象，定义在包对象中的成员，作为其对应包下所有 class 和 object 的共享变量，可以被直接访问。

```scala
package object chapter06 {
  // 定义当前包共享的属性和方法
  val commonValue = "Scala"

  def commonMethod() = {
    println(s"我在学习${commonValue}")
  }
}
```

###### Test02_PackageObject.scala

> 【访问包对象】
>
> 如采用嵌套方式管理包，则包对象可与包定义在同一文件中，但是要保证包对象与包声明在同一作用域中（同一层级下）。

```scala
//package chapter06
//
//object Test02_PackageObject {
//  def main(args: Array[String]): Unit = {
//    commonMethod()
//    println(commonValue)
//  }
//}

package chapter06 {

  object Test02_PackageObject {
    def main(args: Array[String]): Unit = {
      commonMethod()
      println(commonValue)
    }
  }

}

package ccc {
  package ddd {

    object Test02_PackageObject {
      def main(args: Array[String]): Unit = {
        println(school)
      }
    }

  }

}

// 定义一个包对象
package object ccc {
  val school: String = "CQUPT"
}
```

> 包 ccc 与包对象 ccc 要在同一层级下，才能访问 school
>
> 若把
>
> ```scala
> package object ccc {
>   val school: String = "CQUPT"
> }
> ```
>
> 改为
>
> ```scala
> package object ddd {
>   val school: String = "CQUPT"
> }
> ```
>
> 由于package ddd 与 package object ddd 不在同一层级下，那么在 main 中就不能访问到 school 了。
>
> 如果改成下面这种就又可以了：
>
> ```scala
> package ccc {
>   package ddd {
> 
>     object Test02_PackageObject {
>       def main(args: Array[String]): Unit = {
>         println(school)
>       }
>     }
> 
>   }
> 
>   // 定义一个包对象
>   package object ddd {
>     val school: String = "CQUPT"
>   }
> 
> }
> ```

###### Test03_Class.scala

> 【类和对象】
>
> 类：可以看成一个模板
> 对象：表示具体的事物
>
> 回顾：Java 中的类
> 如果类是 public 的，则必须和文件名一致
> 一般，一个 .java 有一个 public 类
> 注意：Scala 中没有 public，一个 .scala 中可以写多个类 
> 基本语法
> [修饰符 ] class 类名{
> 		类体
> }
> 说明
> （1）Scala 语法中，类并不声明为 public ，所有这些类都具有公有可见性（即默认就是 public）
> （2）一个 Scala 源文件可以包含多个类

```scala
import scala.beans.BeanProperty

object Test03_Class {
  def main(args: Array[String]): Unit = {
    // 创建一个对象
    val student = new Student()
    //    student.name   // error, 不能访问private属性
    println(student.age)
    println(student.sex)
    student.sex = "female"
    println(student.sex)
  }
}

// 定义一个类
class Student {
  // 定义属性
  private var name: String = "alice"
  @BeanProperty
  var age: Int = _
  var sex: String = _
}
```

> ```scala
> 0
> null
> female
> ```

###### Test04_Access.scala

> 【访问权限】
>
> 在 Java 中，访问权限分为：public、private、protected 和默认。在 Scala 中，则可以通过类似的修饰符达到同样的效果。但是使用上有区别。
> （1） Scala 中属性和方法的默认访问权限为 public，但 Scala 中无 public 关键字。
> （2） private 为私有权限，只在类的内部和伴生对象中可用。
> （3） protected 为受保护权限， Scala 中受保护权限比 Java 中更严格，同类、子类可以访问，同包无法访问。
> （4） private[包名] 增加包访问权限，包名下的其他类也可以使用

```scala
object Test04_Access {
  def main(args: Array[String]): Unit = {
    // 创建对象
    val person: Person = new Person()
    //    person.idCard    // error	(2)
    //    person.name    // error	(3)
    println(person.age)	//	(4)
    println(person.sex)

    person.printInfo()

    val worker: Worker = new Worker()
    //    worker = new Worker()
    //    worker.age = 23
    worker.printInfo()
  }
}

// 定义一个子类
class Worker extends Person {
  override def printInfo(): Unit = {
    //    println(idCard)    // error
    name = "bob"
    age = 25
    sex = "male"

    println(s"Worker: $name $sex $age")
  }
}
```

Test04_ClassForAccess.scala

```scala
object Test04_ClassForAccess {

}

// 定义一个父类
class Person {
  private var idCard: String = "3523566"
  protected var name: String = "alice"
  var sex: String = "female"
  private[chapter06] var age: Int = 18

  def printInfo(): Unit = {
    println(s"Person: $idCard $name $sex $age")
  }
}
```

> ```
> 18
> female
> Person: 3523566 alice female 18
> Worker: bob male 25
> ```

###### Test05_Constructor.scala

> 【构造器】主构造器、辅助构造器
>
> （1）辅助构造器函数的名称 this，可以有多个 ，编译器通过参数的个数及类型来区分。
> （2）辅助构造方法不能直接构建对象 ，必须直接或者间接调用主构造方法。
> （3）构造器调用其他另外的构造器， 要求被调用构造器必须提前声明。

```scala
object Test05_Constructor {
  def main(args: Array[String]): Unit = {
    val student1 = new Student1
    student1.Student1()
    
    println("========================")

    val student2 = new Student1("alice")
    
    println("========================")

    val student3 = new Student1("bob", 25)
  }
}

// 定义一个类
class Student1() {// 主构造器
  // 定义属性
  var name: String = _
  var age: Int = _

  println("1. 主构造方法被调用")

  // 声明辅助构造方法
  def this(name: String) {
    this() // 直接调用主构造器
    println("2. 辅助构造方法一被调用")
    this.name = name
    println(s"name: $name age: $age")
  }

  def this(name: String, age: Int) {
    this(name)
    println("3. 辅助构造方法二被调用")
    this.age = age
    println(s"name: $name age: $age")
  }

  def Student1(): Unit = {
    println("一般方法被调用")
  }
}
```

> ```scala
> 1. 主构造方法被调用
> 一般方法被调用
> ========================
> 1. 主构造方法被调用
> 2. 辅助构造方法一被调用
> name: alice age: 0
> ========================
> 1. 主构造方法被调用
> 2. 辅助构造方法一被调用
> name: bob age: 0
> 3. 辅助构造方法二被调用
> name: bob age: 25
> ```

###### Test06_ConstructorParams.scala

> 【构造器参数】
>
> Scala 类的主构造器函数的形参包括三种类型：未用任何修饰、var 修饰、val 修饰
> （1）未用任何修饰符修饰，这个参数就是一个局部变量
> （2）var 修饰参数，作为类的成员属性使用，可以修改
> （3）val 修饰参数，作为类只读属性使用，不能修改

```scala
object Test06_ConstructorParams {
  def main(args: Array[String]): Unit = {
    val student2 = new Student2
    student2.name = "alice"
    student2.age = 18
    println(s"student2: name = ${student2.name}, age = ${student2.age}")

    val student3 = new Student3("bob", 20)
    println(s"student3: name = ${student3.name}, age = ${student3.age}")

    val student4 = new Student4("cary", 25)
    //    println(s"student4: name = ${student4.name}, age = ${student4.age}")
    student4.printInfo()

    val student5 = new Student5("bob", 20)
    println(s"student5: name = ${student5.name}, age = ${student5.age}")

    //    student5.age = 21	// student5的name age赋值后就不能再更改了

    val student6 = new Student6("cary", 25, "CQUPT")
    println(s"student6: name = ${student6.name}, age = ${student6.age}")
    student6.printInfo()
  }
}

// 定义类
// 无参构造器
class Student2 {
  // 单独定义属性
  var name: String = _
  var age: Int = _
}

// 上面定义等价于
class Student3(var name: String, var age: Int)

// 主构造器参数无修饰
class Student4(name: String, age: Int) {
  def printInfo() {
    println(s"student4: name = ${name}, age = $age")
  }
}

//class Student4(_name: String, _age: Int){
//  var name: String = _name
//  var age: Int = _age
//}

class Student5(val name: String, val age: Int)

class Student6(var name: String, var age: Int) {
  var school: String = _

  def this(name: String, age: Int, school: String) {
    this(name, age)
    this.school = school
  }

  def printInfo() {
    println(s"student6: name = ${name}, age = $age, school = $school")
  }
}
```

> ```scala
> student2: name = alice, age = 18
> student3: name = bob, age = 20
> student4: name = cary, age = 25
> student3: name = bob, age = 20
> student6: name = cary, age = 25
> student6: name = cary, age = 25, school = CQUPT
> ```

###### Test07_Inherit.scala

> 【继承和多态】
>
> 基本语法
>
> class 子类名 extends 父类名 { 类体 }
>
> （1）子类继承父类的 属性 和 方法
> （2） scala 是单继承
> （3）继承的调用顺序：父类构造器->子类构造器

```scala
object Test07_Inherit {
  def main(args: Array[String]): Unit = {
      
    val student1: Student7 = new Student7("alice", 18)

    println("=========================")

    val student2 = new Student7("bob", 20, "std001")

    student1.printInfo()
    student2.printInfo()

    val teacher = new Teacher
    teacher.printInfo()

    def personInfo(person: Person7): Unit = {
      person.printInfo()
    }

    println("=========================")

    val person = new Person7
    personInfo(student1)
    personInfo(teacher)
    personInfo(person)
  }
}

// 定义一个父类
class Person7() {
  var name: String = _
  var age: Int = _

  println("1. 父类的主构造器调用")

  def this(name: String, age: Int) {
    this()
    println("2. 父类的辅助构造器调用")
    this.name = name
    this.age = age
  }

  def printInfo(): Unit = {
    println(s"Person: $name $age")
  }
}

// 定义子类
class Student7(name: String, age: Int) extends Person7(name, age) {
  var stdNo: String = _

  println("3. 子类的主构造器调用")

  def this(name: String, age: Int, stdNo: String) {
    this(name, age)
    println("4. 子类的辅助构造器调用")
    this.stdNo = stdNo
  }

  override def printInfo(): Unit = {
    println(s"Student: $name $age $stdNo")
  }
}

class Teacher extends Person7 {
  override def printInfo(): Unit = {
    println(s"Teacher")
  }
}
```

###### Test08_DynamicBind.scala

> 【多态】动态绑定、静态绑定
>
> 在Java中，属性是静态绑定的，方法是动态绑定的
>
> 在Scala中，属性和方法都是动态绑定的

TestDynamicBind.java

```java
public class TestDynamicBind {
    public static void main(String[] args) {
        Worker worker = new Worker();
        System.out.println(worker.name);
        worker.hello();
        worker.hi();

        System.out.println("=========================");

        // 多态
        Person person = new Worker();
        System.out.println(person.name);    // 静态绑定属性
        person.hello();    // 动态绑定方法
//        person.hi();     // error
    }
}

class Person {
    String name = "person";

    public void hello() {
        System.out.println("hello person");
    }
}

class Worker extends Person {
    String name = "worker";

    public void hello() {
        System.out.println("hello worker");
    }

    public void hi() {
        System.out.println("hi worker");
    }
}
```

> ```java
> worker
> hello worker
> hi worker
> =========================
> person
> hello worker
> ```
>
> person.name仍然输出是person，而person.hello(); 调用的则是子类的hello方法

```scala
object Test08_DynamicBind {
  def main(args: Array[String]): Unit = {
    val student: Person8 = new Student8
    println(student.name)
    student.hello()
  }
}

class Person8 {
  val name: String = "person"

  def hello(): Unit = {
    println("hello person")
  }
}

class Student8 extends Person8 {
  override val name: String = "student"

  override def hello(): Unit = {
    println("hello student")
  }
}
```

> ```scala
> student
> hello student
> ```

###### Test09_AbstractClass.scala

> 【抽象类】抽象属性、抽象方法
>
> 1）基本语法
>
> （1）定义抽象类：abstract class Person{}	//通过 abstract 关键字标记抽象类
> （2）定义抽象属性：val/var name:String	//一个属性没有初始化，就是抽象属性
> （3）定义抽象方法：def hello():String	//只声明而没有实现的方法，就是抽象方法
>
> 2）继承&重写
> （1）如果父类为抽象类，那么子类需要将抽象的属性和方法实现，否则子类也需声明为抽象类
> （2）重写非抽象方法需要用 override 修饰，重写抽象方法则可以不加 override
> （3）子类中调用父类的方法使用 super 关键字
> （4）子类对**抽象属性**进行实现，父类抽象属性可以用 var 修饰；
> 子类对**非抽象属性**重写 ，父类非抽象属性只支持 val 类型，而不支持 var。
> 因为 var 修饰的为可变变量，子类继承之后就可以直接使用，没有必要重写。

```scala
object Test09_AbstractClass {
  def main(args: Array[String]): Unit = {
    val student = new Student9
    student.eat()
    student.sleep()
  }
}

// 定义抽象类
abstract class Person9 {
  // 非抽象属性
  var name: String = "person"

  // 抽象属性
  var age: Int

  // 非抽象方法
  def eat(): Unit = {
    println("person eat")
  }

  // 抽象方法
  def sleep(): Unit
}

// 定义具体的实现子类
class Student9 extends Person9 {
  // 实现抽象属性和方法
  var age: Int = 18

  def sleep(): Unit = {
    println("student sleep")
  }

  // 重写非抽象属性和方法
  //  override val name: String = "student"
  name = "student"

  override def eat(): Unit = {
    super.eat()
    println("student eat")
  }
}
```

> ```scala
> person eat
> student eat
> student sleep
> ```
>
> 由于在抽象父类中`var name: String = "person"`，name被定义为var，则在子类中只需要重新赋值就行了，而不需要再去重写name；而如果抽象父类中name是定义为val，则在子类中就可重写name，`override val name: String = "student"`，就是正确的。

###### Test10_AnonymousClass.scala

> 【匿名子类】

```scala
object Test10_AnonymousClass {
  def main(args: Array[String]): Unit = {
    val person: Person10 = new Person10 {
      override var name: String = "alice"

      override def eat(): Unit = println("person eat")
    }
    println(person.name)
    person.eat()
  }
}

// 定义抽象类
abstract class Person10 {
  var name: String

  def eat(): Unit
}
```

> ```scala
> alice
> person eat
> ```
>
> 没看懂，和匿名子类有什么关系。
>
> 就是说，没有特意去实现一个子类，再创建一个实例，调用这个方法，也能够得到相同结果吗？

###### Test11_Object.scala

> 【单例对象（伴生对象）】单例对象、apply方法
>
> Scala语言是完全面向对象的语言，所以并没有静态的操作（即在Scala中没有静态的概念）。但是为了能够和Java语言交互（因为Java中有静态概念），就产生了一种特殊的对象来模拟类对象，该对象为单例对象。若单例对象名与类名一致，则称该单例对象为这个类的伴生对象，这个类的所有“静态”内容都可以放置在它的伴生对象中声明。
>
> （1）单例对象采用 object 关键字声明
> （2）单例对象对应的类称之为 伴生类 ，伴生对象的名称和伴生类名一致
> （3）单例对象中的属性和方法都可以通过伴生对象名（类名）直接调用访问

```scala
object Test11_Object {
  def main(args: Array[String]): Unit = {
    //    val student = new Student11("alice", 18)
    //    student.printInfo()

    val student1 = Student11.newStudent("alice", 18)
    student1.printInfo()

    val student2 = Student11.apply("bob", 19)
    student2.printInfo()

    val student3 = Student11("bob", 19)
    student3.printInfo()
  }
}

// 定义类
class Student11 private(val name: String, val age: Int) {
  def printInfo() {
    println(s"student: name = ${name}, age = $age, school = ${Student11.school}")
  }
}

// 伴生对象
object Student11 {
  val school: String = "CQUPT"

  // 定义一个类的对象实例的创建方法
  def newStudent(name: String, age: Int): Student11 = new Student11(name, age)

  def apply(name: String, age: Int): Student11 = new Student11(name, age)
}
```

> ```scala
> student: name = alice, age = 18, school = CQUPT
> student: name = bob, age = 19, school = CQUPT
> student: name = bob, age = 19, school = CQUPT
> ```
>
> （1）通过伴生对象的 apply 方法， 实现不使用 new 方法创建对象 。
> （2）如果想让主构造器变成私有的，可以在之前加上 private。
> （3）apply 方法可以重载。
> （4）Scala 中 obj (arg) 的语句实际是在调用该对象的 apply 方法，即 obj.apply(arg) 。用以统一面向对象编程和函数式编程的风格。
> （5）当使用 new 关键字构建对象时，调用的其实是类的构造方法；当直接使用类名构建对象时，调用的其实时伴生对象的 apply 方法。

###### Test12_Singleton.scala

> 【单例模式】

```scala
object Test12_Singleton {
  def main(args: Array[String]): Unit = {
    val student1 = Student12.getInstance()
    student1.printInfo()

    val student2 = Student12.getInstance()
    student2.printInfo()

    // 打印输出引用地址
    println(student1)
    println(student2)
  }
}

class Student12 private(val name: String, val age: Int) {
  def printInfo() {
    println(s"student: name = ${name}, age = $age, school = ${Student11.school}")
  }
}

// 饿汉式
//object Student12 {
//  private val student: Student12 = new Student12("alice", 18)
//  def getInstance(): Student12 = student
//}

// 懒汉式
object Student12 {
  private var student: Student12 = _

  def getInstance(): Student12 = {
    if (student == null) {
      // 如果没有对象实例的话，就创建一个
      student = new Student12("alice", 18)
    }
    student
  }
}
```

> ```scala
> student: name = alice, age = 18, school = CQUPT
> student: name = alice, age = 18, school = CQUPT
> chapter06.Student12@4563e9ab
> chapter06.Student12@4563e9ab
> ```

###### Test13_Trait.scala

> 【特质（Trait）】类似于Java中的接口（Interface）
>
> Scala 语言中，采用特质 trait（特征）来代替接口的概念，也就是说，多个类具有相同的特质（特征）时，就可以将这个特质（特征）独立出来，采用关键字 trait 声明。
> Scala 中的 trait 中即可以有抽象属性和方法，也可以有具体的属性和方法。一个类可以**混入（ mixin ）**多个特质。这种感觉类似于 Java 中的抽象类。
> Scala 引入 trait 特征，第一是可以替代 Java 的接口，第二也是对单继承机制的一种补充。

```scala
object Test13_Trait {
  def main(args: Array[String]): Unit = {
    val student: Student13 = new Student13
    student.sayHello()
    student.study()
    student.dating()
    student.play()
  }
}

// 定义一个父类
class Person13 {
  val name: String = "person"
  var age: Int = 18

  def sayHello(): Unit = {
    println("hello from: " + name)
  }

  def increase(): Unit = {
    println("person increase")
  }
}

// 定义一个特质
trait Young {
  // 声明抽象和非抽象属性
  var age: Int
  val name: String = "young"

  // 声明抽象和非抽象的方法
  def play(): Unit = {
    println(s"young people $name is playing")
  }

  def dating(): Unit
}

class Student13 extends Person13 with Young {
  // 重写冲突的属性
  override val name: String = "student"

  // 实现抽象方法
  def dating(): Unit = println(s"student $name is dating")

  def study(): Unit = println(s"student $name is studying")

  // 重写父类方法
  override def sayHello(): Unit = {
    super.sayHello()
    println(s"hello from: student $name")
  }
}
```

> ```scala
> hello from: student
> hello from: student student
> student student is studying
> student student is dating
> young people student is playing
> ```

###### Test14_TraitMixin.scala

> 【Trait 混入（ mixin ）】

```scala
object Test14_TraitMixin {
  def main(args: Array[String]): Unit = {
    val student = new Student14
    student.study()
    student.increase()

    student.play()
    student.increase()

    student.dating()
    student.increase()

    println("===========================")

    // 动态混入
    val studentWithTalent = new Student14 with Talent {
      override def dancing(): Unit = println("student is good at dancing")

      override def singing(): Unit = println("student is good at singing")
    }

    studentWithTalent.sayHello()
    studentWithTalent.play()
    studentWithTalent.study()
    studentWithTalent.dating()
    studentWithTalent.dancing()
    studentWithTalent.singing()
  }
}

// 再定义一个特质
trait Knowledge {
  var amount: Int = 0

  def increase(): Unit
}

trait Talent {
  def singing(): Unit

  def dancing(): Unit
}

class Student14 extends Person13 with Young with Knowledge {
  // 重写冲突的属性
  override val name: String = "student"

  // 实现抽象方法
  def dating(): Unit = println(s"student $name is dating")

  def study(): Unit = println(s"student $name is studying")

  // 重写父类方法
  override def sayHello(): Unit = {
    super.sayHello()
    println(s"hello from: student $name")
  }

  // 实现特质中的抽象方法
  override def increase(): Unit = {
    amount += 1
    println(s"student $name knowledge increased: $amount")
  }
}
```

> ```scala
> student student is studying
> student student knowledge increased: 1
> young people student is playing
> student student knowledge increased: 2
> student student is dating
> student student knowledge increased: 3
> ===========================
> hello from: student
> hello from: student student
> young people student is playing
> student student is studying
> student student is dating
> student is good at dancing
> student is good at singing
> ```

###### Test15_TraitOverlying.scala

> 【特质叠加/执行顺序】
>
> 由于一个类可以混入（mixin ）多个 trait ，且 trait 中可以有具体的属性和方法，若混入
> 的特质中具有相同的方法（方法名，参数列表，返回值均相同），必然会出现继承冲突问题。
> 冲突分为以下两种：
>
> 第一种，一个类（ Sub ）混入的两个 trait（ TraitA TraitB ）中具有相同的具体方法，且
> 两个 trait 之间没有任何关系，解决这类冲突问题，直接在类（ Sub ）中重写冲突方法。
>
> 就像Text13那样
>
> 第二种，一个类（ Sub ）混入的两个 trait（TraitA TraitB ）中具有相同的具体方法，且
> 两个 trait 继承自相同的 trait（ TraitC ），及所谓的“钻石问题”，解决这类冲突问题 Scala
> 采用了 **特质叠加** 的策略。
>
> 当一个类混入多个特质的时候，scala 会对所有的特质及其父特质按照一定的顺序进行排序。

```scala
object Test15_TraitOverlying {
  def main(args: Array[String]): Unit = {
    val student = new Student15
    student.increase()

    // 钻石问题特征叠加
    val myFootBall = new MyFootBall
    println(myFootBall.describe())
  }
}

// 定义球类特征
trait Ball {
  def describe(): String = "ball"
}

// 定义颜色特征
trait ColorBall extends Ball {
  var color: String = "red"

  override def describe(): String = color + "-" + super.describe()
}

// 定义种类特征
trait CategoryBall extends Ball {
  var category: String = "foot"

  override def describe(): String = category + "-" + super.describe()
}

// 定义一个自定义球的类
class MyFootBall extends CategoryBall with ColorBall {
  override def describe(): String = "my ball is a " + super[CategoryBall].describe()
}

trait Knowledge15 {
  var amount: Int = 0

  def increase(): Unit = {
    println("knowledge increased")
  }
}

trait Talent15 {
  def singing(): Unit

  def dancing(): Unit

  def increase(): Unit = {
    println("talent increased")
  }
}

class Student15 extends Person13 with Talent15 with Knowledge15 {
  override def dancing(): Unit = println("dancing")

  override def singing(): Unit = println("singing")

  override def increase(): Unit = {
    super[Person13].increase()
  }
}
```

> ```scala
> person increase
> my ball is a foot-ball
> ```
>
> 自定义球类中`super[CategoryBall].describe()`改为`super.describe()`的话，输出为`red-foot-ball`
>
> MyClass 中的 super 指代 Color，Color 中的 super 指代 Category，Category 中的 super 指代 Ball
>
> 如果想要调用某个指定的混入特质中的方法，可以增加约束：super[]，例如：super[CategoryBall].describe()

###### Test16_TraitSelfType.scala

> 【特质自身类型】

```scala
object Test16_TraitSelfType {
  def main(args: Array[String]): Unit = {
    val user = new RegisterUser("alice", "123456")
    user.insert()
  }
}

// 用户类
class User(val name: String, val password: String)

trait UserDao {
  _: User =>

  // 向数据库插入数据
  def insert(): Unit = {
    println(s"insert into db: ${this.name}")
  }
}

// 定义注册用户类
class RegisterUser(name: String, password: String) extends User(name, password) with UserDao
```

> _: User => 中，
>
> _可以是其它的字符比如：abc
>
> ```scala
> insert into db: alice
> ```
>
> UserDao要使用User中的一些属性，但是又不想和User有什么继承上的关系，则就可以指定一个自身类型，然后就相当于在UserDao中有了一个User对象一样，就可以调用User中的一些属性了。
>
> 当前要用到的这个User类的实例，相当于是从外部注入的，就实现依赖注入的功能。

> **特质和抽象类的区别**
>
> 优先使用特质。一个类扩展多个特质是很方便的，但却只能扩展一个抽象类。
>
> 如果你需要构造函数参数，使用抽象类。因为抽象类可以定义 带参数 的构造函数，而特质不行（有无参构造器） 。

###### Test17_Extends.scala

> 【拓展】类型检查和转换、枚举类、应用类
>
> obj.isInstanceOf[T] T]：判断 obj 是不是 T 类型
>
> obj.asInstanceOf[T] T]：将 obj 强转成 T 类型
>
> classOf 获取对象的类名
>
> 使用 type 关键字可以定义新的数据数据类型名称，本质上就是类型的一个别名

```scala
object Test17_Extends {
  def main(args: Array[String]): Unit = {
    // 1. 类型的检测和转换
    val student: Student17 = new Student17("alice", 18)
    student.study()
    student.sayHi()
    val person: Person17 = new Student17("bob", 20)
    person.sayHi()

    // 类型判断
    println("student is Student17: " + student.isInstanceOf[Student17])
    println("student is Person17: " + student.isInstanceOf[Person17])
    println("person is Person17: " + person.isInstanceOf[Person17])
    println("person is Student: " + person.isInstanceOf[Student17])

    val person2: Person17 = new Person17("cary", 35)
    println("person2 is Student17: " + person2.isInstanceOf[Student17])

    // 类型转换
    if (person.isInstanceOf[Student17]) {
      val newStudent = person.asInstanceOf[Student17]
      newStudent.study()
    }

    println(classOf[Student17])
	println("========================")

    // 2. 测试枚举类
    println(WorkDay.MONDAY)
  }
}

class Person17(val name: String, val age: Int) {
  def sayHi(): Unit = {
    println("hi from person " + name)
  }
}

class Student17(name: String, age: Int) extends Person17(name, age) {
  override def sayHi(): Unit = {
    println("hi from student " + name)
  }

  def study(): Unit = {
    println("student study")
  }
}

// 定义枚举类对象
object WorkDay extends Enumeration {
  val MONDAY = Value(1, "Monday")
  val TUESDAY = Value(2, "TuesDay")
}

// 定义应用类对象
object TestApp extends App {
  println("app start")

  type MyString = String
  val a: MyString = "abc"
  println(a)
}
```

> ```scala
> student study
> hi from student alice
> hi from student bob
> student is Student17: true
> student is Person17: true
> person is Person17: true
> person is Student: true
> person2 is Student17: false
> student study
> class chapter06.Student17
> ========================
> Monday
> ```

##### 集合

###### Test01_ImmutableArray,scala

> 【不可变数组】能用不可变就用不可变

```scala
object Test01_ImmutableArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建数组
    val arr: Array[Int] = new Array[Int](5)
    // 另一种创建方式
    val arr2 = Array(12, 37, 42, 58, 97)
    println(arr)

    // 2. 访问元素
    println(arr(0))
    println(arr(1))
    println(arr(4))
    //    println(arr(5))

    arr(0) = 12
    arr(4) = 57
    println(arr(0))
    println(arr(1))
    println(arr(4))

    println("========================")

    // 3. 数组的遍历
    // 1) 普通for循环
    for (i <- 0 until arr.length) {
      println(arr(i))
    }

    for (i <- arr.indices) println(arr(i))

    println("---------------------")

    // 2) 直接遍历所有元素，增强for循环
    for (elem <- arr2) println(elem)

    println("---------------------")

    // 3) 迭代器
    val iter = arr2.iterator

    while (iter.hasNext)
      println(iter.next())

    println("---------------------")

    // 4) 调用foreach方法
    arr2.foreach((elem: Int) => println(elem))

    arr.foreach(println)

    println(arr2.mkString("--"))

    println("========================")
    // 4. 添加元素
    val newArr = arr2.:+(73)
    println(arr2.mkString("--"))
    println(newArr.mkString("--"))

    val newArr2 = newArr.+:(30)
    println(newArr2.mkString("--"))

    val newArr3 = newArr2 :+ 15
    val newArr4 = 19 +: 29 +: newArr3 :+ 26 :+ 73
    println(newArr4.mkString(", "))
  }
}
```

> ```scala
> [I@4563e9ab
> 0
> 0
> 0
> 12
> 0
> 57
> ========================
> 12
> 0
> 0
> 0
> 57
> 12
> 0
> 0
> 0
> 57
> ---------------------
> 12
> 37
> 42
> 58
> 97
> ---------------------
> 12
> 37
> 42
> 58
> 97
> ---------------------
> 12
> 37
> 42
> 58
> 97
> 12
> 0
> 0
> 0
> 57
> 12--37--42--58--97
> ========================
> 12--37--42--58--97
> 12--37--42--58--97--73
> 30--12--37--42--58--97--73
> 19, 29, 30, 12, 37, 42, 58, 97, 73, 15, 26, 73
> ```
>

###### Test02_ArrayBuffer.scala

> 【可变数组】

```scala
import scala.collection.mutable
import scala.collection.mutable.ArrayBuffer

object Test02_ArrayBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变数组
    val arr1: ArrayBuffer[Int] = new ArrayBuffer[Int]()
    val arr2 = ArrayBuffer(23, 57, 92)

    println(arr1)
    println(arr2)

    // 2. 访问元素
    //    println(arr1(0))     // error
    println(arr2(1))
    arr2(1) = 39
    println(arr2(1))

    println("======================")
    // 3. 添加元素
    val newArr1 = arr1 :+ 15
    println(arr1)
    println(newArr1)
    println(arr1 == newArr1)

    val newArr2 = arr1 += 19
    println(arr1)
    println(newArr2)
    println(arr1 == newArr2)
    newArr2 += 13
    println(arr1)

    77 +=: arr1
    println(arr1)
    println(newArr2)

    println("----------------------")

    arr1.append(36)
    arr1.prepend(11, 76)
    arr1.insert(1, 13, 59)
    println(arr1)

    arr1.insertAll(2, newArr1)
    arr1.prependAll(newArr2)

    println(arr1)

    println("======================")
    // 4. 删除元素
    arr1.remove(3)
    println(arr1)

    arr1.remove(0, 10)
    println(arr1)

    arr1 -= 13
    println(arr1)

    println("======================")
    // 5. 可变数组转换为不可变数组
    val arr: ArrayBuffer[Int] = ArrayBuffer(23, 56, 98)
    val newArr: Array[Int] = arr.toArray
    println(newArr.mkString(", "))
    println(arr)

    // 6. 不可变数组转换为可变数组
    val buffer: mutable.Buffer[Int] = newArr.toBuffer
    println(buffer)
    println(newArr)
  }
}
```

> ```scala
> ArrayBuffer()
> ArrayBuffer(23, 57, 92)
> 57
> 39
> ======================
> ArrayBuffer()
> ArrayBuffer(15)
> false
> ArrayBuffer(19)
> ArrayBuffer(19)
> true
> ArrayBuffer(19, 13)
> ArrayBuffer(77, 19, 13)
> ArrayBuffer(77, 19, 13)
> ----------------------
> ArrayBuffer(11, 13, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(11, 13, 15, 59, 76, 77, 19, 13, 36, 11, 13, 15, 59, 76, 77, 19, 13, 36)
> ======================
> ArrayBuffer(11, 13, 15, 76, 77, 19, 13, 36, 11, 13, 15, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(15, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(15, 59, 76, 77, 19, 36)
> ======================
> 23, 56, 98
> ArrayBuffer(23, 56, 98)
> ArrayBuffer(23, 56, 98)
> [I@6cd8737
> ```

###### Test03_MulArray.scala

> 【二维数组】

```scala
object Test03_MulArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建二维数组
    val array: Array[Array[Int]] = Array.ofDim[Int](2, 3)

    // 2. 访问元素
    array(0)(2) = 19
    array(1)(0) = 25

    println(array.mkString(", "))
    for (i <- 0 until array.length; j <- 0 until array(i).length) {
      println(array(i)(j))
    }
    for (i <- array.indices; j <- array(i).indices) {
      print(array(i)(j) + "\t")
      if (j == array(i).length - 1) println()
    }

    array.foreach(line => line.foreach(println))

    array.foreach(_.foreach(println))
  }
}
```

> ```scala
> [I@b684286, [I@880ec60
> 0
> 0
> 19
> 25
> 0
> 0
> 0	0	19	
> 25	0	0	
> 0
> 0
> 19
> 25
> 0
> 0
> 0
> 0
> 19
> 25
> 0
> 0
> ```

###### Test04_List.scala

> 【不可变列表】

```scala
object Test04_List {
  def main(args: Array[String]): Unit = {
    // 1. 创建一个List
    val list1 = List(23, 65, 87)
    println(list1)	// List(23, 65, 87)

    // 2. 访问和遍历元素
    println(list1(1))
    //    list1(1) = 12
    list1.foreach(println)

    // 3. 添加元素
    val list2 = 10 +: list1
    val list3 = list1 :+ 23
    println(list1)	// List(23, 65, 87)
    println(list2)	// List(10, 23, 65, 87)
    println(list3)	// List(23, 65, 87, 23)

    println("==================")

    val list4 = list2.::(51)
    println(list4)	// List(51, 10, 23, 65, 87)

    val list5 = Nil.::(13)
    println(list5)	// List(13)

    val list6 = 73 :: 32 :: Nil
    val list7 = 17 :: 28 :: 59 :: 16 :: Nil
    println(list7)	// List(17, 28, 59, 16)

    // 4. 合并列表
    val list8 = list6 :: list7
    println(list8)	// List(List(73, 32), 17, 28, 59, 16)

    val list9 = list6 ::: list7
    println(list9)	// List(73, 32, 17, 28, 59, 16)

    val list10 = list6 ++ list7
    println(list10)	// List(73, 32, 17, 28, 59, 16)

  }
}
```

> ```scala
> List(23, 65, 87)
> 65
> 23
> 65
> 87
> List(23, 65, 87)
> List(10, 23, 65, 87)
> List(23, 65, 87, 23)
> ==================
> List(51, 10, 23, 65, 87)
> List(13)
> List(17, 28, 59, 16)
> List(List(73, 32), 17, 28, 59, 16)
> List(73, 32, 17, 28, 59, 16)
> List(73, 32, 17, 28, 59, 16)
> 
> Process finished with exit code 0
> 
> ```

###### Test05_ListBuffer.scala

> 【可变列表】

```scala
import scala.collection.mutable.ListBuffer

object Test05_ListBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变列表
    val list1: ListBuffer[Int] = new ListBuffer[Int]()
    val list2 = ListBuffer(12, 53, 75)

    println(list1)	// ListBuffer()
    println(list2)	// ListBuffer(12, 53, 75)

    println("==============")

    // 2. 添加元素
    list1.append(15, 62)
    list2.prepend(20)

    list1.insert(1, 19, 22)

    println(list1)	// ListBuffer(15, 19, 22, 62)
    println(list2)	// ListBuffer(20, 12, 53, 75)

    println("==============")

    31 +=: 96 +=: list1 += 25 += 11
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)

    println("==============")
    // 3. 合并list
    val list3 = list1 ++ list2	// list1 和 list2 都不变
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)
    println(list2)	// ListBuffer(20, 12, 53, 75)

    println("==============")

    list1 ++=: list2	// list1 不变 list2 前边加上 list1
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)
    println(list2)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11, 20, 12, 53, 75)

    println("==============")

    // 4. 修改元素
    list2(3) = 30	// 第 4 个元素被更改
    list2.update(0, 89)
    println(list2)	// ListBuffer(89, 96, 15, 30, 22, 62, 25, 11, 20, 12, 53, 75)

    // 5. 删除元素
    list2.remove(2)	// 删掉第 3 个元素 15
    list2 -= 25	// 删掉值为 25 的元素
    println(list2)	// ListBuffer(89, 96, 30, 22, 62, 11, 20, 12, 53, 75)
  }
}
```

###### Test06_ImmutableSet.scala

> 【不可变集合set】

```scala
object Test06_ImmutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1 = Set(13, 23, 53, 12, 13, 23, 78)
    println(set1)	// Set(78, 53, 13, 12, 23)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 129
    println(set1)	// Set(78, 53, 13, 12, 23)
    println(set2)	// Set(78, 53, 13, 129, 12, 23)
    println("==================")

    // 3. 合并set
    val set3 = Set(19, 13, 23, 53, 67, 99)
    val set4 = set2 ++ set3	// 将 set2 和 set3 合并
    println(set2)	// Set(78, 53, 13, 129, 12, 23)
    println(set3)	// Set(53, 13, 67, 99, 23, 19)
    println(set4)	// Set(78, 53, 13, 129, 12, 67, 99, 23, 19)

    // 4. 删除元素
    val set5 = set3 - 13	// set3 不变 set5 删除元素 13
    println(set3)	// Set(53, 13, 67, 99, 23, 19)
    println(set5)	// Set(53, 67, 99, 23, 19)
  }
}
```

###### Test07_MutableSet.scala

> 【可变集合set】

```scala
import scala.collection.mutable

object Test07_MutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1: mutable.Set[Int] = mutable.Set(13, 23, 53, 12, 13, 23, 78) // 元素被去重
    println(set1)	// Set(12, 78, 13, 53, 23)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 11
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set2)	// Set(12, 78, 13, 53, 11, 23)

    set1 += 11	// set1 本身添加元素 11
    println(set1)	// Set(12, 78, 13, 53, 11, 23)

    val flag1 = set1.add(10)
    println(flag1)	// true
    println(set1)	// Set(12, 78, 13, 53, 10, 11, 23)
    val flag2 = set1.add(10)	// 已存在元素 10 添加失败
    println(flag2)	// false
    println(set1)	// Set(12, 78, 13, 53, 10, 11, 23)

    println("==================")

    // 3. 删除元素
    set1 -= 11
    println(set1)	// Set(12, 78, 13, 53, 10, 23)

    val flag3 = set1.remove(10)
    println(flag3)	// true
    println(set1)	// Set(12, 78, 13, 53, 23)
    val flag4 = set1.remove(10)
    println(flag4)	// false
    println(set1)	// Set(12, 78, 13, 53, 23)

    println("==================")

    // 4. 合并两个Set
    val set3 = mutable.Set(13, 12, 13, 27, 98, 29)
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 13, 29, 98)

    val set4 = set1 ++ set3
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 13, 29, 98)
    println(set4)	// Set(12, 27, 78, 13, 53, 29, 98, 23)
    
    set3 ++= set1
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 78, 13, 53, 29, 98, 23)
  }
}
```

###### Test08_ImmutableMap.scala

> 【集合MAP】

```scala
object Test08_ImmutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: Map[String, Int] = Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)	// Map(a -> 13, b -> 25, hello -> 3)
    println(map1.getClass)	// class scala.collection.immutable.Map$Map3

    println("==========================")
    // 2. 遍历元素
    map1.foreach(println)
    map1.foreach((kv: (String, Int)) => println(kv))

    println("============================")

    // 3. 取map中所有的key 或者 value
    for (key <- map1.keys) {
      println(s"$key ---> ${map1.get(key)}")
    }

    // 4. 访问某一个key的value
    println("a: " + map1.get("a").get)	// a: 13
    println("c: " + map1.get("c"))	// c: None 由于没有 "c" 对应的 value, 如果 map1.get("c").get,会抛异常
    println("c: " + map1.getOrElse("c", 0))	// c: 0

    println(map1("a"))	// 13
  }
}
```

> ```scala
> Map(a -> 13, b -> 25, hello -> 3)
> class scala.collection.immutable.Map$Map3
> ==========================
> (a,13)
> (b,25)
> (hello,3)
> (a,13)
> (b,25)
> (hello,3)
> ============================
> a ---> Some(13)
> b ---> Some(25)
> hello ---> Some(3)
> a: 13
> c: None
> c: 0
> 13
> ```

###### Test09_MutableMap.scala

> 【可变MAP】

```scala
import scala.collection.mutable

object Test09_MutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: mutable.Map[String, Int] = mutable.Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)	// Map(b -> 25, a -> 13, hello -> 3)
    println(map1.getClass)	// class scala.collection.mutable.HashMap

    println("==========================")

    // 2. 添加元素
    map1.put("c", 5)
    map1.put("d", 9)
    println(map1)	// Map(b -> 25, d -> 9, a -> 13, c -> 5, hello -> 3)

    map1 += (("e", 7))
    println(map1)	// Map(e -> 7, b -> 25, d -> 9, a -> 13, c -> 5, hello -> 3)

    println("====================")

    // 3. 删除元素
    println(map1("c"))	// 5
    map1.remove("c")
    println(map1.getOrElse("c", 0))	// 0

    map1 -= "d"
    println(map1)	// Map(e -> 7, b -> 25, a -> 13, hello -> 3)

    println("====================")

    // 4. 修改元素
    map1.update("c", 5)
    map1.update("e", 10)
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)

    println("====================")

    // 5. 合并两个Map
    val map2: Map[String, Int] = Map("aaa" -> 11, "b" -> 29, "hello" -> 5)
    //    map1 ++= map2
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)
    println(map2)	// Map(aaa -> 11, b -> 29, hello -> 5)

    println("---------------------------")
    val map3: Map[String, Int] = map2 ++ map1	// map1 中的值 添加到 map2 中并更新修改成为 map3
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)
    println(map2)	// Map(aaa -> 11, b -> 29, hello -> 5)
    println(map3)	// Map(e -> 10, a -> 13, b -> 25, c -> 5, hello -> 3, aaa -> 11)
  }
}
```

Test10_Tuple.scala

> 【元组】

```scala
object Test10_Tuple {
  def main(args: Array[String]): Unit = {
    // 1. 创建元组
    val tuple: (String, Int, Char, Boolean) = ("hello", 100, 'a', true)
    println(tuple)	// (hello,100,a,true)

    // 2. 访问数据
    println(tuple._1)	// hello
    println(tuple._2)	// 100
    println(tuple._3)	// a
    println(tuple._4)	// true

    println(tuple.productElement(1))	// 100

    println("====================")
    // 3. 遍历元组数据
    for (elem <- tuple.productIterator)
      println(elem)

    println("====================")
    // 4. 嵌套元组
    val mulTuple = (12, 0.3, "hello", (23, "scala"), 29)
    println(mulTuple._4._2)	// scala
  }
}
```

> ```scala
> (hello,100,a,true)
> hello
> 100
> a
> true
> 100
> ====================
> hello
> 100
> a
> true
> ====================
> scala
> ```

###### Test11_CommonOp.scala

> 【集合常用函数】基本属性和常用操作

```scala
object Test11_CommonOp {
  def main(args: Array[String]): Unit = {
    val list = List(1, 3, 5, 7, 2, 89)
    val set = Set(23, 34, 423, 75)

    //    （1）获取集合长度
    println(list.length)	// 6

    //    （2）获取集合大小
    println(set.size)	// 4
    println("====================")

    //    （3）循环遍历
    for (elem <- list)
      println(elem)

    set.foreach(println)

    //    （4）迭代器
    for (elem <- list.iterator) println(elem)

    println("====================")
    //    （5）生成字符串
    println(list)	// List(1, 3, 5, 7, 2, 89)
    println(set)	// Set(23, 34, 423, 75)
    println(list.mkString("--"))	// 1--3--5--7--2--89

    //    （6）是否包含
    println(list.contains(23))	// false
    println(set.contains(23))	// true
  }
}
```

> ```scala
> 6
> 4
> ====================
> 1
> 3
> 5
> 7
> 2
> 89
> 23
> 34
> 423
> 75
> 1
> 3
> 5
> 7
> 2
> 89
> ====================
> List(1, 3, 5, 7, 2, 89)
> Set(23, 34, 423, 75)
> 1--3--5--7--2--89
> false
> true
> ```

###### Test12_DerivedCollection.scala

> 【集合常用函数】衍生集合

```scala
object Test12_DerivedCollection {
  def main(args: Array[String]): Unit = {
    val list1 = List(1, 3, 5, 7, 2, 89)
    val list2 = List(3, 7, 2, 45, 4, 8, 19)

    //    （1）获取集合的头
    println(list1.head)	// 1

    //    （2）获取集合的尾（不是头的就是尾）
    println(list1.tail)	// List(3, 5, 7, 2, 89)

    //    （3）集合最后一个数据
    println(list2.last)	// 19

    //    （4）集合初始数据（不包含最后一个）
    println(list2.init)	// List(3, 7, 2, 45, 4, 8)

    //    （5）反转
    println(list1.reverse)	// List(89, 2, 7, 5, 3, 1)

    //    （6）取前（后）n个元素
    println(list1.take(3))	// List(1, 3, 5)
    println(list1.takeRight(4))	// List(5, 7, 2, 89)

    //    （7）去掉前（后）n个元素
    println(list1.drop(3))	// List(7, 2, 89)
    println(list1.dropRight(4))	// List(1, 3)

    println("=========================")
    //    （8）并集
    val union = list1.union(list2)
    println("union: " + union)	// union: List(1, 3, 5, 7, 2, 89, 3, 7, 2, 45, 4, 8, 19)
    println(list1 ::: list2)	// List(1, 3, 5, 7, 2, 89, 3, 7, 2, 45, 4, 8, 19)

    // 如果是set做并集，会去重
    val set1 = Set(1, 3, 5, 7, 2, 89)
    val set2 = Set(3, 7, 2, 45, 4, 8, 19)

    val union2 = set1.union(set2)
    println("union2: " + union2)	// union2: Set(5, 89, 1, 2, 45, 7, 3, 8, 19, 4)
    println(set1 ++ set2)	// Set(5, 89, 1, 2, 45, 7, 3, 8, 19, 4)

    println("-----------------------")
    //    （9）交集
    val intersection = list1.intersect(list2)
    println("intersection: " + intersection)	// intersection: List(3, 7, 2)

    println("-----------------------")
    //    （10）差集
    val diff1 = list1.diff(list2)	// 属于 list1, 不属于 list2
    val diff2 = list2.diff(list1)	// 属于 list2, 不属于 list1
    println("diff1: " + diff1)	// diff1: List(1, 5, 89)
    println("diff2: " + diff2)	// diff2: List(45, 4, 8, 19)

    println("-----------------------")
    //    （11）拉链
    println("zip: " + list1.zip(list2))	// zip: List((1,3), (3,7), (5,2), (7,45), (2,4), (89,8))
    println("zip: " + list2.zip(list1))	// zip: List((3,1), (7,3), (2,5), (45,7), (4,2), (8,89))

    println("-----------------------")
    //    （12）滑窗
    for (elem <- list1.sliding(3))
      println(elem)
    println("-----------------------")

    for (elem <- list2.sliding(4, 2))
      println(elem)

    println("-----------------------")
    for (elem <- list2.sliding(3, 3))	// 滚动窗口
      println(elem)
  }
}
```

> ```scala
> // 滑窗
> List(1, 3, 5)
> List(3, 5, 7)
> List(5, 7, 2)
> List(7, 2, 89)
> -----------------------
> List(3, 7, 2, 45)
> List(2, 45, 4, 8)
> List(4, 8, 19)
> -----------------------
> List(3, 7, 2)
> List(45, 4, 8)
> List(19)
> ```

###### Test13_SimpleFunction.scala

> 【集合常用函数】集合计算简单函数

```scala
object Test13_SimpleFunction {
  def main(args: Array[String]): Unit = {
    val list = List(5, 1, 8, 2, -3, 4)
    val list2 = List(("a", 5), ("b", 1), ("c", 8), ("d", 2), ("e", -3), ("f", 4))

    //    （1）求和
    var sum = 0
    for (elem <- list) {
      sum += elem
    }
    println(sum)	// 17

    println(list.sum)	// 17

    //    （2）求乘积
    println(list.product)	// -960

    //    （3）最大值
    println(list.max)	// 8
    println(list2.maxBy((tuple: (String, Int)) => tuple._2))	// (c,8)
    println(list2.maxBy(_._2))	// (c,8)

    //    （4）最小值
    println(list.min)	// -3
    println(list2.minBy(_._2))	// (e,-3)

    println("========================")

    //    （5）排序
    // 5.1 sorted
    val sortedList = list.sorted
    println(sortedList)	// List(-3, 1, 2, 4, 5, 8)

    // 从大到小逆序排序
    println(list.sorted.reverse)	// List(8, 5, 4, 2, 1, -3) 不推荐
    // 传入隐式参数
    println(list.sorted(Ordering[Int].reverse))	// List(8, 5, 4, 2, 1, -3)

    println(list2.sorted)	// List((a,5), (b,1), (c,8), (d,2), (e,-3), (f,4))

    // 5.2 sortBy
    println(list2.sortBy(_._2))	// List((e,-3), (b,1), (d,2), (f,4), (a,5), (c,8)) 默认从小到大
    println(list2.sortBy(_._2)(Ordering[Int].reverse))	// List((c,8), (a,5), (f,4), (d,2), (b,1), (e,-3)) 从大到小

    // 5.3 sortWith
    println(list.sortWith((a: Int, b: Int) => {
      a < b
    }))	// List(-3, 1, 2, 4, 5, 8)
    println(list.sortWith(_ < _))	// List(-3, 1, 2, 4, 5, 8)
    println(list.sortWith(_ > _))	// List(8, 5, 4, 2, 1, -3)
  }
}
```

###### Test12_DerivedCollection.scala

> 【集合常用函数】集合计算高级函数
>
> （1）过滤：遍历一个集合并从中获取满足指定条件的元素组成一个新的集合
>
> （2）转化/映射（map）：将集合中的每一个元素映射到某一个函数
>
> （3）扁平化
>
> （4）扁平化+映射：集合中的每个元素的子元素映射到某个函数并返回新集合，flatMap 相当于先进行 map 操作，再进行 flatten 操作
>
> （5）分组（group）：按照指定的规则对集合的元素进行分组
>
> （6）简化（归约）
>
> （7）折叠

```scala
object Test14_HighLevelFunction_Map {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    // 1. 过滤
    // 选取偶数
    val evenList = list.filter((elem: Int) => {
      elem % 2 == 0
    })
    println(evenList)	// List(2, 4, 6, 8)

    // 选取奇数
    println(list.filter(_ % 2 == 1))	// List(1, 3, 5, 7, 9)

    println("=======================")

    // 2. 映射map
    // 把集合中每个数乘2
    println(list.map(_ * 2))	// List(2, 4, 6, 8, 10, 12, 14, 16, 18)
    println(list.map(x => x * x))	// List(1, 4, 9, 16, 25, 36, 49, 64, 81) 平方

    println("=======================")

    // 3. 扁平化
    val nestedList: List[List[Int]] = List(List(1, 2, 3), List(4, 5), List(6, 7, 8, 9))

    val flatList = nestedList(0) ::: nestedList(1) ::: nestedList(2)
    println(flatList)	// List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    val flatList2 = nestedList.flatten
    println(flatList2)	// List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    println("=======================")

    // 4. 扁平映射
    // 将一组字符串进行分词，并保存成单词的列表
    val strings: List[String] = List("hello world", "hello scala", "hello java", "we study")
    val splitList: List[Array[String]] = strings.map(_.split(" ")) // 分词
    val flattenList = splitList.flatten // 打散扁平化

    println(flattenList)	// List(hello, world, hello, scala, hello, java, we, study)

    val flatmapList = strings.flatMap(_.split(" "))
    println(flatmapList)	// List(hello, world, hello, scala, hello, java, we, study)

    println("========================")

    // 5. 分组groupBy
    // 分成奇偶两组
    val groupMap: Map[Int, List[Int]] = list.groupBy(_ % 2)
    val groupMap2: Map[String, List[Int]] = list.groupBy(data => if (data % 2 == 0) "偶数" else "奇数")

    println(groupMap)	// Map(1 -> List(1, 3, 5, 7, 9), 0 -> List(2, 4, 6, 8))
    println(groupMap2)	// Map(奇数 -> List(1, 3, 5, 7, 9), 偶数 -> List(2, 4, 6, 8))

    // 给定一组词汇，按照单词的首字母进行分组
    val wordList = List("china", "america", "alice", "canada", "cary", "bob", "japan")
    println(wordList.groupBy(_.charAt(0)))	// Map(c -> List(china, canada, cary), a -> List(america, alice), b -> List(bob), j -> List(japan))
  }
}
```

###### Test15_HighLevelFunction_Reduce.scala

> 【集合常用函数】集合计算高级函数
>
> （6）简化（归约）
>
> （7）折叠

```scala
object Test15_HighLevelFunction_Reduce {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4)

    // 1. reduce
    println(list.reduce(_ + _))	// 10
    println(list.reduceLeft(_ + _))	// 10
    println(list.reduceRight(_ + _))	// 10

    println("===========================")

    val list2 = List(3, 4, 5, 8, 10)
    println(list2.reduce(_ - _))	// -24
    println(list2.reduceLeft(_ - _))	// -24
    println(list2.reduceRight(_ - _)) // 3 - (4 - (5 - (8 - 10))), 6

    println("===========================")
    // 2. fold
    println(list.fold(10)(_ + _)) // 10 + 1 + 2 + 3 + 4, 20
    println(list.foldLeft(10)(_ - _)) // 10 - 1 - 2 - 3 - 4, 0
    println(list2.foldRight(11)(_ - _)) // 3 - (4 - (5 - (8 - (10 - 11)))), -5
  }
}
```

###### Test16_MergeMap.scala

> 【集合常用函数】MergeMap

```scala
import scala.collection.mutable

object Test16_MergeMap {
  def main(args: Array[String]): Unit = {
    val map1 = Map("a" -> 1, "b" -> 3, "c" -> 6)
    val map2 = mutable.Map("a" -> 6, "b" -> 2, "c" -> 9, "d" -> 3)

    //    println(map1 ++ map2)

    val map3 = map1.foldLeft(map2)(
      (mergedMap, kv) => {
        val key = kv._1
        val value = kv._2
        mergedMap(key) = mergedMap.getOrElse(key, 0) + value
        mergedMap
      }
    )

    println(map3)	// Map(b -> 5, d -> 3, a -> 7, c -> 15)
  }
}
```

> mergedMap就是map2，kv是map1的元素，把map1的每个元素融合到mergeMap中

###### Test17_CommonWordCount.scala

> 【集合常用函数】WordCount

```scala
object Test17_CommonWordCount {
  def main(args: Array[String]): Unit = {
    val stringList: List[String] = List(
      "hello",
      "hello world",
      "hello scala",
      "hello spark from scala",
      "hello flink from scala"
    )

    // 1. 对字符串进行切分，得到一个打散所有单词的列表
    //    val wordList1: List[Array[String]] = stringList.map(_.split(" "))
    //    val wordList2: List[String] = wordList1.flatten
    //    println(wordList2)
    val wordList = stringList.flatMap(_.split(" "))
    println(wordList)

    // 2. 相同的单词进行分组
    val groupMap: Map[String, List[String]] = wordList.groupBy(word => word)
    println(groupMap)

    // 3. 对分组之后的list取长度，得到每个单词的个数
    val countMap: Map[String, Int] = groupMap.map(kv => (kv._1, kv._2.length))

    // 4. 将map转换为list，并排序取前3
    val sortList: List[(String, Int)] = countMap.toList
      .sortWith(_._2 > _._2)
      .take(3)

    println(sortList)
  }
}
```

> ```scala
> List(hello, hello, world, hello, scala, hello, spark, from, scala, hello, flink, from, scala)
> Map(world -> List(world), flink -> List(flink), spark -> List(spark), scala -> List(scala, scala, scala), from -> List(from, from), hello -> List(hello, hello, hello, hello, hello))
> List((hello,5), (scala,3), (from,2))
> ```

###### Test18_ComplexWordCount.scala

> 【集合常用函数】WordCount复杂版

```scala
object Test18_ComplexWordCount {
  def main(args: Array[String]): Unit = {
    val tupleList: List[(String, Int)] = List(
      ("hello", 1),
      ("hello world", 2),
      ("hello scala", 3),
      ("hello spark from scala", 1),
      ("hello flink from scala", 2)
    )

    // 思路一：直接展开为普通版本
    val newStringList: List[String] = tupleList.map(
      kv => {
        (kv._1.trim + " ") * kv._2
      }
    )
    println(newStringList)

    // 接下来操作与普通版本完全一致
    val wordCountList: List[(String, Int)] = newStringList
      .flatMap(_.split(" ")) // 空格分词
      .groupBy(word => word) // 按照单词分组
      .map(kv => (kv._1, kv._2.size)) // 统计出每个单词的个数
      .toList
      .sortBy(_._2)(Ordering[Int].reverse)
      .take(3)

    println(wordCountList)

    println("================================")

    // 思路二：直接基于预统计的结果进行转换
    // 1. 将字符串打散为单词，并结合对应的个数包装成二元组
    val preCountList: List[(String, Int)] = tupleList.flatMap(
      tuple => {
        val strings: Array[String] = tuple._1.split(" ")
        strings.map(word => (word, tuple._2))
      }
    )
    println(preCountList)

    // 2. 对二元组按照单词进行分组
    val preCountMap: Map[String, List[(String, Int)]] = preCountList.groupBy(_._1)
    println(preCountMap)

    // 3. 叠加每个单词预统计的个数值
    val countMap: Map[String, Int] = preCountMap.mapValues(
      tupleList => tupleList.map(_._2).sum
    )
    println(countMap)

    // 4. 转换成list，排序取前3
    val countList = countMap.toList
      .sortWith(_._2 > _._2)
      .take(3)
    println(countList)
  }
}
```

> ```scala
> List(hello , hello world hello world , hello scala hello scala hello scala , hello spark from scala , hello flink from scala hello flink from scala )
> List((hello,9), (scala,6), (from,3))
> ================================
> List((hello,1), (hello,2), (world,2), (hello,3), (scala,3), (hello,1), (spark,1), (from,1), (scala,1), (hello,2), (flink,2), (from,2), (scala,2))
> Map(world -> List((world,2)), flink -> List((flink,2)), spark -> List((spark,1)), scala -> List((scala,3), (scala,1), (scala,2)), from -> List((from,1), (from,2)), hello -> List((hello,1), (hello,2), (hello,3), (hello,1), (hello,2)))
> Map(world -> 2, flink -> 2, spark -> 1, scala -> 6, from -> 3, hello -> 9)
> List((hello,9), (scala,6), (from,3))
> ```

###### Test19_Queue.scala

> 【队列Queue】

```scala
import scala.collection.immutable.Queue
import scala.collection.mutable

object Test19_Queue {
  def main(args: Array[String]): Unit = {
    // 创建一个可变队列
    val queue: mutable.Queue[String] = new mutable.Queue[String]()

    queue.enqueue("a", "b", "c")

    println(queue)	// Queue(a, b, c)
    println(queue.dequeue())	// a
    println(queue)	// Queue(b, c)
    println(queue.dequeue())	// b
    println(queue)	// Queue(c)

    queue.enqueue("d", "e")

    println(queue)	// Queue(c, d, e)
    println(queue.dequeue())	// c
    println(queue)	// Queue(d, e)

    println("==========================")

    // 不可变队列
    val queue2: Queue[String] = Queue("a", "b", "c")
    val queue3 = queue2.enqueue("d")	// queue2 未变
    println(queue2)	// Queue(a, b, c)
    println(queue3)	// Queue(a, b, c, d)

  }
}
```

###### Test20_Parallel.scala

> 【并行集合】

```scala
import scala.collection.immutable
import scala.collection.parallel.immutable.ParSeq

object Test20_Parallel {
  def main(args: Array[String]): Unit = {
    // 串行
    val result: immutable.IndexedSeq[Long] = (1 to 100).map(
      x => Thread.currentThread.getId
    )
    println(result)

    val result2: ParSeq[Long] = (1 to 100).par.map(
      x => Thread.currentThread.getId
    )
    println(result2)
  }
}
```

> ```scala
> Vector(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1)
> ParVector(11, 11, 11, 11, 11, 11, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 13, 13, 13, 13, 13, 13, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 14, 14, 14, 14, 14, 14, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12)
> ```

##### 模式匹配

> Scala 中的模式匹配类似于 Java 中的 Switch 语法

###### Test01_PatternMatchBase.scala

> 【基本语法/模式守卫】
>
> （1）如果所有 case 都不匹配，那么会执行 case _ 分支，类似于 Java 中 default 语句，若此时没有 case _ 分支，那么会抛出 MatchError
> （2）每个 case 中，不需要使用 break 语句，自动中断 case 
> （3）match case 语句可以匹配任何类型，而不只是字面量
> （4）=> 后面的代码块，直到下一个 case 语句之前的代码是作为一个整体执行，可以使用{}括起来，也可以不括
>
> 模式守卫：如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫。

```scala
object Test01_PatternMatchBase {
  def main(args: Array[String]): Unit = {
    // 1. 基本定义语法
    val x: Int = 5
    val y: String = x match {
      case 1 => "one"
      case 2 => "two"
      case 3 => "three"
      case _ => "other"
    }
    println(y)	// other

    // 2. 示例：用模式匹配实现简单二元运算
    val a = 25
    val b = 13

    def matchDualOp(op: Char): Int = op match {
      case '+' => a + b
      case '-' => a - b
      case '*' => a * b
      case '/' => a / b
      case '%' => a % b
      case _ => -1
    }

    println(matchDualOp('+'))	// 38
    println(matchDualOp('/'))	// 1
    println(matchDualOp('\\'))	// -1

    println("=========================")

    // 3. 模式守卫
    // 求一个整数的绝对值
    def abs(num: Int): Int = {
      num match {
        case i if i >= 0 => i
        case i if i < 0 => -i
      }
    }

    println(abs(67))	// 67
    println(abs(0))	// 0
    println(abs(-24))	// 24
  }
}
```

###### Test02_MatchTypes.scala

> 【模式匹配类型】
>
> Scala 中，模式匹配可以匹配所有的字面量，包括字符串，字符，数字，布尔值等等
>
> 匹配常量、匹配类型、匹配数组、匹配列表、匹配元组

```scala
object Test02_MatchTypes {
  def main(args: Array[String]): Unit = {
    // 1. 匹配常量
    def describeConst(x: Any): String = x match {
      case 1 => "Int one"
      case "hello" => "String hello"
      case true => "Boolean true"
      case '+' => "Char +"
      case _ => ""
    }

    println(describeConst("hello"))	// String hello
    println(describeConst('+'))	// Char +
    println(describeConst(0.3))	// 输出为空

    println("==================================")

    // 2. 匹配类型
    def describeType(x: Any): String = x match {
      case i: Int => "Int " + i
      case s: String => "String " + s
      case list: List[String] => "List " + list
      case array: Array[Int] => "Array[Int] " + array.mkString(",")
      case a => "Something else: " + a
    }

    println(describeType(35))	// Int 35
    println(describeType("hello"))	// String hello
    println(describeType(List("hi", "hello")))	// List List(hi, hello)
    println(describeType(List(2, 23)))	// List List(2, 23) 泛型擦除?
    println(describeType(Array("hi", "hello")))	// Something else: [Ljava.lang.String;@7dc36524
    println(describeType(Array(2, 23)))	// Array[Int] 2,23

    // 3. 匹配数组
    for (arr <- List(
      Array(0),
      Array(1, 0),
      Array(0, 1, 0),
      Array(1, 1, 0),
      Array(2, 3, 7, 15),
      Array("hello", 1, 30),
    )) {
      val result = arr match {
        case Array(0) => "0"
        case Array(1, 0) => "Array(1, 0)"
        case Array(x, y) => "Array: " + x + ", " + y // 匹配两元素数组
        case Array(0, _*) => "以0开头的数组"
        case Array(x, 1, z) => "中间为1的三元素数组"
        case _ => "something else"
      }

      println(result)
    }

    println("=========================")

    // 4. 匹配列表
    // 方式一
    for (list <- List(
      List(0),
      List(1, 0),
      List(0, 0, 0),
      List(1, 1, 0),
      List(88),
      List("hello")
    )) {
      val result = list match {
        case List(0) => "0"
        case List(x, y) => "List(x, y): " + x + ", " + y
        case List(0, _*) => "List(0, ...)"
        case List(a) => "List(a): " + a
        case _ => "something else"
      }
      println(result)
    }

    // 方式二
    val list = List(1, 2, 5, 7, 24)
    // val list1 = List(24)	// something else

    list match {
      case first :: second :: rest => println(s"first: $first, second: $second, rest: $rest")
      case _ => println("something else")
    }

    println("===========================")
    // 5. 匹配元组
    for (tuple <- List(
      (0, 1),
      (0, 0),
      (0, 1, 0),
      (0, 1, 1),
      (1, 23, 56),
      ("hello", true, 0.5)
    )) {
      val result = tuple match {
        case (a, b) => "" + a + ", " + b
        case (0, _) => "(0, _)"
        case (a, 1, _) => "(a, 1, _) " + a
        case (x, y, z) => "(x, y, z) " + x + " " + y + " " + z
        case _ => "something else"
      }
      println(result)
    }
  }
}
```

> ```scala
> // 匹配数组
> 0
> Array(1, 0)
> 以0开头的数组
> 中间为1的三元素数组
> something else
> 中间为1的三元素数组
> // 匹配列表
> // 方式一
> 0
> List(x, y): 1, 0
> List(0, ...)
> something else
> List(a): 88
> List(a): hello
> // 方式二
> first: 1, second: 2, rest: List(5, 7, 24)
> // 匹配元组
> 0, 1
> 0, 0
> (a, 1, _) 0
> (a, 1, _) 0
> (x, y, z) 1 23 56
> (x, y, z) hello true 0.5
> ```

###### Test03_MatchTupleExtend.scala

> 【增强元组匹配】

```scala
object Test03_MatchTupleExtend {
  def main(args: Array[String]): Unit = {
    // 1. 在变量声明时匹配
    val (x, y) = (10, "hello")
    println(s"x: $x, y: $y")	// x: 10, y: hello

    val List(first, second, _*) = List(23, 15, 9, 78)
    println(s"first: $first, second: $second")	// first: 23, second: 15

    val fir :: sec :: rest = List(23, 15, 9, 78)
    println(s"first: $fir, second: $sec, rest: $rest")	// first: 23, second: 15, rest: List(9, 78)

    println("=====================")

    // 2. for推导式中进行模式匹配
    val list: List[(String, Int)] = List(("a", 12), ("b", 35), ("c", 27), ("a", 13))

    // 2.1 原本的遍历方式
    for (elem <- list) {
      println(elem._1 + " " + elem._2)
    }

    // 2.2 将List的元素直接定义为元组，对变量赋值
    for ((word, count) <- list) {
      println(word + ": " + count)
    }

    println("-----------------------")

    // 2.3 可以不考虑某个位置的变量，只遍历key或者value
    for ((word, _) <- list)
      println(word)

    println("-----------------------")

    // 2.4 可以指定某个位置的值必须是多少
    for (("a", count) <- list) {
      println(count)
    }
  }
}
```

> ```scala
> x: 10, y: hello
> first: 23, second: 15
> first: 23, second: 15, rest: List(9, 78)
> =====================
> a 12
> b 35
> c 27
> a 13
> a: 12
> b: 35
> c: 27
> a: 13
> -----------------------
> a
> b
> c
> a
> -----------------------
> 12
> 13
> ```

###### Test04_MatchObject.scala

> 匹配对象

```scala
object Test04_MatchObject {
  def main(args: Array[String]): Unit = {
    val student = new Student("alice", 19)

    // 针对对象实例的内容进行匹配
    val result = student match {
      case Student("alice", 18) => "Alice, 18"
      case _ => "Else"
    }

    println(result)	// Else
  }
}

// 定义类
class Student(val name: String, val age: Int)

// 定义伴生对象
object Student {
  def apply(name: String, age: Int): Student = new Student(name, age)

  // 必须实现一个unapply方法，用来对对象属性进行拆解
  def unapply(student: Student): Option[(String, Int)] = {
    if (student == null) {
      None
    } else {
      Some((student.name, student.age))
    }
  }
}
```

###### Test05_MatchCaseClass.scala

> 匹配样例类

```scala
object Test05_MatchCaseClass {
  def main(args: Array[String]): Unit = {
    val student = Student1("alice", 18)

    // 针对对象实例的内容进行匹配
    val result = student match {
      case Student1("alice", 18) => "Alice, 18"
      case _ => "Else"
    }

    println(result)	// Alice, 18
  }
}

// 定义样例类
case class Student1(name: String, age: Int)
```

###### Test06_PartialFunction.scala

> 【偏函数中的模式匹配】

```scala
object Test06_PartialFunction {
  def main(args: Array[String]): Unit = {
    val list: List[(String, Int)] = List(("a", 12), ("b", 35), ("c", 27), ("a", 13))

    // 1. map转换，实现key不变，value2倍
    val newList = list.map(tuple => (tuple._1, tuple._2 * 2))

    // 2. 用模式匹配对元组元素赋值，实现功能
    val newList2 = list.map(
      tuple => {
        tuple match {
          case (word, count) => (word, count * 2)
        }
      }
    )

    // 3. 省略lambda表达式的写法，进行简化
    val newList3 = list.map {
      case (word, count) => (word, count * 2)
    }

    println(newList)	// List((a,24), (b,70), (c,54), (a,26))
    println(newList2)	// List((a,24), (b,70), (c,54), (a,26))
    println(newList3)	// List((a,24), (b,70), (c,54), (a,26))

    // 偏函数的应用，求绝对值
    // 对输入数据分为不同的情形：正、负、0
    val positiveAbs: PartialFunction[Int, Int] = {
      case x if x > 0 => x
    }
    val negativeAbs: PartialFunction[Int, Int] = {
      case x if x < 0 => -x
    }
    val zeroAbs: PartialFunction[Int, Int] = {
      case 0 => 0
    }

    def abs(x: Int): Int = (positiveAbs orElse negativeAbs orElse zeroAbs) (x)

    println(abs(-67))	// 67
    println(abs(35))	// 35
    println(abs(0))	// 0
  }
}
```

##### 异常-隐式转换-泛型

> Scala 的异常的工作机制和 Java 一样，但是 Scala 没有“ checked （编译期）”异常，即 Scala 没有编译异常这个概念，异常都是在运行的时候捕获处理。
>
> 用 throw 关键字，抛出一个异常对象。所有异常都是 Throwable 的子类型。throw 表达式是有类型的，就是 Nothing ，因为 Nothing 是所有类型的子类型，所以 throw 表达式可以用在需要类型的地方

###### Test01_Exception.scala

> 【异常处理】

```scala
object Test01_Exception {
  def main(args: Array[String]): Unit = {
    try{
      val n = 10 / 0
    } catch {
      case e: ArithmeticException => {
        println("发生算术异常")
      }
      case e: Exception => {
        println("发生一般异常")
      }
    } finally {
      println("处理结束")
    }
  }
}
```

> ```scala
> 发生算术异常
> 处理结束
> ```

###### Test02_Implicit.scala

> 【隐式转换】隐式函数、隐式类、隐式参数
>
> 当编译器第一次编译失败的时候，会在当前的环境中查找能让代码编译通过的方法，用于将类型进行转换，实现二次编译
>
> 隐式解析机制：
>
> （1）首先会在当前代码作用域下查找隐式实体（隐式方法、隐式类、隐式对象）。 （一般是这种情况）
> （2）如果第一条规则查找隐式实体失败，会继续在隐式参数的类型的作用域里查找。类型的作用域是指与 该类型相关联的全部伴生对象以及该类型所在包的包对象。
>
> 隐式函数：隐式转换可以在不需改任何代码的情况下，扩展某个类的功能
>
> 隐式参数：普通方法或者函数中的参数可以通过 implicit 关键字声明为隐式参数，调用该方法时，就可以传入该参数，编译器会在相应的作用域寻找符合条件的隐式值。
>
> （1）同一个作用域中，相同类型的隐式值只能有一个
> （2）编译器按照隐式参数的类型去寻找对应类型的隐式值，与隐式值的名称无关
> （3）隐式参数优先于默认参数
>
> 隐式类：在Scala2.10 后提供了隐式类，可以使用 implicit 声明类，隐式类的非常强大，同样可以扩展类的功能，在集合中隐式类会发挥重要的作用
>
> （1）其所带的构造参数有且只能有一个
> （2）隐式类必须被定义在“类”或“伴生对象”或“包对象”里，即隐式类不能是顶级的

```scala
object Test02_Implicit {
  def main(args: Array[String]): Unit = {

    val new12 = new MyRichInt(12)
    println(new12.myMax(15))	// 15

    // 1. 隐式函数
    implicit def convert(num: Int): MyRichInt = new MyRichInt(num)

    println(12.myMax(15))	// 15

    println("============================")

    // 2. 隐式类
    implicit class MyRichInt2(val self: Int) {
      // 自定义比较大小的方法
      def myMax2(n: Int): Int = if (n < self) self else n

      def myMin2(n: Int): Int = if (n < self) n else self
    }

    println(12.myMin2(15))	// 12

    println("============================")

    // 3. 隐式参数

    implicit val str: String = "alice"
    //    implicit val str2: String = "alice2"	// 同一个作用域中，相同类型的隐式值只能有一个
    implicit val num: Int = 18

    def sayHello()(implicit name: String): Unit = {
      println("hello, " + name)
    }

    def sayHi(implicit name: String = "ahua"): Unit = {	// 隐式参数优先于默认参数
      println("hi, " + name)
    }

    sayHello
    sayHi

    // 简便写法
    def hiAge(): Unit = {
      println("hi, " + implicitly[Int])
    }

    hiAge()
  }
}

// 自定义类
class MyRichInt(val self: Int) {
  // 自定义比较大小的方法
  def myMax(n: Int): Int = if (n < self) self else n

  def myMin(n: Int): Int = if (n < self) n else self
}
```

> ```scala
> // 隐式参数
> hello, alice
> hi, alice
> hi, 18
> ```

###### Test03_Generics.scala

> 【泛型】

```scala
object Test03_Generics {
  def main(args: Array[String]): Unit = {
    // 1. 协变和逆变
    val child: Parent = new Child
    //    val childList: MyCollection[Parent] = new MyCollection[Child]
    val childList: MyCollection[SubChild] = new MyCollection[Child]

    // 2. 上下限
    def test[A <: Child](a: A): Unit = {
      println(a.getClass.getName)
    }

    test[SubChild](new SubChild)
  }
}

// 定义继承关系
class Parent {}

class Child extends Parent {}

class SubChild extends Child {}

// 定义带泛型的集合类型
class MyCollection[-E] {}	// 逆变
```

