# C++ 标准库

## 标准库简介

1. C++98 是第一份 C++ 标准规格
2. C++03 是对 C++98 bug 的修正
3. TR1（Technical Report on C++ Libray Extensions） 内含大幅度的标准库扩充，位于 namespace std::tr1 内
4. C++11 是第二份 C++ 标准

从 C++11 开始每 3 年发布一次新标准

> C++11 对 C++ 98 的向后兼容性仅适用于源码，不保证二进制兼容

C++ 标准的源头各种各样，也导致标准库中不同组件的设计思想不一样。比如 `String` 被设计为一个安全而便利的组件；而 STL 被设计为“将不同的数据结构与算法结合起来，产生最佳效能”，并不便利也不安全。

## 标准库的基本概念

### 命令空间 std

和 class 不同，namespace 具有扩展开放性，可发生于任何源码上。

C++ 标准库中的所有标识符都被定义于一个名为 `std` 的 namespace 内（或者子 namespace 内）。

使用标准库标识符的三种选择

- 直接指定标识符. `std::ostream`
- 使用 using declaration. `using std::ostream;`
- 使用 using directive. `using namespace std;` 这种方式容易导致意外的名称冲突，应该避免使用

### 头文件

C++ 标准库的头文件是没有后缀名的

> 真实文件系统中的头文件是否有扩展名由编译器决定

```c++
#include <iostream>
#include <string>
```

这种写法也适用于 C 标准头文件，但必须采用前缀字符 c

```c++
#include <cstdlib> // was: <stdlib.h>
#include <string> // was: <string.h>
```

在这些头文件中，每个标识符都被声明于 namespace std。

### Error 和 Exception 的处理

C++ 标准库不同组件对于错误处理差异很大，例如 string class，支持具体的差错处理；而 STL 效率重于安全，因此几乎不校验逻辑差错。

![](../images/cpp_stdexcept_2.png)

逻辑错误通常可以避免

运行期异常则是由一个位于程序作用域之外的原因触发，例如资源不足。

异常类的头文件

```c++
#include <exception>    // for classes exception an bad_exception
#include <stdexcept>    // for most logic and runtime error classes
#include <system_error> // for system errors(since c++11)
#include <new>          // for out-of-memory exceptions
#include <ios>          // for I/O exceptions
#include <future>       // for errors with async() and futures(since c++11)
#include <typeinfo>     // for bad_cast and bad_typeid
```

所有标准异常都提供 `what()`,某些异常类还提供了 `code()`

### 可调用对象

Callable Object 可以是

- 一个函数
- 一个指向成员函数的指针（第一个实参是类对象的引用或者指针）
- 一个函数对象（该对象拥有 `operator()`）
- 一个 lambda，严格来说它是一种函数对象

如果想声明 callable object，一般可使用 class `std::function<>`

### 并发与多线程

C++ 11 开始

- 具备了这样的内存模型：当修改“被不同的两个线程使用的”两个不同对象时，它们彼此独立
- 引入了一个新的关键字 `thread_local`，用来定义 “线程特定”的变量和对象

STL 容器提供的并发保证：

- 并发的只读方法是允许的
- 对标准stream（`std::cin/std::cout/std::cerr`）进行格式化输入和输出的并发处理是可能的。

### 分配器

C++ 标准库定义了一个 default allocator 如下：

```c++
namespace std {
    template <typename T>
    class allocator;
}
```

绝大多数程序都是用 default allocator

## 通用工具

### Pair 和 Tuple

#### Pair

[文档](https://en.cppreference.com/w/cpp/utility/pair)

struct pair 定义于 `<utility>`

```c++
namespace std {
    template<typename T1, typename T2>
    struct pair {
        T1 first;
        T2 second;
    }
}
```

标准库中关联容器(`map/unordered_map`)就是使用 pair 来管理其以 key/value pair 形式存在的元素。任何函数如果需返回两个 value，也需要用到 pair，例如 `minmax()`

获取元素

```c++
typedef std::pair<int, float> IntFloatPair;
IntFloatPair p(42, 3.14);

std::get<0>(p)  // 等价于 p.first
std::get<1>(p)  // 等价于 p.second

std::tuple_size<IntFloatPair>::value // 2 ，获取元素个数
std::tuple_element<0, IntFloatPair>::type // int，获取元素类型
```

构造 pair

```c++
// 构造器
std::pair<int, float>(42, 3.14)
// 便捷方法，推导出 <int,double>
std::make_pair(42, 3.14)
```

Template 函数 `make_pair()` 使你无需写出类型就能生成一个 pair 对象。同时结合 auto 使用虽然很方便，但是推导出来的类型可能并不明确。

在作为实参时，还可以用初值列

```c++
void f(std::pair<int, const char*>);
void g(std::pair<int, std::string>);
...

f(std::make_pair(42, "empty"));
g(std::make_pair(42, "chair")); // type conversions

f({42, "empty"});
g({42, "chairs"}); 
```

`make_pair()` 的参数支持移动语义，同时结合 `ref()/cref()`（定义于 `<functional>`） 还支持引用语义

```c++
int i = 0;
auto p = std::make_pair(std::ref(i), std::ref(i)); // pair<int&, int&>
++pfirst;
++psecond;
assert(i == 2);
```

还可以使用 `std::tie()` 进行对象解构

```c++
std::tie(std::ignore, c) = p; // extract second value into c(ignore first one)
```

#### Tuple

[文档](https://en.cppreference.com/w/cpp/utility/tuple)

定义于 `<tuple>`

```c++
namespace std {
    template< class... Types >
    class tuple;
}
```

和 pair 一样，tuple 可以通过构造函数和 `make_tuple()` 模板方法构建，使用 `std::get()` 获取元素

```c++
tuple<int, float, string> t1(41, 6.3, "nico");

cout << get<0>(t1) << " ";
cout << get<1>(t1) << " ";

auto t2 = make_tuple(22, 44, "nico");

get<1>(t1) = get<1>(t2);
```

`std::get<i>()` 传入的索引值必须在编译器可知

tuple 的元素类型可以是 reference

```c++
string s;
tuple<string&> t(s);

get<0>(t) = "hello";
```

`make_tuple()` 默认是值传递，如果要传入引用类型，需要使用 `std::ref()`

```c++
std::string s;

auto y = std::make_tuple(ref(s)); // tuple<string&>
std::get<0>(y) = "my value" // modifies s
```

运用 reference 搭配 `make_tuple()`，可以提取 tuple 的元素值

```c++
std::tuple<int, float, std::string> t(77, 1.1, "more light");
int i;
float f;
std::string s;

std::make_tuple(std::ref(i), std::ref(f), std::ref(s)) = t;
```

为了简化使用，标准库提供了 `std::tie()`，它可以建立一个内含 reference 的 tuple，使用 `tie()` 后的简化写法

```c++
std::tuple<int, float, std::string> t(77, 1.1, "more light");
int i;
float f;
std::string s;

std::tie(i, f, s) = t;
```

如果想忽略某个元素，可以使用 `std::ignore`

```c++
std::tie(i, f, std::ignore) = t;
```

为了避免单一值被隐式转换成“带有一个元素”的 tuple， tuple “接受不定个数的实参” 的构造器被声明为 `explicit`

tuple 另外的辅助函数，主要是为了支持泛型编程

- `tuple_size<tupletype>::value` 可获取元素个数
- `tuple_element<idx, tupletype>::type` 可取得第 idx 个元素的类型（也就是 `get()` 返回值的类型）
- `tuple_cat()` 可将多个 tuple 串接成一个 tuple

#### tuple 的输入输出

大量运用了模板元编程中的迭代，然后使用偏特化版本用来终结递归调用。

```c++
#include <tuple>
#include <iostream>

template<int IDX, int MAX, typename... Args>
struct PRINT_TUPLE {
    static void print(std::ostream& strm, const std::tuple<Args...>& t) {
        strm << std::get<IDX>(t) << (IDX+1==MAX ? "" : ",");
        PRINT_TUPLE<IDX+1,MAX,Args...>::print(strm, t);
    }
}

template<int MAX, typename... Args>
struct PRINT_TUPLE<MAX, MAX, Args> {
    static void print(std::ostream& strm, const std::tuple<Args...>& t) {
    }
}


template<typename... Args>
std::ostream& operator << (std::ostream& strm, const std::tuple<Args...>& t)
{
    strm << "[";
    PRINT_TUPLE<0， sizeof...(Args), Args...>::print(strm, t);
    return strm << "]";
}

```

#### tuple 和 pair 转换

可以拿一个 pair 作为初值，初始化一个双元素 tuple，也可以将一个 pair 赋值给一个双元素 tuple。

### 智能指针

定义于 `<memory>` 内

智能指针可以解决 dangling pointer（空悬指针） 和 resource leak（资源泄露问题）

#### shared_ptr

shared_ptr 的初始化

```c++
shared_ptr<string> pNico(new string("nico"));
// 统一初始化
shared_ptr<string> pNico{new string("nico")};
// 便捷函数
shared_ptr<string> pNico = make_shared<string>("nico");
```

也可以先声明 shared_ptr, 然后对它赋值一个 new pointer。然而不可以使用 assignment 操作符，必须改用 `reset()`

```c++
shared_ptr<string> pNico; // nullptr
pNico = new string("nico"); // ERROR
pNico.reset(new string("nico")); // OK
```

可以使用 `use_count()` 方法得到当前所有者数量

最后一个 shared_ptr 在析构函数中默认使用 delete 清理资源，也可以声明自定义的 deleter

```c++
shared_ptr<string> pNico(new string("nico"),
                            [](string *p) {
                                count << "delete " << *p << endl;
                                delete p;
                            })
```

shared_ptr 默认的 delete 遇到 Array 会有问题

```c++
std::shared_ptr<int> p(new int[10]); // ERROR，but compiles，内存泄漏
std::shared_ptr<int[]> p(new int[10]); // ERROR: does not compile，类型不一致
// 正确用法
std::shared_ptr<int> p(new int[10], [](int *p) { delete[] p; }); // 自定义 deleter
std::shared_ptr<int> p(new int[10], std::default_delete<int[]>()); // 直接使用为 unique_ptr 提供的辅助函数
// unique_ptr 允许传递数组的元素类型作为 template 实参
std::unique_ptr<int[]> p(new int[10]); // OK
// unique_ptr 也可以用自定义 deleter，不可以要求明确 template 实参
std::unique_ptr<int, void(*)(int*)> p(new int[10], [](int *p) { delete[] p; });
```

shared_ptr 不提供 `operator[]`，所以无法使用下标运算符。至于 unqiue_ptr，它有一个针对 array 的偏特化，提供 `operator[]`。shared_ptr 要使用下标运算符得先使用 `get()`。

```c++
p.get()[i] = i * 42;
```

shared pointer 也很容易误用：当使用原始指针创建 shared pointer，可能会出现多个所有者 group，析构时会导致 double free 的问题

```c++
int *p = new int;
shared_ptr<int> sp1(p);
shared_ptr<int> sp2(p);
```

shared_pointer 并非线程安全，所以在多个线程中以 shared pointer 指向同一个对象，必须使用诸如锁等技术解决数据竞争问题。

#### weak_ptr

weak_ptr 允许 “共享但不拥有对象”

不能够使用操作符 `*` 和 `->` 访问 weak_ptr 指向的对象。而是必须另外建立一个 shared_pointer。

总的来说，weak_ptr 只提供了小量操作，只够用来创建、复制、赋值 weak pointer，以及转换为一个 shared pointer，或检查自己是否指向某对象。

调用 `lock()` 方法转换为 shared pointer，如果对象已过期，则返回的 shared pointer 为 nullptr，即等价于

```c++
expired() ? shared_ptr<T>() : shared_ptr<T>(*this)
```

确认指向对象存活的方法：

- `expired() == true`：效率较好
- `use_count() == 0`: 效率不是很好
- `new shared_ptr<T>(weak_ptr_var)`：不存在则抛出 bad_weak_ptr 异常

#### unique_ptr

unique_ptr 是 “其所指向对象” 的唯一拥有者。

```c++
// 初始化
std::unique_ptr<std::string> up(new int);
// 置空的两种写法
up = nullptr;
up.reset()
// 获取拥有的对象并放弃拥有权
int *p = up.release();
// 判断不为空的三种写法
if (up) {}
if (up != nullptr) {}
if (up.get() != nullptr) {}
```

shared_ptr 需要额外的内存存放计数器，unique_ptr 无需额外的开销，它消费的内存应该和 native pointer 相同

### 数值的极值

数值类型的极值与平台相关。C++ 标准库借由 template numberic_limits （`<limits>`）提供这些极值，用以取代 C 语言采用的预处理器常量(`<climits>` 和 `<cfloat>`)。

```c++
numberic_limits<short>::max();
numberic_limits<short>::min();
numberic_limits<int>::max();
numberic_limits<int>::min();
```

### Type Trait

[文档](https://en.cppreference.com/w/cpp/meta)