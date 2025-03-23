# Java 新特性

## JShell: 快速验证简单的小问题

JDK9 正式发布

JShell提供了一种在 JShell 状态下交互式评估 Java 编程语言的声明、语句和表达式的方法。JShell 的状态包括不断发展的代码和执行状态。为了便于快速调查和编码，语句和表达式不需要出现在方法中，变量和方法也不需要出现在类中。

JShell的设计并不是为了取代IDE。JShell在处理简单的小逻辑，验证简单的小问题时，比IDE更有效率。如果我们能够在有限的几行代码中，把要验证的问题表达清楚，JShell就能够快速地给出计算的结果。这一点，能够极大地提高我们的工作效率和学习热情。

使用方式：

```bash
# 启动，-v 详细信息模式
jshell -v

# Hello world
System.out.prontln("Hello JShell")

# 退出
/exit
```

## 文字块: 编写所见即所得的字符串

在JDK 13中以预览版的形式发布，在JDK 15正式发布。

```java
// 老方式
String stringBlock =
        "<!DOCTYPE html>\n" +
        "<html>\n" +
        "    <body>\n" +
        "        <h1>\"Hello World!\"</h1>\n" +
        "    </body>\n" +
        "</html>\n";

// 新方式
String textBlock = """
        <!DOCTYPE html>
        <html>
            <body>
                <h1>"Hello World!"</h1>
            </body>
        </html>
        """;
```

 文字块由零个或多个内容字符组成，从开始分隔符开始，到结束分隔符结束。开始分隔符是由三个双引号字符 (""") ，后面跟着的零个或多个空格，以及行结束符组成的序列。结束分隔符是一个由三个双引号字符 (""")组成的序列。

需要注意的是，开始分隔符必须单独成行；三个双引号字符后面的空格和换行符都属于开始分隔符。结束分隔符只有一个由三个双引号字符组成的序列。结束分隔符之前的字符，包括换行符，都属于文字块的有效内容。

不同于传统字符串的是，在编译期，文字块要顺序通过如下三个不同的编译步骤：

1. 为了降低不同平台间换行符的表达差异，编译器把文字内容里的换行符统一转换成 LF（\u000A）；
2. 为了能够处理Java源代码里的缩进空格，要删除所有文字内容行和结束分隔符共享的前导空格，以及所有文字内容行的尾部空格；
3. 最后处理转义字符，这样开发人员编写的转义序列就不会在第一步和第二步被修改或删除。

## 档案类：精简地表达不可变数据

JDK 14中以预览版的形式发布，在JDK 16正式发布。

官方的说法，Java档案类是用来表示不可变数据的透明载体。这样的表述，有两个关键词，一个是不可变的数据，另一个是透明的载体。

```java
// 老方式
public final class Circle implements Shape {
    public final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// 新方式
public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
Circle circle = new Circle(10.0);
double radius = circle.radius();
```

为了强化“不可变”这一原则，避免面向对象设计的陷阱，Java档案类还做了以下的限制：

1. Java档案类不支持扩展子句，用户不能定制它的父类。隐含的，它的父类是java.lang.Record。父类不能定制，也就意味着我们不能通过修改父类来影响Java档案的行为。
2. Java档案类是个终极（final）类，不支持子类，也不能是抽象类。没有子类，也就意味着我们不能通过修改子类来改变Java档案的行为。
3. Java档案类声明的变量是不可变的变量。这就是我们前面反复强调的，一旦实例化就不能再修改的关键所在。
4. Java档案类不能声明可变的变量，也不能支持实例初始化的方法。这就保证了，我们只能使用档案类形式的构造方法，避免额外的初始化对可变性的影响。
5. Java档案类不能声明本地（native）方法。如果允许了本地方法，也就意味着打开了修改不可变变量的后门。

通常地，我们把Java档案类看成是一种特殊形式的Java类。除了上述的限制，Java档案类和普通类的用法是一样的。

档案类内置了下面的这些方法缺省实现：

- 构造方法
- equals方法
- hashCode方法
- toString方法
- 不可变数据的读取方法

透明载体的意思，通俗地说，就是档案类承载有缺省实现的方法，这些方法可以直接使用，也可以替换掉。

Java档案类是用来表示不可变数据的透明载体，用来简化不可变数据的表达，提高编码效率，降低编码错误。

## 封闭类：刹住失控的扩展性

在JDK 15中以预览版的形式发布, 在JDK 17正式发布

可扩展性的限定方法有四个：

- 使用私有类；
- 使用final修饰符；
- 使用sealed修饰符；
- 不受限制的扩展性。

在我们日常的接口设计和编码实践中，使用这四个限定方法的优先级应该是由高到低的。

封闭类这个概念，涉及到两种类型的类。第一种是被扩展的父类，第二种是扩展而来的子类。通常地，我们把第一种称为封闭类，第二种称为许可类。

封闭类的声明使用 sealed 类修饰符，然后在所有的 extends 和 implements 语句之后，使用 permits 指定允许扩展该封闭类的子类。

由 permits 关键字指定的许可子类（permitted subclasses），必须和封闭类处于同一模块（module）或者包空间（package）里。如果允许扩展的子类和封闭类在同一个源代码文件里，封闭类可以不使用 permits 语句，Java 编译器将检索源文件，在编译期为封闭类添加上许可的子类。

```java
// 同一包空间内
public abstract sealed class Shape permits Circle, Square {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();
}

// 同一模块内
public abstract sealed class Shape
    permits co.ivi.jus.ploar.Circle,
            co.ivi.jus.quad.Square {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();
}

// 同一源文件
public abstract sealed class Shape {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();

    public static final class Circle extends Shape {
        // snipped
    }

    public static final class Square extends Shape {
        // snipped
    }
}
// 等效于
public abstract sealed class Shape
         permits Shape.Circle, Shape.Square {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();

    public static final class Circle extends Shape {
        // snipped
    }

    public static final class Square extends Shape {
        // snipped
    }
}
```

许可类的声明需要满足下面的三个条件：

- 许可类必须和封闭类处于同一模块（module）或者包空间（package）里，也就是说，在编译的时候，封闭类必须可以访问它的许可类；
- 许可类必须是封闭类的直接扩展类；
- 许可类必须声明是否继续保持封闭：
  - 许可类可以声明为终极类（final），从而关闭扩展性；
  - 许可类可以声明为封闭类（sealed），从而延续受限制的扩展性；
  - 许可类可以声明为解封类（non-sealed）, 从而支持不受限制的扩展性。

```java
public abstract sealed class Shape {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();
    
    public static non-sealed class Circle extends Shape {
        // snipped
    }
    
    public static sealed class Square extends Shape {
        // snipped
    }
    
    public static final class ColoredSquare extends Square {
        // snipped
    }

    public static class ColoredCircle extends Circle {
        // snipped
    }
}
```

## 类型匹配：切除臃肿的强制转换

Java的模式匹配是一个新型的、而且还在持续快速演进的领域。类型匹配是模式匹配的一个规范。

类型匹配这个特性，首先在JDK 14中以预览版的形式发布, 在JDK 16正式发布

通常，一个模式是匹配谓词和匹配变量的组合。其中，匹配谓词用来确定模式和目标是否匹配。在模式和目标匹配的情况下，匹配变量是从匹配目标里提取出来的一个或者多个变量。

对于类型匹配来说，匹配谓词用来指定模式的数据类型，而匹配变量就是一个属于该类型的数据变量。需要注意的是，对于类型匹配来说，匹配变量只有一个。

```java
// 老方式
static boolean isSquare(Shape shape) {
    if (shape instanceof Rectangle) {
        Rectangle rect = (Rectangle) shape;
        return (rect.length == rect.width);
    }

    return (shape instanceof Square);
}

// 新方式
static boolean isSquare(Shape shape) {
    if (shape instanceof Rectangle rect) {
        return (rect.length == rect.width);
    }

    return (shape instanceof Square);
}
```

匹配变量的作用域，就是目标变量可以被确认匹配的范围。如果在一个范围内，无法确认目标变量是否被匹配，或者目标变量不能被匹配，都不能使用匹配变量。 

在我们日常的编码实践中，为了简化代码逻辑，减少代码错误，提高生产效率，我们应该优先考虑使用类型匹配，而不是传统的强制类型转换运算符。

同时，类型匹配的性能是优于原来的强制转换的

## switch表达式: 简化多情景操作

在JDK 12中以预览版的形式发布, 在JDK 14正式发布

Java规范里，表达式完成对数据的操作。一个表达式的结果可以是一个数值（i * 4）；或者是一个变量（i = 4）；或者什么都不是（void类型）

Java语句是Java最基本的可执行单位，它本身不是一个数值，也不是一个变量。Java语句的标志性符号是分号（代码）和双引号（代码块），比如if-else语句，赋值语句等。这样再来看，就很简单了：switch表达式就是一个表达式，而switch语句就是一个语句。

```java
# 老方式
class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth;
        switch (month) {
            case Calendar.JANUARY:
            case Calendar.MARCH:
            case Calendar.MAY:
            case Calendar.JULY:
            case Calendar.AUGUST:
            case Calendar.OCTOBER:
            case Calendar.DECEMBER:
                daysInMonth = 31;
                break;
            case Calendar.APRIL:
            case Calendar.JUNE:
            case Calendar.SEPTEMBER:
            case Calendar.NOVEMBER:
                daysInMonth = 30;
                break;
            case Calendar.FEBRUARY:
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    daysInMonth = 29;
                } else {
                    daysInMonth = 28;
                }
                break;
            default:
                throw new RuntimeException(
                    "Calendar in JDK does not work");
        }

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}

// 新方式
class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth = switch (month) {
            case Calendar.JANUARY,
                 Calendar.MARCH,
                 Calendar.MAY,
                 Calendar.JULY,
                 Calendar.AUGUST,
                 Calendar.OCTOBER,
                 Calendar.DECEMBER -> 31;
            case Calendar.APRIL,
                 Calendar.JUNE,
                 Calendar.SEPTEMBER,
                 Calendar.NOVEMBER -> 30;
            case Calendar.FEBRUARY -> {
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    yield 29;
                } else {
                    yield 28;
                }
            }
            default -> throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
                "There are " + daysInMonth + " days in this month.");
    }
}
```

差异：

- switch代码块出现在了赋值运算符的右侧
- 多情景的合并。也就是说，一个case语句，可以处理多个情景
- 一个新的情景操作符，“->”，它是一个箭头标识符。这个符号使用在case语句里，一般化的形式是“case L ->”。
- 箭头标识符右侧的数值。这个数值，代表的就是该匹配情景下，switch表达式的数值。需要注意的是，箭头标识符右侧可以是表达式、代码块或者异常抛出语句，而不能是其他的形式。如果只需要一个语句，这个语句也要以代码块的形式呈现出来。
- 出现了一个新的关键字“yield”
- 在switch表达式里，所有的情景都要列举出来，不能多、也不能少

switch 语句也可以使用箭头标识符

```java
private static int daysInMonth(int year, int month) {
    int daysInMonth = 0;
    switch (month) {
        case Calendar.JANUARY,
             Calendar.MARCH,
             Calendar.MAY,
             Calendar.JULY,
             Calendar.AUGUST,
             Calendar.OCTOBER,
             Calendar.DECEMBER ->
            daysInMonth = 31;
        case Calendar.APRIL,
             Calendar.JUNE,
             Calendar.SEPTEMBER,
             Calendar.NOVEMBER ->
            daysInMonth = 30;
        case Calendar.FEBRUARY -> {
            if (((year % 4 == 0) && !(year % 100 == 0))
                    || (year % 400 == 0)) {
                daysInMonth = 29;
                break;
            }

            daysInMonth = 28;
        }
        // default -> throw new RuntimeException(
        //        "Calendar in JDK does not work");
    }

    return daysInMonth;
}
```

switch 表达式也可以使用冒号标记符, 但不能使用 break 语句。

```java
class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

    int daysInMonth = switch (month) {
        case Calendar.JANUARY:
        case Calendar.MARCH:
        case Calendar.MAY:
        case Calendar.JULY:
        case Calendar.AUGUST:
        case Calendar.OCTOBER:
        case Calendar.DECEMBER:
            yield 31;
        case Calendar.APRIL:
        case Calendar.JUNE:
        case Calendar.SEPTEMBER:
        case Calendar.NOVEMBER:
            yield 30;
        case Calendar.FEBRUARY:
            if (((year % 4 == 0) && !(year % 100 == 0))
                    || (year % 400 == 0)) {
                yield 29;
            } else {
                yield 28;
            }
        default:
            throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}
```

可以记住下面的总结：

- break语句只能出现在switch语句里，不能出现在switch表达式里；
- yield语句只能出现在switch表达式里，不能出现在switch语句里；
- switch表达式需要穷举出所有的情景，而switch语句不需要情景穷举；
- 使用冒号标识符的swtich形式，支持情景间的fall-through；而使用箭头标识符的swtich形式不支持fall-through；
- 使用箭头标识符的swtich形式，一个case语句支持多个情景；而使用冒号标识符的swtich形式不支持多情景的case语句。

## switch匹配: 适配不同的类型

在JDK 17中以预览版的形式发布

具有模式匹配能力的switch，说的是将模式匹配扩展到switch语句和switch表达式

```java
public static boolean isSquare(Shape shape) {
    return switch (shape) {
        case null, Shape.Circle c -> false;
        case Shape.Square s -> true;
    };
}
```

具有模式匹配能力的switch，支持空引用的匹配。如果我们能够有意识地使用这个特性，可以提高我们的编码效率，降低代码错误。

在switch的模式匹配里，我们还可以使用缺省选择情景。但是不建议使用，只有我们能够确信，待匹配类型的升级，不会影响switch表达式的逻辑的时候，我们才能考虑使用缺省选择情景。

```java
public static boolean isSquare(Shape shape) {
    return switch (shape) {
        case Shape.Square s -> true;
        case null, default -> false;
    };
}
```

运营switch模式匹配和封闭类的特性可以用类似于 Rust 的方式优化错误返回和空指针的处理方式

## 基于switch模式匹配和封闭类优化错误码的异常处理方式



## Flow 反应式编程

## 空指针处理优化

## 模型系统

## References

- [深入剖析Java新特性](https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90Java%E6%96%B0%E7%89%B9%E6%80%A7)
