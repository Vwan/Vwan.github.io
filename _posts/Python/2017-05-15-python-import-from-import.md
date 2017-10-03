---
title: Python Import vs From （module) import function(class)
date: 2017-05-15 21:05:13 +0800
layout: post
categories:
- Python
tags:
- Python
---

# Python: Import vs From （module) import function(class)

本文涉及的 Python 基本概念：

- Module
- Class
- import
- from ... import

最近在学习Paython, 遇到一个问题，涉及到import 和 from ... import，module 和 class 的理解，解决方式是将import 替换成 from import, 但其实并非一个好的解决方法, 后来还是改回import。在这里稍微总结一下，以免再犯。

为方便解释，先简化一下问题，后面我会列出我遇到问题时用的代码，以供参考。

**问题 1 重现**

假设我们有两个 module，文件名分别是：module.py, animal.py。对于不太熟悉 python module 的同学，我在后面稍微解释了下。

module.py: 希望调用animal中 Animal类的方法

animal.py: 包含一个名为 Animal 的类

module.py

```
import animal

class Module(object):
	def test(self):
		animal.run()

if __name__=="__main__":
	Module().test()
```

animal.py

```
class Animal(object):
	def run(self):
		print "animal is running"
```

运行" python module.py"的时候出现错误：

```
Traceback (most recent call last):
  File "module.py", line 8, in <module>
    Module().test()
  File "module.py", line 5, in test
    animal.run()
AttributeError: 'module' object has no attribute 'run'
```

这是因为，在import animal 的时候，python 会创建一个对 模块名 animal 的引用，如果要访问该模块内其他方法或类时，需要写全，如 animal.attribute。

> Imports the module X, and creates a reference to that module in the current namespace. Then you need to define completed module path to access a particular attribute or method from inside the module (e.g.: X.name or X.attribute)

在我上面的例子中，我需要访问的是 animal 中 的 Animal 类中定义的方法 run。因为这个 run 方法是 Animal 类的实例方法，所以我还需要创建一个Animal实例才可以访问。

所以，将 module.py 中 第五行代码 "		animal.run()" 改为：

> animal.Animal().run()

再运行即可成功。

module.py 修改后的代码：

```
import animal

class Module(object):
	def test(self):
		animal.Animal().run()

if __name__=="__main__":
	Module().test()

```

对于刚接触 Python 的同学，有必要先了解一下 python 中 **module **的定义：

> A module is a file containing Python definitions and statements. The file name is the module name with the suffix .py appended

module 就是一个 后缀是 .py 的Python文件，文件名就是 module的名字，文件中可以定义一些函数方法或者class类，这个module可以通过 import 指令导入其他module，以便重用(reuse)。

比如我这里的例子中 animal.py，其中animal就是一个python module，里面定义了一个Animal类。module.py 文件中通过 import animal，导入animal，这样我便可以调用animal中Animal 类的方法。

**问题 2 重现**

现在我们再创建一个cat.py: 包含一个名为 cat 的类，是 Animal的子类

cat.py:

```
import animal

class Cat(animal):
	def run(self):
		print "cat is running"

```

我们修改一下 module.py, 让它调用 cat module 中的方法：

```
import cat

class Module(object):
	def test(self):
		cat.run()

if __name__=="__main__":
	Module().test()
```

现在我们运行 module.py 的话，有没有看出会出现什么问题？

```
Traceback (most recent call last):
  File "module.py", line 1, in <module>
    import cat
  File "C:\projects\Python\500lines\simple-web-server\simple-web-server\cat.py", line 3, in <module>
    class Cat(animal):
TypeError: Error when calling the metaclass bases
    module.__init__() takes at most 2 arguments (3 given)
```

是不是有点糊涂，我就被这个错误卡住了 "```module.__init__() takes at most 2 arguments (3 given)```"。毫无头绪，网上搜索了下，原因是 Class Cat(animal)，这里python 认为是要继承module animal
, 而不是class Animal，所以报错。

> 在cat.py中，import animal 语句告诉python，我们导入的是 animal module.

那为什么继承module的话会出现这种模糊不清的错误提示呢：```module.__init__() takes at most 2 arguments (3 given)```？

其实不是模糊不清，背后的逻辑请参考[What happens when you inherent from a module instead of a class in Python? - Stack Overflow](http://stackoverflow.com/questions/37876318/what-happens-when-you-inherent-from-a-module-instead-of-a-class-in-python?noredirect=1&lq=1)，解释的很清楚，下次遇到这种问题的时候就不会没有头绪了。

简单的解释就是：

python在遇到继承的时候，会内部调用```__init__```方法，如果是继承class的话，就会调用```type.__init__()``` 方法，该方法有三个参数；而如果是继承module的话，就会调用```module.__init__()```
方法，该方法只接受两个参数。所以就会出现上面那个错误：```module.__init__() takes at most 2 arguments (3 given)```

```
type(BaseClass).__init__(cls, name_of_subclass, (BaseClass,), dict_of_subclass)
# or simpler
type(BaseClass)(name_of_subclass, (BaseClass,), dict_of_subclass)
```

那为什么继承class 是调用```type.__init__```, 而module则是 ```module.__init__```呢？ 这就涉及到 python module 和 class 的区别了。我们可以使用 type() 方法在python shell中快速察看一下：
```
>>> import animal
>>> type(animal)
<type 'module'>
>>> from animal import Animal
>>> type(Animal)
<type 'type'>
```

当我们导入 module animal的时候，animal 的类型是module;
当我们从animal 导入 Animal的时候，Animal的类型是 type，也就是类;

现在我们的问题 2 知道原因何在了吧，我们需要Cat类继承Animal Class, 而不是 animal module。

从上面 "from animal import Animal" 语句我们可以看出，这样导入后我们可以直接调用Animal，而不需要animal.Animal。

现在修改一下cat.py:

```
from animal import Animal

class Cat(animal):
	def run(self):
		print "cat is running"
```
以及 module.py:

```
from cat import Cat

class Module(object):
	def test(self):
		Cat().run()

if __name__=="__main__":
	Module().test()
```

运行 python module.py，终于得到我们要的结果了

```
PS C:\projects\Python\500lines\simple-web-server\simple-web-server> python module.py
cat is running
```

大家可以看到，现在我们用的是 from...import...语句：

> from X import *
    Imports the module X, and creates references to all public objects defined by that module in the current namespace (that is, everything that doesn’t have a name starting with _) or whatever name you mentioned.

```*```号表示 X 中的所有非私有对象，除了```__```开头的。
如果用特定的对象取代 * ，比如from cat import Cat, 便是从module cat 导入 class Cat， 而非所有的module cat中的对象。

在使用这种导入的时候，module X 本身并没有设置，所以我们不能用 X.name 或 X.其他对象 来引用 module中的对象。我们直接使用那些对象即可。

至于 import module 和 from module import FC 哪种更好，则见仁见智，但普遍倾向于前者，毕竟一劳永逸，导入后用modulename.blahblah 可访问module中的任意可访问的对象，也符合python的namespace hirearchy理念。

所以，您试着再改回import X呗。这里就不贴代码了。

**我的实际问题**

我是跟着这个[500 Lines or Less | A Simple Web Server](http://aosabook.org/en/500L/a-simple-web-server.html)在学，简化一下项目的结构如下：

**server.py** 是主入口程序，其中有一个 Cases 列表，其中罗列的是多个class的实例(instance), 如 case_no_file() 是 class case_no_file的实例。

**cases.py**中定义了这些列表 item class 如 case_no_file，其父类是 base_case

**base_case.py** 中定义了父类 base_case

部分代码如下：

* Server.py:

```
import BaseHTTPServer
import cases

class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    Cases = [case_no_file(),
             case_cgi_file(),
             case_existing_file(),
             case_directory_index_file(),
             case_directory_no_index_file(),
             case_always_fail()]
```

* base_case.py:

```
class base_case(object):
    '''Parent for case handlers.'''

    def handle_file(self, handler, full_path):
        try:
            with open(full_path, 'rb') as reader:
                content = reader.read()
            handler.send_content(content)
        except IOError as msg:
            msg = "'{0}' cannot be read: {1}".format(full_path, msg)
            handler.handle_error(msg)
```

* cases.py:

```
import base_case

class case_existing_file(base_case):
    '''File exists.'''

    def test(self, handler):
        return os.path.isfile(handler.full_path)

    def act(self, handler):
        self.handle_file(handler, handler.full_path)

```

运行 "python server.py"的时候，报错了：

```
Traceback (most recent call last):
  File "server.py", line 2, in <module>
    import cases
  File "c:\projects\python\500lines\simple-web-server\simple-web-server\cases.py", line 4, in <module>
    class case_existing_file(base_case):
TypeError: Error when calling the metaclass bases
    module.__init__() takes at most 2 arguments (3 given)
```

根据前面的分析，修改 server.py 和 cases.py如下后，运行正常：这里仍然用的是 import module。

server.py:

```
import cases

class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    Cases = [cases.case_no_file(),
            cases.case_cgi_file(),
            cases.case_existing_file(),
            cases.case_directory_index_file(),
            cases.case_directory_no_index_file(),
            cases.case_always_fail()
			]
```

cases.py:

```
import base_case

class case_existing_file(base_case.base_case):
    '''File exists.'''

    def test(self, handler):
        return os.path.isfile(handler.full_path)

    def act(self, handler):
        self.handle_file(handler, handler.full_path)
```

总结：

1 module 是什么

> A module is a file containing Python definitions and statements. The file name is the module name with the suffix .py appended

2 module 和 class 的区别：

> module 包含 class
> module 只是一个实例，不能继承。 class可以有多个实例，可以单继承，多继承
> module 名一般小写，class名一般大写
> 继承时如果传入的是module, 提示的错误是```module.__init__方法调用参数有误```，比较模糊，具体分析见上文。

3 import 和 from ... import的区别：
> import 导入的是 module
> from a import b, 是从 module a 中导入 b, b可以是 module, class, function, variable
> 一般建议用import module;

**参考资料**

[python - `from ... import` vs `import .` - Stack Overflow](http://stackoverflow.com/questions/9439480/from-import-vs-import)

[What happens when you inherent from a module instead of a class in Python? - Stack Overflow](http://stackoverflow.com/questions/37876318/what-happens-when-you-inherent-from-a-module-instead-of-a-class-in-python?noredirect=1&lq=1)

欢迎一起讨论学习！
