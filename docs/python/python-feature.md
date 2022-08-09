# Python 语言特性

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

## 注释

在Python中，注释用井号 `#`标识。井号后面的内容都会被Python解释器忽略

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

### 