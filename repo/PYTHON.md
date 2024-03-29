[toc]

# 安装pip

https://www.runoob.com/w3cnote/python-pip-install-usage.html

# 常用数据结构

## 字符串

注意和C字符串的区别：字符串是不可修改的

```python
s1 = 'hello, world!'
s2 = "hello, world!"
# 以三个双引号或单引号开头的字符串可以折行
s3 = """
hello, 
world!
"""
print(s1, s2, s3, end='')
```

Python为字符串类型提供了非常丰富的运算符，我们可以使用`+`运算符来实现字符串的拼接，可以使用`*`运算符来重复一个字符串的内容，可以使用`in`和`not in`来判断一个字符串是否包含另外一个字符串（成员运算），我们也可以用`[]`和`[:]`--左闭右开，类似range(start,stop[, step])，range默认从0开始运算符从字符串取出某个字符或某些字符（切片运算）

```python
s1 = 'hello ' * 3
print(s1) # hello hello hello 
s2 = 'world'
s1 += s2
print(s1) # hello hello hello world
print('ll' in s1) # True
print('good' in s1) # False
str2 = 'abc123456'
# 从字符串中取出指定位置的字符(下标运算)
print(str2[2]) # c
# 字符串切片(从指定的开始索引到指定的结束索引)
print(str2[2:5]) # c12
print(str2[2:]) # c123456
print(str2[2::2]) # c246  2起始，间隔2
print(str2[::2]) # ac246 
print(str2[::-1]) # 654321cba
print(str2[-3:-1]) # 45  什么剑冢用法
```

常用字符串处理函数

```python
str1 = 'hello, world!'
# 通过内置函数len计算字符串的长度
print(len(str1)) # 13
# 获得字符串首字母大写的拷贝
print(str1.capitalize()) # Hello, world!
# 获得字符串每个单词首字母大写的拷贝
print(str1.title()) # Hello, World!
# 获得字符串变大写后的拷贝
print(str1.upper()) # HELLO, WORLD!
# 从字符串中查找子串所在位置
print(str1.find('or')) # 8 有个空格
print(str1.find('shit')) # -1
# 与find类似但找不到子串时会引发异常
# print(str1.index('or'))
# print(str1.index('shit'))
# 检查字符串是否以指定的字符串开头
print(str1.startswith('He')) # False
print(str1.startswith('hel')) # True
# 检查字符串是否以指定的字符串结尾
print(str1.endswith('!')) # True
# 将字符串以指定的宽度居中并在两侧填充指定的字符
print(str1.center(50, '*'))
# 将字符串以指定的宽度靠右放置左侧填充指定的字符
print(str1.rjust(50, ' '))
str2 = 'abc123456'
# 检查字符串是否由数字构成
print(str2.isdigit())  # False
# 检查字符串是否以字母构成
print(str2.isalpha())  # False
# 检查字符串是否以数字和字母构成
print(str2.isalnum())  # True
str3 = '  jackfrued@126.com '
print(str3)
# 获得字符串修剪左右两侧空格之后的拷贝
print(str3.strip())
```

格式化输出

````python
a, b = 5, 10
print('%d * %d = %d' % (a, b, a * b))
print('{0} * {1} = {2}'.format(a, b, a * b))
print(f'{a} * {b} = {a * b}')
````

## 列表

列表（`list`），也是一种结构化的、非标量类型，它是值的有序序列，每个值都可以通过索引进行标识，定义列表可以将列表的元素放在`[]`中，多个元素用`,`进行分隔，可以使用`for`循环对列表元素进行遍历，也可以使用`[]`或`[:]`运算符取出列表中的一个或多个元素。

```python
list1 = [1, 3, 5, 7, 100]
print(list1) # [1, 3, 5, 7, 100]
# 乘号表示列表元素的重复
list2 = ['hello'] * 3
print(list2) # ['hello', 'hello', 'hello']
# 计算列表长度(元素个数)
print(len(list1)) # 5
# 下标(索引)运算
print(list1[0]) # 1
print(list1[4]) # 100
# print(list1[5])  # IndexError: list index out of range
print(list1[-1]) # 100
print(list1[-3]) # 5
list1[2] = 300
print(list1) # [1, 3, 300, 7, 100]
# 通过循环用下标遍历列表元素
for index in range(len(list1)):
    print(list1[index])
# 通过for循环遍历列表元素
for elem in list1:
    print(elem)
# 通过enumerate函数处理列表之后再遍历可以同时获得元素索引和值
for index, elem in enumerate(list1):
    print(index, elem)
```

##元组

用一个变量来存储多个数据，元组的元素不能修改

```python
# 定义元组
t = ('骆昊', 38, True, '四川成都')
print(t)
# 获取元组中的元素
print(t[0])
print(t[3])
# 遍历元组中的值
for member in t:
    print(member)
# 重新给元组赋值
# t[0] = '王大锤'  # TypeError
# 变量t重新引用了新的元组原来的元组将被垃圾回收
t = ('王大锤', 20, True, '云南昆明')
print(t)
# 将元组转换成列表
person = list(t)
print(person)
# 列表是可以修改它的元素的
person[0] = '李小龙'
person[1] = 25
print(person)
# 将列表转换成元组
fruits_list = ['apple', 'banana', 'orange']
fruits_tuple = tuple(fruits_list)
print(fruits_tuple)
```

已经有了列表这种数据结构，为什么还需要元组这样的类型呢？

1. 元组中的元素是无法修改的，事实上我们在项目中尤其是[多线程](https://zh.wikipedia.org/zh-hans/多线程)环境（后面会讲到）中可能更喜欢使用的是那些不变对象（一方面因为对象状态不能修改，所以可以避免由此引起的不必要的程序错误，简单的说就是一个不变的对象要比可变的对象更加容易维护；另一方面因为没有任何一个线程能够修改不变对象的内部状态，一个不变对象自动就是线程安全的，这样就可以省掉处理同步化的开销。一个不变对象可以方便的被共享访问）。所以结论就是：如果不需要对元素进行添加、删除、修改的时候，可以考虑使用元组，当然如果一个方法要返回多个值，使用元组也是不错的选择。
2. 元组在创建时间和占用的空间上面都优于列表。我们可以使用sys模块的getsizeof函数来检查存储同样的元素的元组和列表各自占用了多少内存空间，这个很容易做到。我们也可以在ipython中使用魔法指令%timeit来分析创建同样内容的元组和列表所花费的时间，下图是我的macOS系统上测试的结果。

## set

已经有了列表这种数据结构，为什么还需要元组这样的类型呢？

1. 元组中的元素是无法修改的，事实上我们在项目中尤其是[多线程](https://zh.wikipedia.org/zh-hans/多线程)环境（后面会讲到）中可能更喜欢使用的是那些不变对象（一方面因为对象状态不能修改，所以可以避免由此引起的不必要的程序错误，简单的说就是一个不变的对象要比可变的对象更加容易维护；另一方面因为没有任何一个线程能够修改不变对象的内部状态，一个不变对象自动就是线程安全的，这样就可以省掉处理同步化的开销。一个不变对象可以方便的被共享访问）。所以结论就是：如果不需要对元素进行添加、删除、修改的时候，可以考虑使用元组，当然如果一个方法要返回多个值，使用元组也是不错的选择。
2. 元组在创建时间和占用的空间上面都优于列表。我们可以使用sys模块的getsizeof函数来检查存储同样的元素的元组和列表各自占用了多少内存空间，这个很容易做到。我们也可以在ipython中使用魔法指令%timeit来分析创建同样内容的元组和列表所花费的时间，下图是我的macOS系统上测试的结果。

## 生成式和生成器

```python
f = [x for x in range(1, 10)]
print(f)
f = [x + y for x in 'ABCDE' for y in '1234567']
print(f)
# 用列表的生成表达式语法创建列表容器
# 用这种语法创建列表之后元素已经准备就绪所以需要耗费较多的内存空间
f = [x ** 2 for x in range(1, 1000)]
print(sys.getsizeof(f))  # 查看对象占用内存的字节数
print(f)
# 请注意下面的代码创建的不是一个列表而是一个生成器对象
# 通过生成器可以获取到数据但它不占用额外的空间存储数据
# 每次需要数据的时候就通过内部的运算得到数据(需要花费额外的时间)
f = (x ** 2 for x in range(1, 1000))
print(sys.getsizeof(f))  # 相比生成式生成器不占用存储数据的空间
print(f)
for val in f:
    print(val)
```



# 一些疑惑

## python类型

###动态类型 VS 静态类型

python都没有声明语言的类型，如何工作，如 a = 3

动态类型：**动态类型语言**是指在运行期间才去做数据类型检查的语言。在用动态语言编程时，不用给变量指定数据类型，第一次赋值给变量时，在内部将数据类型记录下来。



从概念上来说，python会执行三个不同的步骤去完成这个请求:

\1. 创建一个对象代表值3
\2. 创建一个变量a，如果它还没有创建的话

\3. 将变量与新的对象3连接（引用，指针指向）

```
变量是一个系统表的元素，拥有指向对象的连接的空间
对象是被分配的一块内存，有足够的空间去表现他们所代表的值
引用是自动形成的从变量到对象的指针
```

### 强类型 VS 弱类型

**强类型的优点：**
1、编译时刻能检查出错误的类型匹配，以提高程序的安全性；
2、可以根据对象类型优化相应运算，以提高目标代码的质量；
3、减少运行时刻的开销。

**弱类型**

正好与强类型相反，编译时的检查很弱，它**仅能区分指令和数据**，弱类型语言允许变量类型的隐式转换，允许强制类型转换等，如字符串和数值可以自动转化。

###值传递 VS 地址传递

```python
a = "123"
b = a
a = "xyz"
```

执行第一句Python解释器创建字符串“123”和变量“a”,并把“a”指向“123”。

执行第二句，因为“a”已经存在，并不会创建新的对象，但会创建变量“b”,并把“b”指向“a”指向的字符串“123“。

![python1](.\img\python1.png)

执行第三句，首先会创建字符串“xyz”，然后把“xyz”的地址赋予“a“(“a”指向字符串“xyz”)。

![python2](.\img\python2.png)


我们可以通过调用id()方法查看变量所指向对象在内存中的地址。

Python参数传递统一使用的是**引用传递方式**。因为Python对象分为可变对象(list,dict,set等)和不可变对象(number,string,tuple等)，当传递的参数是可变对象的引用时，因为可变对象的值可以修改，因此可以通过修改参数值而修改原对象，这类似于C语言中的引用传递；当传递的参数是不可变对象的引用时，虽然传递的是引用，参数变量和原变量都指向同一内存地址，但是不可变对象无法修改，所以参数的重新赋值不会影响原对象，这类似于C语言中的值传递。

既然Python只允许引用传递，那有没有办法可以让两个变量不再指向同一内存地址呢？
Python提供了一个copy模块，帮助我们完成这件事。

![这里写图片描述](./img/pythoncpoy.png)

copy是浅拷贝，不管对象多么复杂，都只拷贝第一层，deepcopy是深拷贝，完全复制原变量的所有层的所有数据，在内存中生成一套完全相同的内容

### == VS is

== 是比较两个对象的内容是否相等，即两个对象的“值“”是否相等，不管两者在内存中的引用地址是否一样。