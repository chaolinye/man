# Python 语言特性

## 变量

```python
message = 'Hello Python World!'
print(message)
```

> python 变量名风格是 `小写单词` +  `_`

## 常见数据类型

### 字符串

[官方文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#text-sequence-type-str)

字符串可以是单引号和双引号，也可以是三重引号: `'''三重单引号'''`, `"""三重双引号"""`

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

