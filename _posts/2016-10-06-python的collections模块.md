---
title: "Python collections 模块"
date: 2016-10-06 18:38
categories: Python
---

在Python标准库中有一些很好用的模块，一直没用过，看Requests源码的时候，看到collections模块，
感觉很好用，于是就屁颠屁颠的撸了标准库。在此做个笔记。
并且我在[这里](http://python.usyiyi.cn/translate/python_352/library/collections.html)翻译了部分文档


--------------------------

这个模块实现了专业的容器数据类型用来替代Python的通用内置容器，字典, 列表, 集合, 和 元组.

|||
|-----|------|
|[namedtuple()](#namedtuple)    |这是个工厂方法,生成可以使用名字来访问元素内容的tuple子类|
|[deque](#deque)                        |双端队列|
|[ChainMap](#ChainMap)           |为多个映射创建单个视图的类字典类型,可以说是合并多个字典,但和update不同|
|[Counter](#Counter)                  |计数器，用于计算可哈希对象的个数，是dict的子类|
|[OrderedDict](#OrderedDict)     |有序字典，能记录着数据成员添加的顺序|
|[defaultdict](#defaultdict)           ||
|
 

-------------

# namedtuple
原型:

```Python
collections.namedtuple(typename, field_names, verbose=False, rename=False)
```

这个工厂方法返回一个叫做 _typename_ 的tuple子类,这个新的子类用来创建一个类tuple(tuple-like)的对象,
这个对象拥有可以通过属性访问的字段，并且可以通过下标索引和迭代。field\_names 是一个单独的字符串，这个字符串中包含的所有字段用空格或都好隔开，
例如 'x y' 或 'x, y'.另外, field\_names 也可以是字符串的列表，例如 ['x', 'y'].

除了以下划线开头的名称，任何有效的 Python 标识符都可用于 fieldname 。

如果 rename参数 为 True, 无效的字段名会被自动转换成位置的名称。例如, ['abc', 'def', 'ghi', 'abc'] 将被转换为 ['abc', '\_1', 'ghi', \'_3']
这是因为def是关键字，而abc为重复的字段名。

如果verbose 为 True, 在类被建立后将打印这个类的源代码,(这是个过时的选项).
*namedtuple* 的实例中并没有字典, 所以 *namedtuple*并不会比常规的tuple消耗更多的内存，它是轻量级的.

**下面是namedtuple的用法**

{% highlight python %}

>>> # Basic example
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(11, y=22)     # instantiate with positional or keyword arguments
>>> p[0] + p[1]             # indexable like the plain tuple (11, 22)
33
>>> x, y = p                # unpack like a regular tuple
>>> x, y
(11, 22)
>>> p.x + p.y               # fields also accessible by name
33
>>> p                       # readable __repr__ with a name=value style
Point(x=11, y=22)

{% endhighlight %}


**namedtuple 在为 csv or sqlite3 模块返回的元组命名显得十分有用:**


{% highlight python %}

EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)

{% endhighlight %}


怎么样很棒吧？ 

另外除了从tuple继承而来的方法，namedtuple还支持另外三个方法和两个属性。
为了避免和字段冲突，这些方法和属性都以下划线开头。分别是

{% highlight python %}

# 类方法。从现有的列表或迭代器创建一个新的实例
classmethod somenamedtuple._make(iterable)

>>> t = [11, 22]
>>> Point._make(t)
Point(x=11, y=22)

# 返回一个有序字典，每个键对应于该字段的值
somenamedtuple._asdict()

>>> p = Point(x=11, y=22)
>>> p._asdict()
OrderedDict([('x', 11), ('y', 22)])

# 返回一个新的namedtuple,并把对应的值给替换了
somenamedtuple._replace(**kwargs)
>>> p = Point(x=11, y=22)
>>> p._replace(x=33)
Point(x=33, y=22)


somenamedtuple._source
somenamedtuple._fields  # 字段名称的元组

{% endhighlight %}


-------------------

# deque  双端队列

双端队列，最大的好处就是在获取两端的元素的效率极高，你可能会说，原生的list也可以从头部添加和取出对象啊？就像这样：

{% highlight python %}

l.insert(0,v)
l.pop()

{% endhighlight %}


这里list对象l两种方法的复杂度是O(n),而deque则是O(1),所以当代码中有这样的需求时,一定要记得使用deque。

而且deque提供了一些比较好用的方法，例如rotate(n), 我们可以做一个跑马灯


{% highlight python %}

# coding: utf-8

import sys
import time
from collections import deque

fancy_loading = deque('>--------------------')

while True:
    print('\r%s' % ''.join(fancy_loading),end='')
    fancy_loading.rotate(1)
    sys.stdout.flush()
    time.sleep(0.08)

# Result:

# 一个无尽循环的跑马灯
------------->-------

{% endhighlight %}


## deque 的食用技巧

deque 这么好的数据结构，既是线程安全的，又有很高的效率，食用起来肯定很爽。
下面介绍几种食用技巧：

- 利用有界双向队列来实现Unix中的tail功能。

    {% highlight python %}

    def tail(filename, n=10):
        """Return the last n lines of a file"""
        with open(filename) as f:
            return deque(f, n)

    {% endhighlight %}


- 计算[移动平均数](https://zh.wikipedia.org/wiki/%E7%A7%BB%E5%8B%95%E5%B9%B3%E5%9D%87)，这个在金融量化上好像能用。其实想法是保留最近的添加的n个数据

    {% highlight python %}

    def moving_average(iterable, n=3):
        # moving_average([40, 30, 50, 46, 39, 44]) --> 40.0 42.0 45.0 43.0
        it = iter(iterable)
        d = deque(itertools.islice(it, n-1))
        d.appendleft(0)
        s = sum(d)
        for elem in it:
            s += elem - d.popleft()
            d.append(elem)
            yield s / n

    {% endhighlight %}

正确的食用技巧可以从中享受Python带来的简洁。 爽！

-----------------