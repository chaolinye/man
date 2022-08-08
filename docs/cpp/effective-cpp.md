# Effective C++

## 让自己习惯 C++

### Item 1: 视 C++ 为一个语言联邦

C++ 是一个多重范型编程语言：过程形式、面向对象形式、函数形式、泛型形式、元编程形式。

- C++ 高效编程守则视状况而变化，取决于你使用 C++ 的哪一部分

### Item 2: 尽量以 const, enum, inline 替换 #define

- 对于单纯常量，最好以 `const` 对象或 `enums` 替换 `#define`
- 对于形似函数的宏（`macros`），最好改用 `inline` 函数替换 `#define`

### Item 3: 尽可能使用 const

- 将某些东西声明为 `const` 可帮助编译器侦测出错误用法。`const` 可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体
- 编译器强制实施 `bitwise constness`, 但你编写程序时应该使用 "概念上的常量性"（conceptual constness）
- 当 `const` 和 `non-const` 成员函数有着实质等价的实现时，令 `non-const` 版本调用 `const` 版本可避免代码重复

### Item 4: 确定对象被使用前已先被初始化

- 为内置型对象进行手工初始化，因为 C++ 不保证初始化它们
- 构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作。初值列列出的成员变量，其排列次序应该和它们在 class 中的声明次序相同。
- 为免除 "跨编译单元之初始化次序" 问题，请以 `local static` 对象替换 `non-local static` 对象。

## 构造/析构/赋值运算

### Item 5: 了解 C++ 默默编写并调用哪些函数

- 编译器可以暗自为 class 创建 default 构造函数、copy 构造函数、copy assignment 操作符，以及析构函数

### Item 6: 若不想使用编译器自动生成的函数，就该明确拒绝

- 为驳回编译器自动提供的机能，可将相应的成员函数声明为 private 并且不予实现。使用像 Uncopyable 这样的 base class 也是一种做法

### Item 7：为多态基类声明 virtual 析构函数

- 基类应该声明一个 virtual 析构函数。如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数
- Classes 的设计目的如果不是作为 base classes 使用，或不是为了具备多态性，就不该声明 virtual 析构函数

### Item 8: 别让异常逃离析构函数

- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序
- 如果客户端需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数）执行该操作。

### Item 9: 绝不在构造和析构过程中调用 virtual 函数

- 在构造和析构期间不要调用 `virtual` 函数，因为这类调用从不下降至 `derived class`（比起当前执行构造函数和析构函数的那层） 

### Item 10: 令 operator= 返回一个 reference to *this

### Item 11: 在 operator= 中处理 "自我赋值"

- 确保当对象自我赋值时 `operator=` 有良好行为。其中技术包括比较"来源对象"和"目标对象"的地址、精心周到的语句顺序、以及 `copy-and-swap`
- 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确

### Item 12: 复制对象时勿忘其每一个成分

- Copying 函数应该确保复制 "对象内的所有成员变量" 及 "所有 base class 成分"
- 不要尝试以某个 copying 函数实现另一个 copying 函数。应该将共同机能放进第三个函数中，并由两个 coping 函数共同调用。

## 资源管理

### Item 13: 以对象管理资源

- 为防止资源泄漏，请使用 `RAII`(资源取得时机便是初始化时机)，它们在构造函数中获得资源并在析构函数中释放资源
- 两个常被使用的 RAII classes 分别时 `tr1:shared_ptr` 和 `auth_ptr`。前者通常时较佳选择，因为其 copy 行为比较直观。若选择 auto_ptr, 复制动作会使它（被复制物）指向 null

> RAII 守则：资源在构造期间获得，在析构期间释放

### Item 14: 在资源管理类中小心 coping 行为

- 复制 RAII 对象必须一并复制它所管理的资源，所以资源的 copying 行为决定 RAII 对象的 copying 行为
- 普遍而常见的 RAII class copying 行为是：抑制 copying、施行引用计数法（reference counting）。不过其他行为也都可能被实现

### Item 15: 在资源管理类中提供对原始资源的访问

- API 往往要求访问原始资源，所以每一个 RAII class 应该提供一个 "取得其管理之资源" 的方法
- 对原始资源的访问可能经由显示转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便

### Item 16: 成对使用 new 和 delete 时要采取相同形式

- 如果你在 `new` 表达式中使用 `[]`, 必须在相应的 `delete` 表达式中也使用 `[]`。如果你在 `new` 表达式中不使用 `[]`, 一定不要再相应的 `delete` 表达式中使用 `[]`

### Item 17: 以独立语句将 newed 对象置入智能指针

- 以独立语句将 newed 对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏

## 设计与声明

### Item 18: 让接口容易被正确使用，不易被误用

- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质
- "促进正确使用" 的方法包含接口的一致性，以及与内置类型的行为兼容
- "阻止误用" 的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任
- `tr1::shared_ptr` 支持定制型删除器。这可防范 DLL 问题，可被用来自动解除互斥锁等等

### Item 19: 设计 class 犹如设计 type

- 新 type 的对象应该如何被创建和销毁？
- 对象的初始化和对象的赋值该由什么样的差别？
- 新 type 的对象如果被 passed by value，意味着什么？
- 什么是新 type 的 合法值？
- 你的新 type 需要配置某个继承图系吗？
- 你的新 type 需要什么样的转换？
- 什么样的操作符和函数对此新 type 而言是合理的？
- 什么样的标准函数应该驳回？
- 谁该取用新的 type 的成员？
- 什么是新 type 的 “未声明接口”？
- 你的新 type 有多么一般化？
- 你真的需要一个新 type 吗？

### Item 20: 宁以 pass-by-reference-to-const 替换 pass-by-value

- 尽量以 `pass-by-reference-to-const` 替换 `pass-by-value`。前者通常比较高效，并可避免切割问题。
- 以上规则并不适用于内置类型，以及 STL 的迭代器和函数对象。对它们而言，pass-by-value 往往比较适当

### Item 21: 必须返回对象时，别妄想返回其 reference

- 绝不要返回 pointer 或者 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象


### Item 22: 将成员变量声明为 private

- 切记将成员变量声明为 private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性
- protected 并不比 public 更具封装性

### Item 23: 宁以 non-member、non-friend 替换 member 函数

- 宁可拿 non-member non-friend 函数替换 member 函数。这样做可以增加封装性、包裹弹性（packaging flexibility） 和机能扩充性

### Item 24: 若所有参数皆需类型转换，请为此采用 non-member 函数

- 如果你需要为某个函数的所有参数（包括被 this 指针所指的那个隐喻参数）进行类型转换，那么这个函数必须要是个 non-member

### Item 25: 考虑写出一个不抛异常的 swap 函数

- 当 `std::swap` 对你的类型效率不高时，提供一个 `swap` 成员函数，并确定这个函数不抛出异常.
- 如果你提供一个 `member swap`，也该提供一个 `non-member swap` 用来调用前者。对于 classes（而非 templates），也请特化 `std::swap`
- 调用 `swap` 时应针对 `std::swap` 使用 using 声明式，然后调用 swap 并且不带任何"命名空间资格修饰"
- 为 "用户定义类型" 进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std 而言全新的东西

## 实现

### Item 26：尽可能延后变量定义式的出现时间

- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。

> 对于循环变量，也尽量使用循环内定义，除非效率高度敏感

### Item 27: 尽量少做转型动作

- 如果可以，尽量避免转型，特别式在注重效率的代码中避免 `dynamic_casts`。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- 如果转型式必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
- 宁可使用 C++ style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

### Item 28: 避免返回 handles 指向对象内部成分

- 避免返回 handles（包括 references、指针、迭代器）指向对象内部。遵守这个条款可以增加封装性，帮助 const 成员函数的行为像个 const，并将发生 "虚吊号码牌"（dangling handles）的可能性降至最低。

### Item 29：为 "异常安全" 而努力式值得的

- 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
- “强烈保证” 往往能够以 `copy-and-swap` 实现出来，但 "强烈保证" 并非对所有函数都可实现或具备现实意义。
- 函数提供的 "异常安全保证" 通常最高只等于其所调用之各个函数的 “异常安全保证” 中的最弱者。

### Item 30: 透彻了解 inlining 的里里外外

- 将大多数 inlining 限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为 function templates 出现在头文件，就将它们声明为 inline

### Item 31: 将文件间的编译依存关系降至最低

- 支持 "编译依存性最小化" 的一般构想使：相依于声明式，不要相依于定义式。基于此构想的两个手段式 `Handle classes` 和 `Interface classes`
- 程序库头文件应该以 "完全且仅有声明式"（full and declaration-only forms）的形式存在。这种做法不论是否涉及 templates 都适用

## 继承与面向对象设计

### Item 32: 确定你的 public 继承塑模出 is-a 关系

- "pubic 继承" 意味 `is-a`。适用于 base classes 身上的每一件事情一定也适用于 derived classes 身上，因为每一个 derived class 对象也都是一个 base class 对象。

### Item 33: 避免遮掩继承而来的名称

- derived classes 内的名称会遮掩 base classes 内的名称。在 public 继承下从来没有人希望如此
- 为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数（forwarding functions）

### Item 34：区分接口继承和实现继承

- 接口继承和实现继承不同。在 public 继承之下，derived classed 总是继承 base class 的接口。
- pure virtual 函数只具体指定接口继承。
- 简朴的（非纯）impure virtual 函数具体指定接口继承及缺省实现继承
- non-virtual 函数具体指定接口继承以及强制性实现继承

### Item 35：考虑 virtual 函数以外的其他选择

- virtual 函数的替代方案包括 NVI 手法及 Strategy 设计模式的多种形式。NVI 手法自身是一个特殊形式的 Template Method 设计模式
- 将机能从成员函数移到 class 外部函数，带来的一个缺点是，非成员函数无法访问 class 的 non-public 成员
- tr1::function 对象的行为就像一般函数指针。这样的对象可接纳"与给定之目标签名式（target signature）兼容"的所有可调用物

### Item 36: 绝不重新定义继承而来的 non-virtual 函数

### Item 37：绝不重新定义继承而来的缺省参数值

- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而 virtual 函数——你唯一应该覆写的东西却是动态绑定
- 如果不想维护基类和派生类的缺省参数的一致性，可以使用 NVI 手法，通过在 wrapper 接口中定义缺省参数，virtual 函数则不需要缺省参数了

### Item 38: 通过复合塑模出 has-a 或 is-implemented-in-terms-of

- 复合（composition）的意义和 public 继承完全不同
- 在应用域（application domain），复合意味 has-a （有一个）。在实现域（implementation domain），复合意味 is-implemented-in-terms-of （根据某物实现出）

### Item 39: 明智而审慎地使用 private 继承

- Private 继承意味着 is-implemented-in-terms-of（根据某物实现出）。它通常比复合（composition）的级别低。但是当 derived class 需要访问 protected base class 的成员，或需要重新定义继承而来的 virtual 函数时，这么设计是合理的
- 和复合（composition）不同，private 继承可以造成 empty base 最优化。这对致力于 "对象尺寸最小化" 的程序库开发者而言，可能很中还要

### Item 40: 明智而审慎地使用多重继承

- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对 virtual 继承的需要。
- virtual 继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果 virtual base classed 不带任何数据，将是最具使用价值的情况
- 多重继承的确有正当用途。其中一个情节涉及 "public继承某个 Interface class" 和 "private 继承某个协助实现的 class"的两相组合

## 模板与泛型编程

### Item 41: 了解隐式接口和编译器多态

- classes 和 templates 都支持接口（interfaces）和多态（polymorphism）。
- 对 classes 而言接口是显式的（explicit），以函数签名为中心。多态则是通过 virtual 函数发生于运行期。
- 对 template 参数而言，接口时隐式的（implicit），奠基于有效表达式。多态则是通过 template 具现化和函数重载解析，发生于编译期

### Item 42: 了解 typename 的双重意义

- 声明 template 参数时，前缀关键字 class 和 typename 可互换
- 请使用关键字 typename 标识**嵌套从属类型名称**；但不得在 base class lists（基类列）或 member initialization list（成员初值列
）内以它作为 base class 修饰符

> template 内出现的名称如果相依于某个 template 参数，称之为从属名称。如果从属名称在 class 内呈嵌套状，我们称它为嵌套从属名称。

### Item 43: 学习处理模板化基类的名称

- 可在 derived class templates 内通过 "this->" 指设 base class templates 内的成员名称，或藉由一个明白写出的 "base class 资格修饰符"

### Item 44: 将与参数无关的代码抽离 templates

- Templates 生成多个 classes 和多个函数，所有任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系
- 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可消除，做法是以函数参数或 class 成员变量替换 template 参数。
- 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述的具现类型共享实现代码

### Item 45: 运用成员函数模板接受所有兼容类型

- 请使用 member function templates（成员函数模板）生成 "可接受所有兼容类型" 的函数
- 如果你声明 member templates 用于 "泛化 copy 构造" 或 "泛化 assignment 操作"，你还是需要声明正常的 copy 构造函数和 copy assignment 操作符

### Item 46: 需要类型转换时请为模板定义非成员函数

- 当我们编写一个 class template，而它所提供之 "与此 template 相关的" 函数支持 "所有参数之隐式类型转换" 时，请将那些函数定义为 "class template 内部的 friend 函数"

### Item 47: 请使用 traits classes 表现类型信息

- Traits classes 使得 "类型相关信息" 在编译期可用。它们以 templates 和 "templates 特化" 完成实现
- 整合重载技术（overloading）后，traits classes 有可能在编译器对类型执行 if...else 测试

### Item 48: 认识 template 元编程

- Template metaprogramming （TMP，模板元编程）可将工作由运行期移往编译器，因而得以实现早期错误侦测和更高的执行效率。
- TMP 可被用来生成 "基于政策选择组合" 的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

## 定制 new 和 delete

### Item 49: 了解 new-handler 的行为




