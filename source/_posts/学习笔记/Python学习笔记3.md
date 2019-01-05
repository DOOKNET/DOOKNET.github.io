---
title: Python学习笔记3
date: 2019-01-05 16:35:07
categories:
- 学习笔记
tags:
- Python
- 笔记
---

## 程序设计中的三种基本结构

- 顺序结构：程序按先后顺序，前一步执行结束之后，才能执行后一步。
- 选择结构：又称分支结构，指程序流程可以分为几条不同的路径执行。
- 循环结构：指程序流程重复执行某一项任务。

### 理解程序设计中的三种基本结构

- 问题：任意输入三个以逗号分割的数字，按大小顺序输出。
> 分析：三个数输入后，需要比较两个数字的大小，必要时需要交换两个数字。

```
# 数字比较大小
s = input('x,y,z = ')
x,y,z = s.split(',')    #把字符串用逗号进行分离，返回子串构成的列表
if x>y:
    x,y = y,x    #交换x,y值
if x>z:
    x,z = z,x
if y>z:
    y,z = z,y
print(x,y,z)
```

- 判断输入的数字是否为偶数
> 输入一个整数，如果是奇数，输出“奇数”，否则输出“偶数”

```
while True:
    try:
        num=int(input('输入一个整数：')) #判断输入是否为整数
    except ValueError: #不是纯数字需要重新输入
        print("输入的不是整数！")
        continue
    if num%2==0:
        print('偶数')
    else:
        print('奇数')
    break
```

---

## 程序设计中的循环结构

循环结构是一种重复执行的程序结构。

- 计算1+2+3...+100，这是一个级数求和问题，需要重复执行100次，对100个数依此进行累加
- 假设1个班级中有n名同学，统计男同学和女同学各有多少名
- 给定2个整数，求它们的最大公约数和最小公倍数

---

## while循环结构的定义

```
while 表达式:
    循环体
```

```
while 表达式:
    循环体
else:
    else子句
```

### 读取整形数值到负数结束

从键盘输入若干正整数，求所有输入整数之和。当输入整数为负数时，结束该操作。
> 分析：由于不确定用户即将输入几个正整数，因此属于不确定循环次数的问题。

```
# -*- coding: cp936 -*-
print('输入：')
s = 0
x = input("请求输入一个整数：")
while x>=0:
    s = s+x
    x = input("请求输入一个整数：")
print'整数之和='.s
```

### 输出水仙花数

输出“水仙花数”，所有水仙花数是指1个3位的十进制制数，其各位数字的立方和等于该数本身。例如：153是水仙花数，因为153=1^3+5^3+3^3

---
---

## python函数式编程常用函数——reduce

- reduce()函数用于将指定序列中的所有元素作为参数按一定的规则调用制定函数。reduce函数的语法如下：
> 计算结果 = reduce(映射函数,序列)

映射函数的必须有2个参数。reduce()函数首先以序列的第1和第2个元素为参数调用映射函数，然后将返回结果与序列的第3个元素为参数调用映射函数。以此类推，直至应用到序列的最后一个元素，将计算结果作为reduce()函数的返回结果。

---
---

## 使用序列存储数据简化操作

- 列表是一种数据结构
- sort()是python中的方法，可以对列表中数据排序

---

## 序列的定义

- 序列：一系列按一定顺序排列的数据。
- 位置编号，也称“下标”或“索引”，是整数或整数表达式。
- 引用元素：序列名[位置编号]，位置编号从0开始。
- 序列也可以从尾部访问，最后一个元素是c[-1]。

---

## 列表的定义

- 列表是python中内置数据类型，是数据的有序集合，其中，每一个数据称为元素。
- 一个列表中的数据类型可以各不相同。
- 列表中的其所有元素用逗号分割并放在一队中括号“[”“]”中
  - [10，20，30，40] #所有元素都是整型数据的列表
  - ['crunchy frog','ram bladder','lark vomit'] #所有元素都是字符串的列表
  - ['spam',2.0,5[10,20]] #该列表中包含了一个字符串元素、一个浮点类型元素、一个整型元素和一个列表类型元素

---

## 列表的创建和读取

- 创建列表：使用“=”将一个列表赋值给变量。
- 读取元素：用变量名加元素序号（放中括号中）即可访问列表中某个元素，列表的第一个元素序号为0。若一个列表有n个元素，则访问元素的合法序号范围是n~n-1。

---

## 使用列表进行查询

```
>>> a_list=['a','b','mpilgrim','z','example']
>>> print(a_list[2])
mpilgrim
>>> print(a_list[-1])
example
>>> print(a_list[-5])
a
>>> print(a_list[-7])

Traceback (most recent call last):
  File "<pyshell#4>", line 1, in <module>
    print(a_list[-7])
IndexError: list index out of range
>>> print(a_list[5])

Traceback (most recent call last):
  File "<pyshell#5>", line 1, in <module>
    print(a_list[5])
IndexError: list index out of range
```

---

## 列表的切片

- 切片：可以使用列表序号对来截取列表中的任何部分从而得到一个新列表。序号对中第一个序号表示切片截止（但不包含）位置。
- 当切片的左索引为0时可缺省，当右索引为列表长度时也可缺省。

### 使用字典进行查询

```
>>> a_list=['a','b','mpilgrim','z','example']
>>> print(a_list[1:3])
['b', 'mpilgrim']
>>> print(a_list[1:-1])
['b', 'mpilgrim', 'z']

>>> print(a_list[:3])
['a', 'b', 'mpilgrim']
>>> print(a_list[3:])
['z', 'example']
>>> print(a_list[:])
['a', 'b', 'mpilgrim', 'z', 'example']
```
---

## 列表元素的增加

- 使用“+”将一个新列表附加在原列表的尾部
- 使用append()方法向列表尾部添加一个新元素
- 使用extrnd()方法将一个列表添加在原列表的尾部
- 使用insert()方法将一个元素插入到列表的任意位置

```
>>> a_list = [1]
>>> a_list = a_list + ['a',2.0]
>>> a_list
[1, 'a', 2.0]
>>> a_list,append(True)

>>> a_list.append(True)
>>> a_list
[1, 'a', 2.0, True]

>>> a_list.extend(['x',4])
>>> a_list
[1, 'a', 2.0, True, 'x', 4]

>>> a_list.insert(0,'x')
>>> a_list
['x', 1, 'a', 2.0, True, 'x', 4]
```

---

## 列表元素的检索

- 使用count()方法计算列表中某个元素出现的次数
- 使用in运算符返回某个元素是否在该列表中
- 使用index()方法返回某个元素在列表中的准确位置

```
>>> a_list=['x',1,'a',2.0,True,'x',4]
>>> a_list.count('x')
2
>>> 3 in a_list
False
>>> 2.0 in a_list
True
>>> a_list.index('x')
0
>>> a_list.index(5)

Traceback (most recent call last):
  File "<pyshell#17>", line 1, in <module>
    a_list.index(5)
ValueError: 5 is not in list
>>> 
```

---

## 列表元素的删除

- 当向列表中添加或删除元素时，列表将自动拓展或收缩，列表中永远不会有缝隙。
- 使用del语句删除某个特定位置的元素
- 使用remove方法删除某个特定值的元素
- 使用pop(参数)方法来弹出（删除）指定位置的元素，缺省参数时弹出最后一个元素

```
>>> a_list=['x',1,'a',2.0,True,'x',4]
>>> del a_list[1]
>>> a_list
['x', 'a', 2.0, True, 'x', 4]
>>> a_list.remove('x')
>>> a_list
['a', 2.0, True, 'x', 4]
>>> a_list.remove('x')
>>> a_list
['a', 2.0, True, 4]
>>> a_list.remove('x')

Traceback (most recent call last):
  File "<pyshell#24>", line 1, in <module>
    a_list.remove('x')
ValueError: list.remove(x): x not in list
>>> a_list.pop()
4
```

---

## 列表中的常用函数

- cmp( )

格式：cmp(列表1,列表2)

功能：对两个列表进行比较，若第一个列表大于第二个，则结果为1，相反则为-1，元素完全相同则结果为0。

- len( )
  
格式：len(列表)

功能：返回列表中的元素个数

- max( )和min()

格式：max(列表)，min(列表)

功能：返回列表中的最大或最小元素

- sorted()和reversed()

格式：sorted(列表)，reversed(列表)

功能：前者的功能是对列表的后面增加一个reverse参数，其等于True则表示按降序排列；后者的功能是对列表进行逆序。

- sum()

格式：sum()

功能：对数值型列表的元素进行求和运算，对非数值型列表运算则出错。

---

## 循环输出列表中元素的值

每循环一次输出一个列表元素值，由于列表定义后，其长度是已知的，因此循环次数也是确定的。

```
# -*- coding: cp936 -*-
a_list=['a','b','c','d','e']
a_len=len(a_list)
i=0
while i<a_len:
    print'列表的第',i+1,'个元素是：',a_list[i]
    i +=1
    
运行结果：
列表的第 1 个元素是： a
列表的第 2 个元素是： b
列表的第 3 个元素是： c
列表的第 4 个元素是： d
列表的第 5 个元素是： e
```

---

## 列表中元素排序

列表Li中有一组单词，把单词分别进行升序排序和降序排序。

应用列表的排序函数sort()能完成升序排序和降序排序。

```
# -*- coding: cp936 -*-
Li=['apple','peach','wps','word','access','excel','open','seek']
Li2=Li[:]
print Li
Li.sort()   #列表元素按升序排列
print'升序：'
print Li
print Li2
print'降序：'
Li2.sort(reverse=True)  #列表元素按降序排列
print Li2

运行结果：
['apple', 'peach', 'wps', 'word', 'access', 'excel', 'open', 'seek']
升序：
['access', 'apple', 'excel', 'open', 'peach', 'seek', 'word', 'wps']
['apple', 'peach', 'wps', 'word', 'access', 'excel', 'open', 'seek']
降序：
['wps', 'word', 'seek', 'peach', 'open', 'excel', 'apple', 'access']
```

---

## 多维列表

- 定义一个二维数列表
  - list2=[["CPU","内存"],["硬盘","声卡"]];
- 读取一个二维列表中的元素
  - 列表名[索引1] [索引2]

### 循环输出列表中元素的值

```
list2=[["CPU","内存"],["硬盘","声卡"]];
for i in range(len(list2)):
    list1=list2[i];
    for j in range(len(list1)):
        print(list1[j])

运行结果：
CPU
内存
硬盘
声卡
```

---
---

## 元组

- 元组和列表类似，但其元素是不可变的，元组一旦创建，用任何方法都不可修改其元素。
- 元组的定义方式和列表相同，但定义时所有元素是放在一对圆括号“(”和“)”中，而不是方括号中。
- 下面这些都是合法的元组：(10,20,30,40)('crunchy frog','ram bladder','lark vomit')

---

## 元组的操作

- 创建元组：使用“=”将一个元组赋值给变量。
- 读取元素
- 元组切片

### 列表元素的排序

---

## 元组的检索

- 检索元素

使用count( )方法计算元组中某个元素出现的次数

---

## 元组元素的排序

---

## 元组与列表的区别

- 元组中的数据一旦定义就不允许更改。因此，元组没有append()或extend()方法，

---

## 利用元组来一次性对多个变量赋值

```
>>> v_tuple = (False,3.5,'exp')
>>> (x,y,z) = v_tuple
>>> x
False
>>> y
3.5
>>> z
'exp'
```

---
---

## 字典

- 字典是键值对的无序集合。
- 字典中的每个元素包含两部分：键和值。向字典添加一个键的同时，必须为该键增添一个值。

---

## 字典的创建和查找

- 创建字典

- 字典定义后

- 遍历字典

```
>>> a_dict = {'server':'db.diveintopython3.org','database':'mysql'}
>>> a_dict
{'database': 'mysql','server':'db.diveintopython3.org'}
```

---

## 使用字典进行查询

---

## 字典的更新

- 添加和修改字典
  - 字典没有预定义的大小限制。
  - 可以随时向字典中添加新的键值对，或者修改现有键所关联的值。
  - 添加和修改的方法相同，都是使用“字典变量名[键名]=键值”的形式，区分究竟是添加还是修改是看键名与字典中现有的键名是否重复，因为字典中不允许有重复的键。如不重复则是添加新键值对，如重复则是将该键对应的值修改为新值。

```

```

---

## 字典的其它操作

- 字典的长度

与列表、元组类似，可以用len()函数返回字典中键的数量。

- 字典检索

- 删除元素和字典

```

```

---

## 列表作为函数参数

```
def sum(list):
    total = 0;
    for x in range(len(list)):
        print(list[x],"+");
        total+=list[x];
    print("=",total);
list = [15,25,35,65]
sum(list);

#运行结果
==========RESTART:E:\Workspace\Python_2.7\2018_8_22_Test\test.py==========
(15, '+')
(25, '+')
(35, '+')
(65, '+')
('=', 140)
```

```
def print_dict(dict):
    for(k,v) in dict.items():
        print"dict[%s]="%k,v
dict = {"a":"apple","b":"banana","g":"grape","o":"orange"}
print_dict(dict);

#运行结果
========== RESTART:E:\Workspace\Python_2.7\2018_8_22_Test\test.py==========
dict[a]= apple
dict[b]= banana
dict[o]= orange
dict[g]= grape
```

---

## 函数中修改列表参数

---

## 函数中修改字典参数

---

## 集合的创建和访问

```

```

---

## 集合元素的增加

---

## 集合元素的删除

```
>>> s = set([1,2,3])
>>> s.remove(1)
>>> print(s)
set([2, 3])
>>> s.clear()
>>> print(s)
set([])
```

---

## 集合的运算

### 子集和超集

```

```

---
---

## 复杂的数据结构

- 在解决实际问题时，还经常需要用到其他复杂的数据结构，如堆、栈、队列、树、图等。