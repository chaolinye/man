# C++ 移动语义

移动语义是现代 C++ 的重要特征之一。

## 移动语义的基本特征

### 移动语义的力量

#### 动机

移动的本质是偷取 **临时对象(未命名对象，返回值)** 或者 **明确说明（`std::move`）不再需要值的对象** 的堆内存或者资源。

移动语义允许对对象的复制进行优化，它可以隐式使用 (用于未命名的临时对象或局部返回
值)，也可以显式使用 (通过 std::move())。

在移动语义出现之前，这些对象只能复制，导致没必要的耗时.

最常见的两种可以从移动语义收益的对象：`std::string` 和 `std::vector`。

> 这意味着返回字符串向量，并将其赋值给现有向量不再有性能问题。我们可以像使用整型一样使用字符串向量，从而获得更好的性能。这样，大部分情况下可以直接使用返回值了，不再需要别扭的输出参数了（输出参数大部分情况下性能还是更优，但差距不大，考虑可读性，可以优先选择返回值）。

老版本的 C++ 代码使用具有移动语义的 C++11 编译器重新编译，无需修改代码，即可获得 10% 到 40% 的提速(取决于现有代码的难易程度)。

> 获得优化的原因使：编译器会自动完成临时对象和返回值的移动；如果需要进一步优化，需要在代码中合适的地方使用 `std::move`

#### 移动的实现

一个对象的移动，是通过所属类的移动构造函数或者移动赋值函数实现的

> `std::string` 和 `std::vector` 等标准库类已经实现了

> std::move() 表示不再需要这个值，它将对象标记为可移动的。标记为 std::move() 的对象不会 (部分地) 销毁 (析构函数仍然会调用)。

移动的 C++ 标准库的对象仍然是有效的对象，但其值为未定义。

#### 复制是一种应急方式

如果类没有实现这些移动函数，也会降级使用复制构造函数或者复制赋值函数，因此能做到向后兼容性

#### const 对象的移动语义

不能移动用 const 声明的对象

const 对象的 std::move() 没起作用

> const 对象不允许修改，自然也不允许被窃取内容，即不能移动

!> 因此，从 C++11 开始，用 const 返回值就不再是好的方式了

!> 如果按值 (而不是按引用) 返回，不要将返回值声明为 const。仅在声明引用或者指针返回类型时使用 const

### 移动语义的核心

#### 右值引用

> 右值引用使用 && 声明，没有 const。

根据语义，右值引用只能引用**没有名称的临时对象**，或**使用 std::move() 的对象**

与成功初始化返回值引用一样，引用将返回值的生命周期延长到引用的生命周期结束

> 普通的 const 左值引用已经具有此行为

如果编译器自动检测到从生命周期结束的对象中获取值，将自动切换到移动语义:

- 传递一个临时对象的值，该对象将在语句执行后自动撤销。
- 传递一个使用 std::move() 的非 const 对象。

可以在通过 `std::move()` 传递命名对象后使用，但通常不这样做。推荐的编程方式是不在 `std::move()` 后面使用对象

#### std::move

std::move() 对应的是右值引用类型的 static_cast。这允许我们将命名对象传递给右值引用。

> `std::move(obj)` 等同于 `static_cast<decltype(obj)&&>(obj)`

std::move() 标记的对象也可以通过 const 左值引用，传递给接受实参但不接受非 const 左值引用的函数。

> std::move 声明在 `<utility>` 头文件中，但在编译时通常不包含这个头文件，因为几乎所有的头文件都包含了 `<utility>`

#### 移动的对象

C++ 标准库保证移动的对象处于有效但未定义的状态。您仍然可以(重新) 使用它们，只要您不对它们的价值做任何期望。

> 对象移动后，一般就是成员的值没有了

移动后很多操作都是可以的，比如

- 重新赋值

    ```cpp
        std::vector<std::string> allRows;
        std::string row;
        while (std::getline(std::cin, row)) {   // read next line into row
            allRows.push_back(std::move(row));  // and move it to somewhere
        }
    ```

- 实现交换函数

    ```cpp
    template<typename T>
    void swap(T &a, T &b) {
        T tmp{std::move(a)};
        a = std::move(b);
        b = std::move(tmp);
    };
    ```

通常，已移动的对象是可以销毁的有效对象 (析构函数不应该失败)，重用以获得其他值，并支持类型的所有操作对象，而不需要知道具体值。

#### 通过引用进行重载

引入右值引用后的引用调用方式有：

- const 左值引用

    > 可以接受所有类型实参，可以作为 default 实现
    > 语义上的意思是只传递实参的读访问权。这个参数就是我们所说的 `in 参数`。

- 左值引用

    > 只接受可修改的命名对象
    > 语义上的意思是让传递的参数具有读/写访问权限。参数就是`输出或输入/输出参数`。

- 右值引用

    > 可接受没有名称的临时对象和用 std::move() 标记的非 const 对象
    > 语义上的意思是给对传递的参数的写访问权来窃取值。它是一个 in 参数，附加的约束是**调用者不再需要这个值**。

- const 右值引用

    > 可接受没有名称的临时对象和用 std::move() 标记的 const 或非 const 对象
    > const 右值引用是可能的，但针对它们的实现通常没有意义。

#### 按值传递

标记为 std::move() 的对象也可以传递给**按值接受参数**的函数。在这种情况下，移动语义用于初始化参数，这可以使按值调用非常高效。

> 按值接受参数比按右值引用接受参数多一次移动，但是比较通用，不用重载多个引用

### 类中的移动语义

#### 普通类中移动语义

!> 移动语义不可传递。

> 这是必要的，如果要传递移动语义，就不能使用两次传递了移动语义的对象。

用户声明复制构造函数、复制赋值操作符或析构函数将禁用类中对移动语义的自动支持。这不会影响派生类中的支持

> 因此没有特定的需要，就不要实现或声明析构函数

这也意味着默认情况下，多态基类禁用了移动语义。但是对于派生类的成员，移动语义仍然会自动生成 (如果派生类没有显式声明特殊的成员函数)

自动生成的移动操作在以下情况可能会有问题:

- 对成员变量有限制
-   - 值有限制
    - 值相互依赖
- 使用了引用语义的成员 (指针，智能指针，…)
- 对象没有默认构造

#### 实现复制/移动函数

对于每个类，移动构造函数和移动赋值操作符都是自动生成的 (除非没有办法这样做)。

手动实现移动构造函数和移动赋值操作符时，通常都应该有 `noexcept` 声明

> 在 vector 扩容的时候需要 noexcept 证明可以通过移动来扩容，而不用害怕失败的移动打破一致性状态而只能使用复制


#### 特殊成员函数的规则

![](../images/cpp-move-particular-method.png)

> 五个特殊成员函数，只要**声明（包括自定义，=default, =delete）**其它任何一个，都会阻止移动语义的自动支持

!> 由此可见，用 `=default` 声明一个特殊成员函数与根本不声明它是不同的

用户声明一个移动构造函数或移动赋值操作符将禁用类中对复制语义的自动支持，将得到的是只移动类型 (除非这些特殊的移动成员函数删除)。

> 永远不要 `=delete` 特殊的移动成员函数.如果想要同时禁用复制和移动，删除复制的特殊成员函数就足够了。

> 如果某个类型的移动语义不可用或已被删除，则这不会影响具有此类型成员的类的移动语义的生成。

如果没有特定的需要，不要声明析构函数。从多态基类派生的类也一样，没有特殊需要，不要声明析构函数

支持移动操作但不支持复制操作的类是有意义的。可以使用这种只移动类型传递资源的所有权或句柄，而无需共享或复制

#### 三五法则

- C++11 之前，这条原则称为“3 法则”: 要么声明全部三种 (复制构造函数、赋值操作符和析构函数)，要么一个都不声明。

- 从 C++11 开始，该规则就变成了“5 法则”，通常的表述方式是: 要么声明所有 5 种 (复制构造函数、移动构造函数、复制赋值操作符、移动赋值操作符和析构函数)，要么一个都不声明。

> 这里的声明指：实现 or `=default` or `=delete`

更多地把三五发展作为一个指导方针，当其中一个特殊成员函数是用户声明的时候，仔细考虑这 5 个特殊成员函数，通过它们之间的关系来定义即可。

比如为了只启用复制语义，应该声明默认复制特殊成员函数，这样移动特殊成员函数自然不会生成，无需刻意删除

### 如何从移动语义中获益

#### 避免命名对象

```cpp
foo (MyType{ 42 , "hello" } ) ;
```

使用无名称的临时对象会自动运用移动语义。

#### 避免不必要的 std::move()

按值返回局部对象会自动使用移动语义，如果使用 `std::move()` 显示声明，反而会导致返回值优化（RVO）失效

> RVO 优化：允许返回对象作为返回值使用

```cpp
std::string foo() {
    std::string s;
    //...
    return std::move(s);
}
```

!> 因此，如果按值返回局部对象时，不要使用 `std::move()`

在某些应用程序中，返回语句中的 `std::move()` 是合适的。一个是移出成员的值，另一个是返回带有移动语义的参数。**即返回非局部对象**

#### 用移动语义初始化成员

一般通过传值或者引用给构造函数来初始化成员，主要以下三种方式

1. 传值给构造函数，成员的初始化在可移动场景需要两次移动，在不可移动场景，需要一次复制，一次移动

2. 可以定义通用的 const 左值引用，但是无法利用移动语义

3. 为了能运用移动语义与复制，重载至少两种定义(const 左值引用，右值引用)，如果有多个成员，重载的个数是笛卡尔积。

根据经验，2 的性能往往比较差，而虽然 1 比 3 多了一次移动，但由于移动代价不高，二者的性能差异不大

结合性能和简洁性考虑，**初始化成员应该优先使用传值+移动的方式**

以上讨论的特殊情况是**创建并初始化一个新值**

**对于 setter 的场景，最佳方法是通过 const 左值引用和赋值来获取新值**，而不使用 std::move()。

传值，会导致产生临时形参，需要多一次内存分配
传右值引用，可能会导致容量收缩，导致后续的操作需要重新分配内存

#### 类中使用移动语义

基类需要声明 virtual 析构函数，因此移动特殊成员函数被禁用了，需要显式声明，但是只声明移动函数，复制函数也会被禁止，因此还需要显式声明复制函数。

另外，为了避免多态体系中移动导致的切片问题，应该禁用在多态类层次结构中使用赋值操作符，因此移动赋值函数和赋值赋值函数应该设置为 delete

对于派生类，通常不需要声明特殊的成员函数。特别是，不需要再次声明虚析构函数 (除非必须实现)。再次声明析构函数 (无论是否是虚函数) 将禁用派生类成员 (这里是 vector) 对移动语义的支持.

### 引用的重载

#### getter 的返回类型

关于 getter 应该通过值返回，还是通过常量引用返回的问题。

使用返回引用值比原始对象的时间长的话，会有生命周期上的风险

通过移动语义，就可以解决这个困境的方法。如果这样做安全，可以通过引用返回。如果遇到生命周期的问题，可以通过值返回。

使用函数的引用限定符来实现

```cpp
class Person {
private:
    std::string name;
public:
    std::string getName() &&{   // when no longer need the value
        return std::move(name);
    }
    const std::string& getName() const& {   // in all other cases
        return name;
    }
};
```

带有 `&& 限定符`的是有不再需要值的对象时使用(一个即将死亡的对象或用 std::move() 标记的对象)。

带有 `const& 限定符`的版本用于所有其他情况

结合两者，性能都很好。

这个特性意味着即使在调用成员函数时也可以使用 `std::move`

```cpp
void foo() {
    Person p;
    // ...
    coll.push_bash(p.getName);              // calls getName const&
    // ...
    coll.push_bash(std::move(p).getName()); // calls getName() &&

}
```

#### 重载

从 C++98 开始，可以重载成员函数来实现 const 和非 const 版本

有了引用限定符，就有了四种可能:

```cpp
class C {
public:
    void foo() const& {
        //...
    }
    void foo() && {
        //...
    }
    void foo() & {
        //...
    }
    void foo() const&& {
        //...
    }
}
```

!> 注意，不允许引用和非引用限定符的重载

#### 何时使用引用

引用限定符的设计目的是在为不再需要其值的对象，调用不同的成员函数，以利用移动语义提高性能

### 已移动状态

- 每个类需要说明对象的移动状态，必须确保是可销毁的 (通常情况下没有实现特殊成员函数)。然而，类的用户可能期望/要求更多。
- C++ 标准库函数的要求也适用于已移动对象。
- 生成的特殊移动成员函数可能会将已移动对象带入某种状态，从而破坏类的不变量。这种情况可能会发生，特别是当:
    - 类没有具有确定值的默认构造函数 (因此没有自然的移出状态)
    - 成员的值有限制 (比如断言)
    - 成员的值相互依赖
    - 使用了具有类似指针语义的成员 (指针、智能指针等)。
- 如果已移动状态中断不变量或使操作无效，应该使用以下选项来修复这个问题:
    - 禁用移动语义
    - 修缮了移动语义的实现
    - 类内部处理已破坏的不变量，并将对外隐藏
    - 通过记录已移动对象的约束和前置条件来消除对类不变量的影响

## 移动语义和 noexcept

noexcept 要解决的问题：vector 重新分配时，因为 push_back() 需要提供一种保证: 要么成功，要么无效，而会不安全的移动会破坏这种保证，因此不能使用移动语义。

因此设计 noexcept 指明元素移动时不会抛出异常，才在重新分配时使用移动语义。

noexcept 还支持表达式，语义上就是在满足表达式的时候保证 noexcept。

> noexcept 的引入是为了允许条件保证不抛出异常。通常，在编译时知道函数不能抛出异常可以改进代码和优化，因为不必处理由于抛出异常而进行的清理

当声明 noexcept 条件时，有几个适用的规则:

- noexcept 条件必须是编译时表达式，该表达式的值可转换为 bool 类型。
- 不能重载不同条件的函数。
- 在类层次结构中，noexcept 条件是接口的一部分。用不是 noexcept 的函数覆盖不是 noexcept 的基类函数是错误的。

如果实现了移动构造函数、移动赋值操作符或 swap()，可以用 `(有条件的)noexcept 表达式`进行声明。

对于其他函数，如果从不抛出异常，可以用无条件的 noexcept 标记。

析构函数总是使用 noexcept 来声明 (即使是在实现的时候)

### 值的种类

C++ 中的值类型是逐渐演进的

最初只有 lvalue 和 rvalue, 规则是: lvalue 可以出现在赋值的左边, rvalue 只能出现在赋值的右侧。

后来发现对于 const int 类型的值并不能放在赋值的左侧，然而它的其它特性和 lvalue 完全一致，也不适合称为 rvalue。

然后就修改了定义：lvalue 现在是程序中具有指定位置的对象 (即可以获取地址)。以同样的方式，rvalue 现在只是一个可读的值。

出现移动语义后，以前的 rvalue 变成了一个复合值类别，现在表示新的主值类别 prvalue(对于以前的所有 rvalue) 和 xvalue（eXpire value, 不再需要这个值，主要是用 std::move() 标记的对象）

c++11 的值类别

![](../images/cpp11-value.png)

主要值类别:
- lvalue (用于命名对象或字符串字面量)
- prvalue (用于未命名的临时对象)
- xvalue (对于标记为 std::move() 的对象)

常用判断规则：

- 所有用作表达式的名称都是 lvalue。
- 所有用作表达式的字符串字面值都是 lvalue。
- 所有非字符串字面值 (4.2、true 或 nullptr) 都是 prvalue。
- 所有没有名称的临时对象 (特别是回的对象) 都是 prvalues。
- 所有标记为 std::move() 的对象，及其值成员都是 xvalues。

> 由于移动语义不可传递，rvalue 传递给 rvalue 引用，但是 rvalue 引用是 lvalue

不同类别值绑定引用的优先级规则表

![](../images/cpp-value-match-reference.png)

> const lvalue 引用可以接受所有内容，并在没有提供其他重载的情况下充当备选机制。

可以使用 decltype 检查值类型（用表达式的方式）

> decltype 检查名称，无法判断值类型，只能用表达式的方式

- 对于 prvalue，产生值类型:type
- 对于 lvalue，将其类型作为 lvalue 引用:type&
- 对于 xvalue，将其类型作为 rvalue 引用:type&&

检查方法：

- `!std::is_reference_v<decltype((expr))>` 检查 expr 是否为 prvalue。
- `std::is_lvalue_reference_v<decltype((expr))>` 检查 expr 是否为 lvalue。
- `std::is_rvalue_reference_v<decltype((expr))>` 检查 expr 是否为 xvalue。
- `!std::is_lvalue_reference_v<decltype((expr))>` 检查 expr 是否为 rvalue。

## 泛型代码中的移动语义

### 完美转发

为了让模板函数能同时引用 lvalue 和 rvalue，所以引入了通用引用的概念.

再者由于移动不可传递，为了能够往下透传引用的类型，引用了 `std::forward<>()` 实现 lvalue 和 rvalue 的完美转发

> std::forward<>() 是一个条件 std::move()。如果参数是 rvalue，则扩展为 std::move()

通用引用是所有重载解析的次优选择。即优先级高于另一个通用的 const 左值引用

![](../images/cpp-references.png)

但是使用通用引用重载泛型构造函数很容易出现出乎意料的情况，不建议使用。

通用引用和右值引用所用的符号相同，很容易混淆，是早期设计上的一种失败。同时通用引用是社区的事实叫法，c++ 官方后来又定为转发引用

!> 只有函数模板形参的 rvalue 引用是通用引用。类模板形参的 rvalue 引用、模板形参的成员以及全特化都是普通的 rvalue 引用，只能绑定到 rvalue。



### 完美转发的细节

通用引用是将引用绑定到任何值类别的对象，并需要保持其为 const 的唯一方法。

```cpp
void iterate(std::string::iterator beg, std::string::iterator end) {

}
void iterate(std::string::const_iterator beg, std::string::const_iterator end) {
    
}

template<typename T>
void process(T&& coll)
{
    iterate(coll.begin, coll.end);
}
```

对于以下声明：

```cpp
template<typename T>
void foo(T&& arg) {
    if
}
```

类型 T 和参数类型推导如下:

![](../images/cpp-creference.png)

> 如果传递了一个 lvalue，T 就是一个 lvalue 引用; 否则，T 不是一个引用。

判断常量和左引用

```cpp
template<typename T>
void foo(T&& arg) 
{
    if constexpr(std::is_const_v<std::remove_reference_t<T>>) {
        ... // passed argument is const
    } else if constexpr(std::is_lvalue_reference_t<T>) {
        ... //
    } else {
        ... //
    }
}
```

C++ 的引用折叠规则：
- Type& & 成为 Type&
- Type& && 成为 Type&
- Type&& & 成为 Type&
- Type&& && 成为 Type&&

`std::move` 等同于 `static_cast<remove_reference_t<T>&&>(t)`

`std::forward` 只向传递的类型参数添加 rvalue 引用， 等同于 `static_cast<T&&>(t)`

### 用 auto&& 完美传递

如何完美处理返回值？

使用 `auto&&` 声明时，也声明了一个`通用引用`。定义一个绑定到所有值类别的引用，该引用的类型保留其初始值的类型和值类别。

```cpp
// forward declarations
std::string retByValue();
std::string& retByRef();
std::string&& retByRefRef();
const std::string& retByConstRef();
const std::string&& retByConstRefRef();

// dedeced auto&& types
std::string s;
auto&& r1{s}; // std::string&
auto&& r2{std::move(s)}; // std::string&&

auto&& r3{retByValue()};    // std::string&&
auto&& r4{retByRef()};  // std::string&
auto&& r5{retByRefRef()};  // std::string&&
auto&& r6{retByConstRef()};  // const std::string&
auto&& r7{retByConstRefRef()};  // const std::string&&
```

然后完美转发 auto&& 引用

```cpp
template<typename T>
void callFoo(T&& ref) {
    foo(std::forward<T>(ref)); 
}

auto&& ref{...};

foo(std::forward<decltype(ref)>(ref));
```

auto&& 实现完美的 Lambda 转发

```cpp
auto callFoo = [](auto&& arg) {
    foo(std::forward<decltype(arg)>(arg))
}
```

> C++ 20 中可以使用 auto&& 作为函数的形参声明，等效于函数模板的通用引用声明

### 完美返回 decltype(auto)

前面讨论了参数的完美转发和返回值的完美传递之后，现在来讨论完美返回。

```cpp
template<typename T>
decltype(auto) callFoo(T&& arg) {
    return foo(std::forward<T>(arg));
}
```

> C++14 开始支持 decltype(auto) 占位符

`decltype(auto)` 遵循 decltype 的推导规则:
- 如果用普通名称初始化或返回普通名称，则返回类型是具有该名称的对象的类型。
- 如果使用表达式初始化或返回表达式，则返回类型为求值表达式的类型和值类别:
- 对于 prvalue，只产生值类型:type
- 对于 lvalue，将其类型作为 lvalue 引用:type&
- 对于 xvalue，将其类型作为 rvalue 引用:type&&

> decltype(auto) 既用于变量声明，也可以用于返回值声明

> 注意 decltype(auto) 不能有其他限定符

> return 语句中，不要把返回值/表达式作为整体用括号括起来

## C++ 标准库中的移动语义

移动语义的主要应用是**只移动类型**

> 只移动类型允许我们移动“拥有的”资源，而不能够复制。复制特殊成员函数会删除。

C++ 标准库中的只移动类型如下:
- 输入输出流
- 线程
- unique 智能指针

> 不能在 std::initializer_lists 中使用只移动类型。因为其是按值传递的，这需要复制元素
> 不能在只移动类型的集合上按值迭代。

## References

- [C++ Move Semantics]()
