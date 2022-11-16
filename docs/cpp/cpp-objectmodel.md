# C++ 对象模型

C++ 对象模型包含两个部分：

1. 语言中直接支持面向对象程序设计的部分
2. 对于各种支持的底层实现机制

> 主要关注第 2 部分

## 关于对象

在 C 语言中, “数据”和“处理数据的操作（函数）”是分开来声明的，也就是说，语言本身并没有支持“数据和函数”之间的关联性，我们把这种程序方法称为**过程性的（procedural）**

同样的逻辑，在 C++ 中，可以把数据和函数放在一起，组成 “**抽象数据类型（ADT）**” 来实现；

更进一步可以引入**继承**机制来实现面向对象；

更进一步还可以引入**模板**来实现参数化。

> C++ 支持四种编程风格：过程性、ADT、面向对象、模板

C++ 的编程风格更加的具有表达性，使用上更容易。

引入的这些封装布局成本也未必会增加。

C++ 在布局以及存取时间上主要的额外负担是由 virtual 机制引入的，包括 `virtual function` 机制 和 `virtual base class` 机制。

另外，还有一些**多重继承**下的额外负担，发生在 “一个 derived class 和其第二或后继 base class 的转换”之间。

除此之外，C++ 程序并不会比 C 慢

C++ 对象模型：

- `Nostatic data members` 被配置于每个 class object 之内
- `static data members` 被存放在所有的 class object 之外
- `static/nostatic funcion members` 被放在所有的 class object 之外
- `virtual functions` 通过 `vptr -> vtbl` 来实现

```cpp
class Point {
public:
    Point(float xval);
    virtual ~Point();

    float x() const;
    static int PointCount();

protected:
    virtual ostream& print(ostream &os) const;

    float _x;
    static int _point_count;
}
```

对应的对象模型：

![](../images/cpp-objectmodel-1.png ":size=50%")


## 构造函数语意学

关于 C++，最常听到的一个抱怨就是，编译器背着程序员做了太多事情。

Conversion 运算符就是最常被引用的例子。

比如给 `cin` 定义一个 `operator int()`，那么 `cin << intVal` 就能编译通过，变成了位运算

> 这里是故意写错输入符

事实上关键词 `explicit` 就是为了能够制止 “单一参数的 constructor” 被当作一个 conversion 运算符

### Default Constructor 的建构操作

如果一个类没有显式定义 constructor，那么 Default constructor 在需要的时候被编译器产生出来

> 这个需要指的是编译器的需要，而不是程序的需要，程序的需要由程序员保证

> 不需要的时候，不会合成 Default Constructor

!> C++ 新手一般有两个常见的误解：1. 任何 class 没有定义默认构造器，都会被合成出一个；2. 编译器合成的默认构造器会明确设定每个 data member 的默认值

这个编译器的需要主要有四种情况：

1. member class object 带有 default constructor

    > 合成的默认构造器会调用该 member 的默认构造函数
    
    > 即使已经定义了构造器，如果构造器中没有调用该 member 的构造器，编译器扩展已存在的构造器，在用户代码之前安插一些码，先调用该 member 的构造器

    > 如果多个 member 需要初始化，会按声明的次序一次调用其构造器

2. base class 带有 default constructor

    > 和 member class object 类似 

3. 存在 virtual function

    对于 virtual function 有两个扩张操作会在编译期间发生
    - 一个 virtual function table （vtbl）会被编译器产生出来，内放 class 的 virtual functions 地址
    - 在每个 class object 中，一个额外的 pointer member （vptr）会被编译器合成，内含相关的 class vtbl 的地址

    为了让这个机制生效，编译器必须为每个派生类对象的 vptr 设定初值，放置适当的 vtbl 地址。对于 class 定义的每个构造器，编译器会安插一些码来做这个事

3. 存在 virtual Base Class

    对于 virtual Base Class 只能有一份实体，因此往往需要用指针指向这份实体，这个指针编译器需要在构造器中安插代码来初始化


### Copy Constructor 的建构操作

和默认构造器一样，拷贝构造和拷贝赋值也不是一定会被生成的，得看编译器需不需要。

没有显式定义拷贝赋值，同时不存在特殊情况，拷贝构造和拷贝赋值都不会被自动生成，而是采用 Bitwise Copy（位拷贝）

> 位拷贝性能比拷贝构造和拷贝赋值要好

无法使用 bitwise copy 的四种特殊情况：

1. member class object 拥有拷贝构造器（无论是定义的还是合成的）
2. base class 拥有拷贝构造器（无论是定义的还是合成的）
3. 内含 virtual function

    > 可能存在对象裁剪，不能通过 bitwise copy 复制原对象的 vptr，而是应该设置新对象 class 的 vptr 值

4. 有 virtual base classes

### 程序转化语意学

#### 明确的初始化操作

```cpp
X x0;

void foo_bar() {
    X x1(x0);
    X x2 = x0;
    X x3 = X(x0);
}
```

会被编译器转换成两个阶段：1. 先定义；2.调用构造器

> **定义**是指**占用内存**的行为

```cpp
void foo_bar() {
    X x1;
    X x2;
    X x3;

    x1.X::X(x0);
    x2.X::X(x0);
    x2.X::X(x0);
}
```

#### 参数的初始化

```cpp
void foo(X x0);

X xx;
//...
foo(xx);
```

转化成

```cpp
// 修改参数为引用类型
void foo(X& x0);

X xx;
// ...
// 定义形参
X __temp0;
// 通过实参构造
__temp0.X::X(xx);
// 调用函数
foo(__temp0)
// 析构形参
__temp0.X::~X();
```

#### 返回值的初始化

```cpp
X bar() {
    X xx;
    //...
    return xx;
}

X xx = bar();

bar().memfunc();
```

转化成：

```cpp
void bar(X& __result) {
    X xx;
    xx.X::X();

    //...

    // 赋值给结果
    _result.X::XX(xx);

    return;
}

X xx;
bar(xx);

X __temp0;
(bar(__temp0), __temp0).memfunc();
```

在使用者层面做优化：

```cpp
X bar(const T &y, const T &z) {
    X xx;
    //... 以 y 和 z 处理xx；
    return xx;
}
```

可以优化为：

```cpp
X bar(const T &y, const T &z) {
    return X(y, z)
}
```

这样转换后，可以减少一个局部变量

```cpp
void bar(X &__result, const T &y, const T&z) {
    __result.X::X(y, z);
    return;
}
```

在编译器层面做优化（即 NRVO）：

转换成：

```cpp
void bar(X &__result) {
    __result.X::X();
    //...
    return;
}
```

!> NRVO 在 return 语句不是函数中的第一层级时可能不生效，这时只能从代码上模仿


### 成员们的初始化队伍

设定 class members 的初值，要么通过成员初始化列表，要么就是在构造函数内

必须使用成员初始化列表的场景:

1. 初始化一个 reference member
2. 初始化一个 const member
3. 调用一个 base class 的 constructor，而它拥有一组参数
4. 调用一个 member class 的 constructor，而它拥有一组参数

base class， member class 的 constructor 如果没有成员初始化列表中调用，编译器也会安插代码在构造器中用户代码的前面调用。因此如果在构造器中通过代码初始化，估计会调用两次 base class 或者 member class 的构造器

```cpp
class Word {
    String _name;
    int _cnt;
public:
    Word() {
        _name = 0;
        _cnt = 0;
    }
}
```

上面的写法不仅会调用两次 String 的构造器，还会产生临时对象

编译器转化后的代码：

```cpp
Word::Word(/* this pointer */) {
    // 编译器生成
    _name.String::String();

    // 产生临时对象
    String temp = String(0);
    // 拷贝
    _name.String::operator=(temp);
    // 销毁临时对象
    temp.String::~String();

    _cnt = 0;
}
```

因为这个原因，很多开发者坚持所有 member 初始化操作必须在成员初始化列表中完成，即使是一个行为良好的 member 如上面的 `_cnt`

成员初始化列表存在一个次序的坑，会导致意想不到的 bug。

编译器会对成员初始化列表重新排序，按照 member 的声明次序，而不是初始化列表中的定义顺序；然后根据声明次序安插代码到 constructor 体内，并置于任何 explicit user code 之前

> 当成员初始化列表中某个 member 依赖于另一个 member 时，要特别注意次序的问题

```cpp
class X {
    int i;
    int j;
public:
    // 存在次序问题
    X(int val): j(val), i(j) {}
}
```

正确的写法

```cpp
class X {
    int i;
    int j;
public:
    // 存在次序问题
    X(int val): j(val) {
        i = j;
    }
}
```

> 另外，在成员初始化列表中使用成员函数也要特别注意，成员函数可能会依赖其它成员变量，也会出现次序的问题。建议少在成员初始化列表中调用成员函数

## Data 语意学

### Data Member 的绑定

```cpp
extern float x;
class Point3d {
public:
    float X() const { return x; }
private:
    float x, y, z;
};
```

对于现代 C++ 编译器，Point3d::X() 返回的是成员变量 x，而不是外部的 x。

> 早期的 C++ 编译器不是这样，会返回外部的 x；这也导致了以前出现了两种防御性程序设计风格（对于现代编译器没有必要了）：
> 1. 把所有 data member 放在 class 声明开始处
> 2. 把所有的 inline functions, 不管大小都放在 class 声明之外

现代编译器对 member functions 本身的分析，会直到整个 class 的声明都出现了才开始。因此，在一个 inline member function 躯体之内的一个 data member 绑定操作，会在整个 class 声明完成之后才发生。

然而，对于 member function 的 argument list 和返回值还是会在第一次扫描时就被会绑定。

```cpp
typedef int length;

class Point3d {
public:
    // length 被绑定为 global
    // _val 被绑定为成员变量
    void mumble(length val) { _val = val; }
    length mumble() { return _val};
private:
    typedef float length;
    length _val;
};
```

!> 仍需坚持的防御性程序风格：始终把 "nested type 声明" 放在 class 的起始处

### Data Member 的布局

nonstatic data members 在 class data member 在 class object 中的排列顺序将和其被声明的顺序一样，任何中间介入的 static data member 都不会被放进对象布局之中

其实C++ 标准只是要求，在同一个 access section 中的 members 的排序只需符号“较晚出现的 members 在 class object 转给你有较高的地址”这一条件即可；对于不同 access section 中的 members 可以自由排列。

但是当前大多数编译器都在忽视 access section，直接依照声明的次序来。

### Data Member 的存取

每个 static data member 只有一个实体，存放在程序的 data segment 之中。每次程序读写 static member，就会被内部转换为对该唯一 extern 实体的直接操作.

```cpp
// origin.chunkSize == 250
Point3d::chunkSize == 250;
// pt->chunkSize == 250
Point3d::chunkSize == 250;
```

> static data member 的存取不受任何继承方式的影响，都会转换成定义该 static data member 的 `类::member` 的方式

> 不同的类的 static data member 的名称在 data segment 中可能会冲突，所有编译器一般会通过 name mangling 来给各个 static data member 生成独一无二的名称（比如加上类名等）

对 nonstatic data member 的存取，编译器需要把 class object 的起始地址加上 data member 的偏移量

``cpp
origin._y = 0.0;

// 转换成
&origin + (&Point3d::_y - 1)
``

> -1 的原因是编译器为了能区分出“一个指向 data member 的指针，指向 class 的第一个 member” 和 “一个指向 data member 的指针，没有指向任何 member” 这两种情况，就把 data member offset 加上 1。这样在求 data member 的真实地址的时候，就得 -1

在用指针或者引用存取虚拟继承得到的 nonstatic member 时，必须延迟至执行期，经由一个额外的间接层才能实现，因此速度也比较慢。

> 通过类型变量存取虚拟继承得到的 nonstatic member 不需要，offset 在编译期就可以知道。

### 继承与 Data Member

在大部分编译器上，派生类对象的内存区域中 base class members 总是先出现，但属于 virtual base class 除外

一般而言，具体继承（相对于虚拟继承）并不会添加空间或存取时间上的额外负担。

> 就是说具体继承的性能并不比 C 的 struct 差。空间大小基本一致，存取的成员函数可以通过 inline 优化

![](../images/cpp-objectmodel-struct.png ":size=50%")

![](../images/cpp-objectmodel-layout1.png ":size=50%")

多级继承：

![](../images/cpp-objectmodel-layout2.png ":size=50%")

加上多态：

多态主要是得额外考虑 vptr 的位置。

一般面向对象语言都是放在头部的，但是考虑到对 C struct 的兼容，大部分 c++ 编译器把 vptr 放在尾端

![](../images/cpp-objectmodel-layout3.png ":size=50%")

多重继承：

对于一个多重派生对象，将其地址指定给第一个 base class的指针，情况和单一继承相同，因为二者都指向相同的起始地址。需付出的成本只有地址的指定操作而已。至于第二个或后继 base class 的地址指定操作，则需要将地址修改过：加上（或减去，如果 downcast）介于中间的 base class subobjects 大小。

```cpp
Vertex3d v3d;
Vertex *pv = &v3d;
```

需要这样的内部转化：

```cpp
Vertex *pv = (Vertex*)(((char*)&v3d) + sizeof(Point3d));
```

> 存取后继 base class 也不需要付出额外的成本，member 的位置编译时就固定了，只是一个简单的 offset 运算，和单一继承一样简单

![](../images/cpp-objectmodel-layout4.png ":size=50%")

虚拟继承：

两种布局方式：

通过指针的方式：

缺点：每个对象都得背负这个 virtual base class 指针，空间成本高

![](../images/cpp-objectmodel-layout5.png ":size=50%")

offset的方式

![](../images/cpp-objectmodel-layout6.png ":size=50%")

> 无论使用哪种实现方式，都需要执行期经过中间层才能获取 virtual base class 的起始地址，效率较低

### 对象成员的效率

除了虚拟继承，无论是聚合、封装、继承下，存取对象存取并不会导致效率损耗。

> 前提是开启编译器的优化

### 指向 Data Member 的指针

```cpp
float Point3d:*p1 = 0;
float Point3d:*p2 = &Point3d::x;
```

取址 nostatic data member 得到的是在 class object 中的偏移量 + 1

> +1 是为了保留 offset=0 代表没有指向任何的 data member

取址 static data member 得到的是真实的地址

## Function 语意学

## 构造、析构、拷贝 语意学

## 执行期语意学

## 站在对象模型的尖端

## Reference

- [深度探索C++对象模型](https://book.douban.com/subject/1091086/)