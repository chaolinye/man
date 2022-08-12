# Python 语言特性

## 代码风格

- 变量名和函数名采用  `小写单词` +  `_`
- 类名采用驼峰式
- 常量采用  `全大写单词` +  `_`
- 缩进使用四个空格

> Python 中的代码块并不是用 `{}` 表示，而是用 `:` 加 缩进来表达

> Python 中省略了 `()` 的表示

> Python 句尾也不需要 `;` 

## 输入输出

```python
# 标准输出
print(message)
# 标准输入，识别回车键
num_str = input('input a number')
num = int(num_str)
```

> 如果调用print()中的字符串很长，可以在合适的位置分行。只需要在每行末尾都加上引号，同时对于除第一行外的其他各行，都在行首加上引号并缩进。

```python
print('long long long long'
    ' long message')
```

## 变量和常量

```python
message = 'Hello Python World!'
print(message)
```

> python 变量名风格是 `小写单词` +  `_`

Python 不支持常量，只能约定大写的变量视为常量

```python
MESSAGE = 'Hello  Constant'
```

多变量赋值:

```python
x, y, z = 0, 0, 0
```

python 变量的作用域:

> [Python中令人头疼的变量作用域问题，终于弄清楚了](https://zhuanlan.zhihu.com/p/374039805)

- 全局变量

    > 定义在 .py 文件内的，且函数、类之外的变量，视为全局变量。

- 局部变量

    > 定义在函数内部的变量、定义在函数声明中的形式参数，视为局部变量。

- 自由变量

    > 定义在函数中，嵌套函数外，且被嵌套函数引用的变量，视为自由变量。

- 内建变量

    > 定义在builtin中的变量，视为内置变量。

> 在函数内对全局变量使用赋值运算符会被视为局部变量的定义，如果要在函数内对全局变量真正赋值，需要使用 `global` 语句声明

```python
a = '1'
def func():
    # 声明该函数内的 a 是全局变量
    global a
    # 修改全局变量
    a = '2'
```

同样，在嵌套函数内对自由变量使用赋值运算符也会被视为重新定义了嵌套函数内的一个局部变量，这时可以使用 `nonlocal` 语句声明

```python
def outter():
    a = '1'
    def inner():
        nonlocal a
        # 修改自由变量
        a = '2'
```

## 表达式

### 算术表达式

在 Python 中，用两个星号 `**` 表示乘方运算

### 布尔表达式

- `==`: 等于
- `!=`: 不等于
- `in`: 某个元素是否存在于列表中 
- `not`: 某个元素是否不存在于列表中 

### 逻辑表达式

- `and`: 与
- `or`: 或
- `not`: 否

## 语句

### if 语句

> python 的 if 语句不需要括号，而是用冒号和缩进来表达

```python
age = 12
if age < 4:
    price = 0
elif age < 18:
    price = 25
else:
    price = 40
```

空的 list 和 None 也会被视为 False，可以用在 if 语句

```python
empty_list = []
if empty_list:
    print("list is empty")

null_obj = None

if null_obj:
    print('null object')
```

### 循环语句

```python
current_number = 1
while current_number <= 10:
    current_number += 1
    if current_number == 3:
        continue
    if current_number == 5:
        break
    print(current_number)
```



## 注释

在Python中，注释用井号 `#`标识。井号后面的内容都会被Python解释器忽略

另外，python 使用多行字符串 `""" comment """` 进行文档注释，可以通过工具生成文档

```python
"""文件说明，在文件的首行开始"""
import AModule

class NewClass:
    """ 类说明，在类定义的内部第一行 """
    def aFun():
        """ 函数说明，在函数定义的内部第一行 """s
```

## 函数

```python
def hello(name):
    msg = f'hello {name}'
    return msg
```

位置实参和关键实参

```python
def func(a, b):
    print(a)
    print(b)

# 位置实参
func(1, 2)
# 关键字实参
func(a=1, b=2)
func(b=2, a=1)
```

默认形参

```python
def func(a, b=2, c=3):
    print(a)
    print(b)
    print(c)

# 位置实参 + 默认形参
func(1)
func(1, 2, 5)
# 位置实参 + 关键字实参 + 默认形参
func(1, c=5)
```

> 代码风格：默认形参和关键字实参的 `=` 两边不用留空格

> 关键字实参结合默认形参才能发挥其效果，相对于位置实参，关键字实参可以指定后面的默认形参的值，而不用指点该默认形参前面的默认形参

任意数量形参

```python
def func(a, *args):
    print(a)
    for arg in args:
        print(b)
# 位置实参
func(1, 2, 3)
```

> args 在函数内部是个 tuples

任意数量关键字参数

```python
def func(a, **kwargs):
    print(a)
    for key, value in kwargs.items:
        print(key, value)

# 关键字实参
func(1, b=2, c=3)
```

> kwargs 在函数内部是个 map

> 很多 python 程序经常会看到形参名**kwargs，它用于收集任意数量的关键字实参。

## 常见数据类型

### 字符串

[官方文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#text-sequence-type-str)

字符串可以是单引号和双引号，也可以是三重引号: `'''三重单引号'''`, `"""三重双引号"""`

在 Python 中处理文本数据是使用 `str` 对象，也称为 字符串。 字符串是由 Unicode 码位构成的**不可变**序列。 字符串字面值有多种不同的写法：

- 单引号: `'允许包含有 "双" 引号'`

- 双引号: `"允许包含'单'引号"`

- 三重引号: `'''三重单引号'''`, `"""三重双引号"""`

> 这种灵活性让你能够在字符串中包含引号和撇号，三重引号可以跨越多行

字符串的常见方法

```python
name = 'ada lovelace'
# 单词首字母大写
print(name.title())
# 全大写
print(name.upper())
# 全小写
print(name.lower())
# 去掉头尾空白字符
print(name.strip())
# 去掉头尾特殊字符
print(name.strip('ae'))
```

格式化字符串，也被成为 `f 字符串`

> f 是 format 的简写

!> f 字符串是 python3.6 引入的

```python
first_name="abc"
last_name="123"
full_name=f"{first_name} {last_name}"
print(full_name)
# 执行表达式
print(f"Hello, {full_name.title()}!")

# python3.6 之前的写法
full_name="{} {}".format(first_name, last_name)
print(full_name)
```

### 数字类型

[文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#numeric-types-int-float-complex)

> 将任意两个数相除时，结果总是浮点数，即便这两个数都是整数且能整除

> 无论是哪种运算，只要有操作数是浮点数，Python默认得到的总是浮点数，即便结果原本为整数也是如此。

书写很大的数时，可使用下划线将其中的数字分组，使其更清晰易读

```python
universe_age = 14_000_000_000
```

> 分组只是方便阅读，同时只有Python 3.6和更高的版本支持数字书写分组

### 序列类型- list，tuple，range

[文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#sequence-types-list-tuple-range)

可以用多种方式构建列表：

- 使用一对方括号来表示空列表: `[]`

- 使用方括号，其中的项以逗号分隔: `[a]`, `[a, b, c]`

- 使用列表推导式: `[x for x in iterable]`

- 使用类型的构造器: `list()` 或 `list(iterable)`

```python
# 定义 list
bicycles = ['trek', 'cannondale', 'redline', 'specialized']

# 通过索引访问元素
print(bicycles[1])
# 访问最后一个元素
print(bicycles[-1])

# 通过索引修改元素
bicycles[0]  = 'ducai'

# 添加元素
bicycles.append("honda")
# 插入元素
bicycles.insert(0, "ducati")

# 使用 del 删除元素
del bicycles[0]
# 使用 pop 删除且返回元素
bicycles.pop(0)
# 删除最末尾的元素
bicycles.pop()
# 根据值删除元素，只删除第一个匹配的元素
bicycles.remove('redline')

# 列表排序
## 改变原对象
bicycles.sort()
bicycles.sort(reverse=True)
## 返回新对象
sorted(bicycles)
sorted(bicycles, reverse=True)

# reverse 列表
bicycles.reverse()

# 获取 list 长度
len(bicycles)

# 遍历 list
for item in bicycles:
    print(item)

# 切片，左闭右开
print(bicycles[0:3])
print(bicycles[:3])
## 使用切片复制 list
new_bicycles = bicycles[:]

# 判断某个元素是否存在列表中
if 'redline' in bicycles:
    print("has redline")
```

Python 函数 `range()` 让你能够轻松地生成一系列数。

```python
# 打印 1-4
for value in range(1, 5):
    print(value)

# range 转 list
numbers = list(range(1, 6))
# 数字 list 的统计计算
print(max(numbers))
print(min(numbers))
print(sum(numbers))
```

python 支持 **列表解析** 语法

```python
squares = [value ** 2 for value in range(1, 11)]
```

tuple 元组，即不可变的列表

可以用多种方式构建元组：

- 使用一对圆括号来表示空元组: `()`

- 使用一个后缀的逗号来表示单元组: `a,` 或 `(a,)`

- 使用以逗号分隔的多个项: `a, b, c` or `(a, b, c)`

- 使用内置的 tuple(): `tuple()` 或 `tuple(iterable)`

!> 元组定义核心是逗号，小括号可以没有

```python
dimensions = [200, 50]
for dimension in dimensions:
    print(dimension)
```

> 虽然不能修改元组的元素，但可以给存储元组的变量赋值。

### 字典 - dict

[文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#mapping-types-dict)

```python
# 使用花括号创建 dict
alien = {'color': 'green', 'points': 5}
# 多行模式, 注意最后的 } 也要缩进
alien = {
    'color': 'green',
    'points': 5
    }

# 读取 dict 的值
## [] 读取没有的值，会报异常，类似于 list 的下标读取
print(alien['color'])
# get 读取没有的值，会返回 None
print(alien.get('points'))

# 添加键值对
alien['x_position'] = 0
alien['y_position'] = 25

# 删除键值对
del alien['points']

# 遍历字典
## 默认是遍历key
for key in alien:
    print(alien[key])
## 显式遍历 key
for key in alien.keys():
    print(alien[key])
## 遍历 value
for value in alien.values():
    print(value)
## 遍历键值对
for key,value in alien.items():
    print(key)
    print(value)
## 有序的遍历
for key in sorted(alien.keys()):
    print(alien[key])
```

> 在Python 3.7中，字典中元素的排列顺序与定义时相同。如果将字典打印出来或遍历其元素，将发现元素的排列顺序与添加顺序相同。

### 集合类型 - set

[文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#set-types-set-frozenset)

```python
# 定义 set，会自动去重
languages = {'python', 'ruby', 'python', 'c'}
```

## 类

```python
class Dog:
    """类的说明"""

    def __init__(self, name, age):
        """构造方法"""
        self.name = name
        self.age = age

    def sit(self):
        """方法说明"""
        print(f"{self.name} is now sitting")

my_dog = Dog('Willie', 6)
print(my_dog.name)
my_dog.age += 1
my_dog.sit()
```

> python 类的每个成员方法都需要定义第一个参数 `self`（名字可改）来传入类的实例对象，其它语言一般都是隐式传入，而 python 的设计理论是`显式总是比隐式好`

> 成员属性都是在 `__init__` 构造方法中定义

> python 类成员没有访问控制修饰符

类的继承：

```python
class Car:      
    """一次模拟汽车的简单尝试。"""      
    def __init__(self, make, model, year):          
        self.make = make          
        self.model = model          
        self.year = year          
        self.odometer_reading = 0      
        
    def get_descriptive_name(self):          
        long_name = f"{self.year} {self.make} {self.model}"          
        return long_name.title()      
        
    def read_odometer(self):          
        print(f"This car has {self.odometer_reading} miles on it.")     
        
    def update_odometer(self, mileage):          
        if mileage >= self.odometer_reading:              
            self.odometer_reading = mileage          
        else:              
            print("You can't roll back an odometer!")      
            
    def increment_odometer(self, miles):          
        self.odometer_reading += miles

# 继承
class ElectricCar(Car):      
    """电动汽车的独特之处。"""    
    def __init__(self, make, model, year):          
        """初始化父类的属性。"""        
        super().__init__(make, model, year)
        my_tesla = ElectricCar('tesla', 'model s', 2019)  
        print(my_tesla.get_descriptive_name())
```

> 子类 `__init__` 方法用 `super().__init__` 调用父类的方法

> 由于是动态语言，python 对象都是运行态类型，不需要多态特性

## 模块

[文档](https://docs.python.org/zh-cn/3/reference/import.html)

python 将每个 `.py` 文件视为一个模块

> 类似于 Javascript

```python
# 导入模块
import moudle_name
module_name.func()

# 导入模块并使用别名（更容易书写和避免冲突）
import module_name as m
m.func()

# 导入模块中的函数、变量、类等，并使用别名
from module_name import func, a_var, AClass as AC
func()
print(a_var)
obj = AC()

# 导入模块中的所有函数，变量等
from module_name import *
func()
print(a_var)
```



