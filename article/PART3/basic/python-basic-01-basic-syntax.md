# Python 版本推荐
Python 主要有两个版本： 2.7.x 和 3.x

官方称 Python2.7 只维护到 2020 年，大部分 Python 库都同时支持 Python 2.7.x 和 3.x 版本，如果你的项目依赖于 Python2.7 的包，那就使用 Python2.7，否则你可以直接用 Python 3.x 开始全新的项目

> **本书基于3.x 版本讲解**

# Python IDE推荐
**1. PyCharm**

PyCharm是一个跨平台的 Python 开发工具，可以帮助用户在使用 Python 时提升效率，比如：调试、语法高亮、代码跳转、自动完成、智能提示等

**2. Sublime Text**

SublimeText 是个著名的编辑器，Sublime Text3 基本上可以 1 秒即启动，反应速度很快。同时它对 Python 的支持也很到位，具有代码高亮、语法提示、自动完成等功能。

# Python基础语法

## Python 运算

### 变量
**1. Python中的变量赋值不需要类型声明**

在内存中创建每个变量时包括了变量的标识、名称和数据这些信息。

```py
count = 100         
ratio = 100.0       
str =  "python"     

print(count)        # 输出 100
print(ratio)        # 输出 100.0
print(str)          # 输出 python
```

**2. Python允许同时为多个变量赋值，也可以为多个对象指定多个变量**

```py
a = b = c = 100

x, y, z = 100, 100.0, "python"
```

**3. 基本运算符如下：**

符号 | 语法 
---|---
+ | 加 
- | 减
* | 乘
/ | 除 
% | 取模 
// | 向下取整 
** | 幂 

## Python 数据类型

Python有5个标准的数据类型，分别是Numbers（数字）、String（字符串）、List（列表）、Tuple（元组）和Dictionary（字典）。

### 字符串

**1. 创建字符串**

字符串是Python中最常用的数据类型。可以使用引号（单引号或双引号）来创建字符串。
```py
str1 = 'Hello World!'
str2 = "Hello Python!"

print(str1)         # 输出 Hello World!
print(str2)         # 输出 Hello Python!
```


**2. 访问字符串**

在Python中访问子字符串时，可以使用方括号“[]”来截取字符串，也可以对已存在的字符串进行修改，并赋值给另一个变量。

```py
str1 = 'Hello World! '
str2 = "Hello Python!"

print(str1[0])          # 输出 H
print(str2[0:4])        # 输出 Hell

str1 = str1 + str2
print(str1)             # 输出 Hello World! Hello Python!
```

**3. 字符串运算符**

符号 | 语法
---|---
+ | 字符串连接
* | 重复输出字符串
[] | 用索引下标获取字符串中的字符
[ : ] | 截取字符串
in | 如果字符串中包含给出的字符，返回True
not in | 如果字符串中包含给出的字符，返回False
% | 格式化字符串

下面给出一个例子来展示这些字符串运算符的用法
```py
str1 = "Hello"
str2 = "Python"

print(str1 + str2)          # 输出 Hello Python!
print(str1 * 2)             # 输出 HelloHello
print(str1[3])              # 输出 l
print(str1[3:5])            # 输出 llo
print("H" in str1)          # 输出 True
print("P" not in str1)      # 输出 True

# 输出 "Hello world, it is 2020!"
print("Hello %s, it is %d!" % ('world', 2020))
```

### 列表

**1. 创建列表**

列表是 Python 中常用的数据结构，相当于数组，具有增删改查的功能。列表的各数据项不需要具有相同的类型，只要把用逗号分隔的不同数据项使用方括号括起来即可。

```py
list1 = ['a', 'b', 10, 10.1]
list2 = ['a', 'b', 'c']
list3 = ["A", "B", "C"]
```

**2. 访问列表**

与字符串一样，列表也可以进行截取、组合等。

操作符 | 语法
---|---
len | 获得列表元素个数
+ | 组合
* | 重复输出列表
in | 元素如果存在列表中，返回True
for | 迭代循环这个列表

```py
list1 = ['a', 'b', 10, 10.1]
list2 = ['a', 'b', 'c']
list3 = ["A", "B", "C"]

print(list1[0])             # 输出 a
print(list2[1, 2])          # 输出 ['a', 'b']

list2.append("python")
print(list2)                # 输出 ['a', 'b', 'c', 'python'] 

print(list3 * 2)            # 输出 ['A', 'B', 'C', 'A', 'B', 'C']
print("3" in list3)         # 输出 False
for x in list3:
    print (x)               # 输出 1 2 3
```

**3. 列表的常用方法**

列表有一些常用的方法，我们可以获用len()得列表中元素的个数、用insert()插入元素、用append()在尾部添加元素、用pop()删除尾部的元素。

```py
list1 = ['a', 'b', 10, 10.1]
list2 = ['a', 'b', 'c']
list3 = ["A", "B", "C"]

print(len(list1))       # 输出 4

list2.append('d')       
print(list2)            # 输出 ['a', 'b', 'c', 'd']

list3.insert(0,"Z")
list3.pop()
print(list3)            # 输出 ['Z', 'A', 'B']
```

### 元祖

**1. 创建元祖** 

Python的元组与列表类似，不同之处在于元组的元素不能修改；元组使用小括号，列表使用方括号。元祖的创建，只需要在括号中添加元素，并使用逗号隔开即可

```py
tuple1 = ('A', 'B', 'C')
tuple2 = (1, 2, 3, 4, 5)
tuple3 = ("a", "b", "c")
tuple4 = ()                 # 创建一个空元祖
```


> **[!WARNING]**
> 元组中只包含一个元素时，需要在元素后面添加逗号，比如：
> 
> ```py
> tuples = ('A',)
> ```

**2. 访问元祖** 

元组与字符串类似，下标索引从0开始，可以进行截取、组合、用下标索引来访问元组中的值。元组中的元素值是一旦初始化就不能修改，因为不能修改所以不支持 append(), insert()这样的方法，但可以对元组进行连接组合；元组中的元素值是不允许删除的，但可以使用del语句来删除整个元组

```py
tuple1 = ('A', 'B', 'C')
tuple2 = (1, 2, 3, 4, 5)
tuple3 = (100,)

print(tuple1[0])            # 输出 A
print(tuple2[1:4])          # 输出 (2, 3, 4)

tuple4 = tuple1 + tuple3
print(tuple4)               # 输出 ('A', 'B', 'C', 100)

del tuple4
print(tuple4)

# 元祖被删除后，输出变量有异常信息
#  File "F:/python_study.py3", line 12, in <module>
#    print(tuple4)
#  NameError: name 'tuple4' is not defined
```


**3. 元祖的运算操作**

元祖同样也支持这些运算符，由于和前面两种运算符相似，这里不再举例。

操作符 | 语法
---|---
len | 获得元祖元素个数
+ | 组合
* | 重复输出元祖
in | 元素如果存在元祖中，返回True
for | 迭代

### 字典

**1. 创建字典**

字典是另一种可变容器模型，并且可存储任意类型的对象。
字典的每个键值对（key-value）用冒号分隔，每个键值对之间用逗号分隔，整个字典包括在花括号中。键一般是唯一的，如果重复，最后一个键值对就会替换前面的，值不需要唯一。

```py
dict1 = {k1:v1, k2:v2}
dict2 = {'a':1, 'b':2, 'a':3}

print(dict2['b'])               # 输出 2
print(dict2['a'])               # 输出 3
print(dict2)                    # 输出 {'a': 3, 'b': 2}
```

**2. 访问字典**

字典的值可以取任何数据类型，但键必须是不可变的。如果要访问字典里的值，只要把相应的键放入熟悉的方括号中即可。

```py
dict = {'name': 'xiaoming', 'age':18, 'gender':'male'}

print(dict['name'])             # 输出 xiaoming
print(dict['age'])              # 输出 18
print(dict['education'])

# 被访问字典里没有的对应的键数据时，则会输出错误
# Traceback (most recent call last):
#   File "F:/python_study4.py", line 5, in <module>
#     print(dict['education'])
# KeyError: 'education'
```

**3. 字典的运算操作**

修改字典键值对时，直接对键重新赋值即可；新增内容时，直接增加新键值对；删除单一元素和删除整个字典是，用del命令

```py
dict = {'name': 'xiaoming', 'age':2, 'gender':'male'}

dict['age'] = 20                        # 修改年龄     
dict['education'] = 'undergraduate'     # 添加学历

del dict['gender']                      # 删除性别
dict.clear()                            # 清空字典的所有键值对
del dict                                # 删除整个字典

print(dict)                             # 输出 <class 'dict'>
print(dict['age'])

# del后字典不存在，这里会输出一个错误
# Traceback (most recent call last):
#  File "D:/Pycharm projects/python_study/basic/study3.py", line 11, in <module>
#     print(dict['age'])
# TypeError: 'type' object is not subscriptable

```

### 集合

**1. 集合的创建、访问与操作**

集合 set 和字典 dictory 类似，不过它只是 key 的集合，不存储 value。同样可以增删查，增加使用 add，删除使用 remove，使用 in查询看某个元素是否存在这个集合中。

```py
s = set(['a', 'b', 'c'])
s.add('d')
s.remove('b')
print(s)                     # 输出 set(['a', 'c', 'd'])
print('c' in s)              # 输出 True
```

## 注释

- 单行注释： `#`

如果注释中有中文，一般会在代码前添加 `# -- coding: utf-8 -`。

- 多行注释：`'''` 或者 `"""`

多行注释，使用三个单引号，或者三个双引号


## 函数 def

函数代码块以 def 关键词开头，后接函数标识符名称和圆括号，在圆括号里是传进来的参数，然后通过 return 进行函数结果得反馈

```py
def magic(festival):
   return 2020 - festival

print magic(1024)          ## 输出 996
```

## 判断与循环语句

> [!NOTE]
> Python 有一个很大的特点，它不像其他语言一样使用 大括号`{ }` 或者 `begin…end` 来分隔代码块，而是采用代码缩进和冒号的方式来区分代码之间的层次关系。
> 
> 所以代码缩进在 Python 中是一种语法，如果代码缩进不统一，比如有的是 tab 有的是空格，就会产生错误或者异常。所以相同层次的代码一定要采用相同层次的缩进。

- 判断语句 if ... else ...

```py
str =  input("What's your final grade?")
score = int(str)
if score>= 90:
       print('Excellent')
else:
       if score < 60:
           print('Fail')
       else:
           print('Good Job')
```

- 循环语句 for ... in

```py
sum = 0
for number in range(100):
    sum = sum + number
print(sum)                   # 输出 4950
```

- 循环语句 while

```py
sum = 0
number = 1
while number < 100:
    sum = sum + number
    number = number + 1
print(sum)                   # 输出 4950
```