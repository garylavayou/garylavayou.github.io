# Python基础

Python是一种解释型、面向对象、动态数据类型的高级程序设计语言。

Python 本身也是由诸多其他语言发展而来的,这包括ABC、 Modula-3、 C、 C++、 Algol-68、 SmallTalk、 Unix shell 和其他的脚本语言等等。

## 环境配置

### Python解释程序

#### 运行Python程序

##### (1) 交互式编程

进入Python解释器环境，交互式执行命令语句。
```shell
$ python [options]  # enter python environment
```

`-q`：进入交互式命令环境后，不输出提示信息。

安装`pyreadline`可以为终端启用`tab`命令自动补全功能。

##### (2) 脚本式编程

**脚本是语句的集合**，Python解释器会创建一个Python运行环境从而执行脚本中的语句。脚本执行完后，解释器退出运行环境。

```shell
python [options] -c "python_command"  # 命令使用";"分隔
python [options] script.py [args] # execute python script
python [options] -m module_name [args] # 执行指定模块的内容（__main__）
```

`-E`：忽略所有`PYTHON*`[环境变量](https://docs.python.org/3/using/cmdline.html#environment-variables)（[运行环境](#运行环境)）。

当Python脚本首行指定了Python解释器路径，且该脚本具有可执行权限时，可直接运行该脚本。

```python
#!/usr/bin/python   # 指定执行该脚本的程序
#!/usr/bin/env python  # 从路径中查找Python解释器以执行该脚本
```

```shell
python -m json.tool demo.json  # 格式化JSON文本
python -m http.server 8080
python -m pydoc -p 8088   # python文档
python -m mimetypes filename
python -m tarfile -c demo.tar demo   # tar
python -m gzip filename   # => input only file, output filename.gz
python -m zipfile -c demo.zip demo
python -m telnetlib -d 192.168.56.200 22
```

##### [运行环境](Python系统编程.md#Python运行环境)

- `PYTHONPATH`用于指定除系统的附加库搜索路径，程序启动后将加载到`sys.path`中。默认搜索顺序为当前路径、用户指定附加搜索路径、Python内置库路径、第三方Python库路径。
- `PYTHONHOME`指定Python标准库位置（`prefix/lib/pythonversion` and `exec_prefix/lib/pythonversion`）；当`PYTHONHOME`为单个路径时，代替`prefix`和`exec_prefix`；反之，可以将`PYTHONHOME`设置为`'prefix:exec_prefix'`。`PYTHONHOME`==不是Python环境的安装目录==。
- `PYTHONSTARTUP`：启动Python Shell时需要执行的脚本路径。

> 如果使用[虚拟环境](Python开发环境.md#虚拟Python环境)，则通过激活命令（如`conda activate`）可保证相关环境变量正确设置。

## 基本语法

### 标识符

所有标识符可以包括英文、数字以及下划线(_)，但不能以数字开头，**区分大小写**。

以下划线开头的标识符是有特殊意义的。

- 以单下划线开头（`_foo`）的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用 `from xxx import *` 而导入；
- 以双下划线开头（`__foo`）代表类的私有成员；
- 以双下划线开头和结尾的（`__foo__`）代表 Python 里特殊方法专用的标识，如`__init__()`代表类的构造函数。

### 运算符

数值运算符参看[数值计算](./Python数值计算#运算符)。

#### 逻辑运算符

逻辑运算符用于控制流程中的条件语句，包括`and` 、`or` 、`not`。

> Python不支持`&&`，`||`。

#### 成员运算符

`in`， `not in`：在指定的容器（序列、元组、字典、集合等）中查找到值，返回值`True`或`False`；==也可用于判断一个字符串是否为另一个字符串的子串。==

```python
list_str = ['--test', '-o output/test', '--debug']
if '--debug' in list_str:
    print('find --debug in parameter list.')
if 'test' not in list_str:
    print('not find test in parameter list.')    
```

> 取决于容器类型，查询[时间复杂度](https://wiki.python.org/moin/TimeComplexity)不同。

#### 身份运算符

`is,` `is not`：判断两个标识符是否引用同一个对象，返回`True`或`False`。 `==`用于判断引用变量的==值==是否相等。

> `id()`函数返回一个整数表示变量的标识；CPython实现的`id(x)`返回`x`的内存地址。
>
> `hash(obj)`返回基于对象内容的一个映射值，具有相同值的两个对象有相同hash值（不同于[消息摘要](#密码学)是基于字节序列内容的）。

#### 对象运算符

`+` 运算符用于拼接序列对象；

`*` 用于重复序列对象：如果对象的元素为值类型，则复制该元素的值并将复制内容拼接；如果**元素为引用类型，则仅复制引用（浅拷贝）**。

```python
print str*2       # 输出字符串两次
```

`[]`：[下标运算符](#序列类型的索引)，取字符串、列表或元组元素。

#### 运算符优先级



### 语句

同一行显示多条语句，方法是用分号“` ;`” 分开。但是我们可以**使用斜杠（** **`\`）将一行的语句分为多行显示**。语句中包含`[]`,`{}`或`()` 括号就不需要使用多行连接符。

用缩进（空格）长度来写语句块。缩进的空白数量是可变的，但是所有代码块语句必须包含相同的缩进空白数量，这个必须严格执行。

`eval`可以执行字符串表示的Python表达式（不支持复杂的代码逻辑，例如赋值操作、循环语句），并返回表达式的值。

```python
x = eval('os.path.abspath(os.path.curdir)')
```

`exec`执行语句或代码块，不能做表达式求值并返回，但可以通过赋值表达定义新的变量并添加到当前上下文中。

```python
exec('x = os.path.abspath(os.path.curdir)')
```

https://www.cnblogs.com/pythonista/p/10590682.html。

### 注释和文档

单行注释：“`#`”

文件开始的注释内容提供解释器与脚本的相关信息。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
```

脚本中包含中文时，需要指定文件的编码方式（UTF-8），默认编码方案是（ASCII）。Python3.X 源码文件默认使用utf-8编码，所以可以正常解析中文，无需指定 UTF-8 编码。

空行并不是Python语法的一部分。书写时不插入空行，Python解释器运行也不会出错。

位于类型定义之后的[三引号文本](#字符串（String）)（`'''`，docstring）将自动生成为该类型的[文档](https://realpython.com/documenting-python-code/)（`type.__doc__`）。

> Package docstrings should be placed at the top of the package’s `__init__.py` file. Module docstrings are placed at the top of the file even before any imports. 

使用`help()`函数可查看当前环境已导入的模块的内容的文档。

##### 文档生成

使用reStructuredText编写代码注释，可通过[Sphinx](../应用软件/文档生成软件.md#解析和转换Python代码)自动转换为参考文档。

### 流程控制

#### 条件

```python
if expression1:
	statements1……
elif expression2:
	statements2……
else:
	statements4……
    
c = a if <condition> else b
```

在Python中没有`switch–case`语句。

#### 循环

```python
while expression1：
	statements……
else:
	statements4……
```

`for`循环可以遍历任何序列的项目，例如字符串、列表；

```python
for i = 1 to 10:
  statements
for var in sequence:  # 迭代过程不能更改迭代的对象
	statements1(s)
	break
	continue
else:
	statements2……
for idx,item in enumerate(lista): # enumerate构造(i, l(i))迭代元组
  print('{idx} - {item}')
for idx,_ in enumerate(lista):
  print('{idx} - {item}')
```

循环**正常**执行完之后，执行`else`语句。

`sequence`可以是索引集合：`range(start,end, step=1)`

#### pass语句

`pass`不做任何事情，一般用做占位语句。特殊变量`...`（`Ellipse`），可用于代替`pass`语句。

#### 异常处理

##### 触发异常

```python
raise ExceptionObject, args, traceback
```

##### 捕捉异常

```python
try:
  pass
except ExceptionName as e: # e为异常对象，如果不使用可省略as语句
  statements1
except (Exception1, Exception2, ...) as e:
  statements2
except:
	statements3
else:
	statements4
finally:
    statementsN
```

> Python 2.x语法：
>
> ```python
> try:
> 	statements
> except ExceptionType, Argument:
> 	statements
> ```

##### 异常处理方法

处理异常时可使用`traceback`获取调用栈的信息。打印堆栈追踪信息：

```python
traceback.print_exc()                 # 打印stacktrace
traceback.format_exc()                # 返回stacktrace为字符串
traceback.print_tb(err.__traceback__) # stacktrace
```

对于捕获到的异常，如果没有合适处理方法可再次抛出异常。

##### 异常类型

所有异常基于`BaseException`。`Exception`类用于定义用户异常。`Warning`也继承自`Exception`类，但通常不用于触发异常，而是用于[产生警告信息](Python输入输出.md#警告信息)。

Exception hierarchy：https://docs.python.org/3/library/exceptions.html#os-exceptions

内置异常类型：

ArithmeticError

BufferError

LookupError： IndexError, KeyError

AssertionError

EOFError

GeneratorExit

ImportError

ModuleNotFoundError

KeyboardInterrupt

MemoryError

NameError, TypeError, UnboundLocalError, ValueError, IOError, WindowsError

NotImplementedError

SystemError, SystemExit, EnvironmentError

OSError

> https://docs.python.org/3/library/exceptions.html#os-exceptions

OverflowError, ZeroDivisionError

RecursionError

ReferenceError

RuntimeError

StopIteration, StopAsyncIteration

SyntaxError, IndentationError, TabError

UnicodeError, UnicodeEncodeError, UnicodeDecodeError, UnicodeTranslateError

#### with-as (Context Manager)

[上下文管理协议](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/index.html)：实现方法是为一个类定义`__enter__`和`__exit__`两个函数。

```python
with open_resource(args) [as target(s)]:
    do_something
```

`with-as`语句的执行过程是，首先执行`__enter__`函数，它的返回值会赋给`as`后面的变量。
 然后开始执行`with-block`中的语句，在`with-block`执行完成或发生异常或退出，会执行`__exit__`函数（释放资源或处理可能产生的异常）。

##### 支持上下文管理协议的类

```python
class Resource():
  def __enter__(self):  # 实际分配资源的操作在该方法中，而不在__init__()中
    print('===connect to resource===')
    return self
  def __exit__(self, exc_type, exc_val, exc_tb):
    print('===close resource connection===')
    return True
```

##### `contextlib.contextmanager`装饰器

使用该[装饰器](Python高级编程.md#装饰器)可将构造资源的函数转换为支持上下文管理协议。

```python
import contextlib
@contextlib.contextmanager
def open_func(file_name):
  # __enter__ method
  file_handler = open(file_name, 'r')
  try:
	  yield file_handler
  # __exit__ method
  except Exception as exc: # 如果不需处理异常则无需使用try-except语句
    pass
  finally:
	  file_handler.close()
```



#### Exit handlers

`atexit`模块用于注册程序==正常退出==前的清理函数。可以注册多个函数，函数执行顺序与注册顺序相反。

```python
atexit.register(func, *args, **kwargs)
atexit.unregister(func)
```

如果子进程由`os.fork()`创建，则继承父进程的退出处理函数；如果子进程由`multiprocessing`模块创建，则不会继承退出处理函数。

> *When used with C-API subinterpreters, registered functions are local to the interpreter they were registered in.*

### 函数

#### 定义函数

```python
def function_name(arg1: int, arg2: float) -> float
    "documentation"
    statements
    [return expr1, expr2, ...]
```

函数内部可访问全局脚本语句定义的变量。可以为输入输出参数添加[类型提示](#类型提示)。

#### 参数

**参数传递**：不可变类型（数值、字符串、元组等）是值传递，**可变类型**（列表、字典、集合等）是引用传递；

**关键字参数**：使得调用函数时传入参数顺序可以与定义时不同，也可以为参数设置默认值（默认值在模块初始化时构造）；关键字参数必须置于所有位置参数之后。

```python
def func(arg1, age=50, name="miki")
```

**不定长参数**

`*args`用于传递任意数量的位置参数（列表或元组），`**kwargs`用于传递任意数量的关键字参数（字典）。

```python
def functionname(formal_args, *args，**kwargs)
```

> 在函数调用时，也可以使用`*args`将参数列表传递到函数中（需要与函数声明的参数个数或`*args`声明匹配），使用`**kwargs`将字典传入函数作为关键字参数（将值赋给与键名相同的参数，或与`**kwargs`声明匹配）。

#### 返回值

通过`return`语句可以设置一个或多个返回值，或不返回值。

> `return`不用于脚本退出，使用`sys.exit()`退出。

当返回多个值时，如果仅提供一个输出参数，则将返回值构造成元组。若提供多个参数存储返回值，则将从元组中一次读取元素给输出参数。

```python
a, b, c = function_name(...)  # 等效于 (a,b,c) = func_name(...)
d = function_name(...)  # d = (a,b,c)
```



#### 匿名函数 （Lambda表达式）

将函数作为**对象**保存和引用，也可以在使用函数的地方直接定义（而非使用`def`定义普通函数）。

```python
add = lambda x, y : x+y
f = lambda x: 1 if x > 0 else -1
the_sum = add(1,2)
```



## 变量

Python的主要内置类型有数值、序列（`string`、`list`和`tuple`）、映射、类、实例和异常，方法也可以看作特殊的对象类型。每个变量（`object`）具有一个标识、类型和值。

### 创建和删除变量

定义变量不需要声明类型，根据赋值的类型确定变量类型。

```python
a = b = c = 1
a, b, c = 1, 2, "john" 
```

变量可重复赋值，赋值前后类型不需要一致（由赋值类型决定）。

删除变量：

```python
del var_a, var_b 
del list[i], dict[name]
```

#### 变量作用域

在文件范围中定义的变量具有**文件作用域**，在文件其后的任意位置（包括调用的函数内部）都能访问。各文件中定义的变量属于不同的**命名空间**（模块名），互不影响。要引用其他模块中定义的全局变量，可使用`import`语句引入其他模块通过模块名访问或直接将该变量引入当前文件的命名空间。

```python
import module
print(module.global_var)
from module import global_var
```

> 在文件中定义的函数引用全局变量时，总是引用该文件作用域中的全局变量，而不会使用调用该函数的文件中的全局变量。

##### 可见性

当函数内部定义了与全局变量同名的局部变量，则全局变量将被隐藏（即使在定义局部变量之前也不能引用该同名全局变量）。使用`global`用于在函数内部对全部变量的声明和修改。

```python
a = 'initialized'
def func()
	global a
    a = 'modified'
```

获取作用域中的变量信息：

```python
dict_vars = locals()  # 获取当前的局部作用域中的变量
dict_vars = globals() # 获取全局作用域中的变量
```

> 包括变量、函数、模块等信息。

在流程控制语句块中定义的变量在离开语句块后仍有效。

#### 类型信息判断

`type()`返回变量的==类型信息可以和类型对象进行比较==。

```python
type_info = type(var_name)
tf = type(x) == int     # return True if x is int.
```

`get_type_hints()`可以查看模块、类、方法或函数的类型信息。

```python
from typing import get_type_hints
```

```python
Vector = list[float]  # type alias
from typing import NewType
UserId = NewType('UserId', int)  # New simple type
```

`isinstance`判断实例是否为某个类型（父类）的实例：

```python
isinstance(obj, Type)		
isinstance(obj, (Type1, Type2, ...))		#  任意一种类型
```

> `Type`是在程序中使用的变量类型，不是字符串，例如`pd.DataFrame`，`np.ndarray`。

` issubclass`判断一种类型是否为某类型的子类：

```python
isdubclass(subType, Type)
```



#### 类型提示

```python
age: int = 1   # 可不提供初始化值
```

容器类型需要使用专门定义的类型修饰变量。

```python
from typing import List, Set, Dict, Tuple, Optional, Callable
x: Set[int] = {6, 7}
x: Dict[str, float] = {'field': 2.0}
x: Tuple[int, str, float] = (3, "yes", 7.5)  # fixed size tuple
x: Tuple[int, ...] = (1, 2, 3)     # variable size tuple
x: Optional[str] = some_function() # values that could be None
x: Callable[[int, float], float] = f # function
```

类型别名：

```python
vector = List[float]
```

迭代器提示。

参数提示仅作为编写程序的辅助工具，在程序运行时并不会做相应的类型检查。

##### 联合类型

可以为参数指定类型，且可使用`typing.Union`指定多种类型；

```python
from typing import Union
def function_name(arg1:Union[str,int])
```

##### 常量类型

```python
from typing import Final    # [Python 3.8]
```



[Type hints cheat sheet (Python 3)](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html)。

### 类

类的定义：由成员，方法，数据属性组成。

```python
class ClassName:
	'''documentation'''   #类文档字符串
    static_var: Type = value
    var1: Type     # Annotate a member does not make it static.
    # 构造函数
    def __init__(self, value1, value2, ...) -> None:
        self.var1 = value1		# 成员变量
        self.var2 = value2
        ...
    # 析构函数
    def __del__(self):
    	release_unmanaged_resources
    # 成员方法
    def method_name(self, arg1, arg2, ...):
      self.xxx        # 调用类的成员变量或方法
    	# statements...
		@staticmethod      # 静态方法
    def method_name(...)
    	statements
    @property					 # read only property
    def prop(self):
    	return self.xxx
    @x.setter
    def x(self, value):
    	self.__x = value
    @x.deleter
    def x(self):
    	del self.__x
```

#### 静态变量

静态成员变量仅能通过类名或静态方法访问。实例可以定义与静态成员同名的成员变量（通过实例引用将隐藏静态变量）。子类可以定义与父类同名的静态变量（隐藏），通过子类名或实例将只能访问子类的静态变量。

==对实例成员的类型注释由于没有初始化，因此不会被视为静态变量==。

静态变量初始化：可在类外部对静态变量进行初始化，从而基于父类的静态变量对子类静态变量进行初始化。在子类内部使用父类静态变量初始化子类静态变量无效（得到`None`）。

#### 构造

浅拷贝：很多类型（例如`list`、`dict`）提供`copy()`方法，支持对象的浅拷贝。这意味着对象中的成员变量如果是引用类型，则两个对象共享该引用类型成员。

析构函数`__del__` ，`__del__`在对象销毁的时候被调用，当对象不再被使用时，`__del__`方法运行。Python使用了引用计数这一简单技术来跟踪和回收垃圾。

```python
obj = ClassName(args) # 创建对象
```

使用`.`运算符来访问对象的属性。

#### 类的成员

类可以看作由元数据、用户定义数据和方法组成的字典，使用`dir(x)`（`x.__dir__()`）返回类包含的属性和方法组成的字典。

类的方法与普通的函数只有一个特别的区别——它们必须有一个额外的第一个参数名称, 按照惯例它的名称是 `self`（`self`不是Python关键字，换成其他标识符仍然有效）。`self`代表的是类的实例，代表当前对象的地址，而`self.__class__`是类的类型信息。

Python成员不存在**访问控制**，仅通过标识符在语义上区分。

- `__<private_member>`：标记为私有属性/方法，其中形如`__INNER_MEMBER__`的成员为类的元数据。

- `_<protected_member>`：标记为保护属性/方法。
- `<public_member>`：标记为公开属性/方法。

##### 类的元数据

通过类或对象均可访问类的元数据（所有对象共享）。

`__class__`：类型元信息（即`type`类型对象，等效于`type(obj)`）；

`__name__`：类名（字符串，==不包含类所在的包名==）。

`__bases__` : 类的所有父类元信息构成的元素（包含了一个由所有父类组成的元组） 。

`__dict__`：所有成员变量名与对应的值组成的字典，等效于`vars(obj)`；

`__doc__`：类的文档字符串 。

`__module__`: 类定义所在的模块（类的全名是`__main__.className`，如果类位于一个导入模块`mymod`中，那么`className.__module__`等于 `mymod`） 

##### 实例的元数据

`obj.__str__(),`：类成员信息的字符串表示，自动转换为字符串类型时调用该方法；

`obj.__repr__()`：返回构造该对象的语句，`repr(obj)`会调用对象的该方法。调用`eval(repr(obj))`可重新构造该对象。

##### 描述符协议

在类里实现了` __get__()` 、`__set__()`、`__delete__()`其中至少一个方法。

```python
class Score:
  def __init__(self, default=0):
    self._score = default
  def __set__(self, instance, value): # 设置属性的值，进行验证
    if not isinstance(value, int):
      raise TypeError('Score must be integer')
    if not 0 <= value <= 100:
      raise ValueError('Valid value must be in [0, 100]')
    self._score = value
  def __get__(self, instance, owner): # 属性不存在、不合法等都可以抛出对应的异常
    return self._score
  def __delete__(self):  # 删除内容
    del self._score
class Student:
  def __init__(math_score):
    self.math = Score(math_score)
  pass
```



#### 数据类

`dataclass`（**Python 3.7+**）专门用于定义和存储数据的类型（更加适合[序列化](Python输入输出.md#对象序列化)），封装了数据的初始化方法和基本运算方法（避免频繁编写这些基础代码）。`dataclass`继承自`object`类型，因此，开发者仍可基于`dataclass`编写自定义方法[^dataclass]。

##### 定义数据

基于**类型提示**语法自动生成对应成员的定义，而无需重复在初始化方法中声明和初始化成员。

```python
from dataclasses import dataclass
@dataclass
class Persion:
    first_name: str
    last_name: str = "Wang"  # 支持默认值, 但必须在所有非默认值参数后
    age: int
    job: str
    full_name: str = field(init=False, repr=False) # *
```

> `dataclass`会自动生成`__init__()`、`__eq__()`（自动对比对象的所有成员）和`__expr__()`方法。由于`dataclass`的构造方法是自动生成的，也因此无法向父类传递初始化参数，所以通常自定义`dataclass`类型不再继承其他类。
>
> `*`：当该成员的初始化依赖于其他成员变量时，不在自动生成的构造函数中初始化，而是在`__post_init__`方法中定义初始化方法。这种方式避免使用属性，在每次调用时需要重复计算值。

##### 非可变对象

声明`@dataclass(frozen=True)`，对象初始化后无法被更改。

##### 数据转换

可轻易转换为元组或字典。

```python
from dataclass import astuple,asdict
person:Tuple = astuple(Person(...))
person:Dict = asdict(Persion(...))
```

##### 比较接口

`@dataclass(order=True)`将自动生成`__lt__`、`__le__`、`__gt__`、`__ge__`方法，从而支持排序比较。默认将对类中所有字段以此进行比较，在类定义中添加特殊的`sort_index`字段，该字段引用其他成员变量的值以定义[排序参考值](Python数据类型md#列表排序)。

```python
@dataclass(order=True)
class Persion:
    # ...
    sort_index: int = field(init=False, repr=False)
    def __post_init__(self):
        self.sort_index = self.age
```



#### 继承语法

```python
class ClassName (ParentClass1[, ParentClass2, ...]):  # 支持多重继承
    'Optional class documentation string'
    def __init__(self, args):
      super().__init__(...)  # <=> super(ClassName, self).__init__(args)
      # ...
```

在继承中基类的构造（`__init__()`方法）不会被自动调用，它需要在其派生类的构造中手动。需要通过基类名调用`__init_()`并且传递`self`变量。



##### 方法重写（override）

当方法被重写后，==通过对象调用方法时将调用子类的方法==（即使是在父类中）。要通过`super()`以显式调用父类方法。

```python
super().method(args)  # <=> super(ClassName, self).method(args)
```

同名函数或成员调用：使用`super().member()`调用[MRO搜索顺序](#MRO（Method-Resolution-Order）)上第一个存在该成员的类。如果要显式调用某一父类的方法，则使用`super(SuperClass, self).method(args)`。

> 在基于`pdb`的调试环境下，如果在调试窗口调用父类方法，使用`super().method(args)`会报错（程序中正常执行）。

##### 运算符重载

##### MRO（Method Resolution Order）

父类的初始化顺序：根据继承关系“深度优先—从左至右”搜索父类，确定初始化顺序（调用`classname.__mro__`查看类的搜索顺序）。

- 深度优先，按继承关系依次调用父类构造函数；
- 从左至右，当深度搜索到达顶端后，==如果顶层父类包含`super().__init__()`调用==，则将调用第二条继承关系上的类型，并执行深度优先初始化；
- 如果两条继承路线存在公共父类，则在第一条继承路线搜索到公共父类前会跳转到第二条继承路线，由第二条继承路线搜索到公共父类；多条继承路线具有公共父类的情况同理。
- 如果搜索过程中，某父类不存在`super().__init__()`，则从该父类的代码开始，并按调用栈反向执行初始化。==在构造过程中已经被前面构造函数初始化过的属性会被后调用的构造方法再次初始化==。

> 如果在MRO搜索顺序中某个直接或间接父类未调用`super().__init__()`，则==MRO初始化过程中断==（该类及MRO搜索顺序的后续类的构造方法不会被调用）。

参数传递：

- ==初始化过程中的参数传递与初始化顺序一致，而非子类分别传递给其各个直接父类==。因此在声明父类时，一定要将能通过`super().__init__()`传递参数的父类声明在前，否则其后的父类无法获取参数。

- 传递给父类参数通常使用`*args`和`**kwargs`代替（除非子类需要对相关参数进行处理）。
  ```python
  def __init__(self, a,b,*args,c=1,b='hello',**kwargs)
  ```

  位置参数的顺序保持子类参数在前，父类参数在后（从而可以使用`*args`来统一接收父类参数）。

[Python Multiple Inheritance - JournalDev](https://www.journaldev.com/14623/python-multiple-inheritance)

##### mixin

抽取单一功能，提供给多个类继承，可视为带实现的接口；mixin模式适用于多个类各自有继承主线，但又共享部分功能。这部分功能即可通过mixin类型实现。

`mixin`类型*不定义新的成员变量*，仅定义关于目标混入对象的计算方法（可使用期望继承的类所包含的成员）。

https://stackoverflow.com/q/533631/6571140。

### 迭代器

迭代器`Iterator`提供`__next__()`方法以遍历所有元素；使用`yield`关键字的方法也可以实现简单迭代器。

```python
def g(n: int) -> Iterator[int]:
    i = 0
    while i < n:
        yield i
        i += 1
```

可迭代对象`Iterable`提供两个方法:`__iter__()`和`next()`。通过`iter(Iterable)`可获取访问可迭代对象的迭代器。[`for`语句](#循环)实际需要传递迭代器对象，通过语法糖简化了对可迭代对象的访问语法。

[python - What exactly are iterator, iterable, and iteration? - Stack Overflow](https://stackoverflow.com/questions/9884132/what-exactly-are-iterator-iterable-and-iteration)

### 内存占用

```python
import sys
s = sys.getsizeof(var)
```

所有内置类型均以对象封装，因此返回的为对象占用的内存（仅计算对象本身占用的内存，而不包括对象引用的内存）。第三方类型类型返回结果不一定准确。

## 程序结构

### 程序入口

模块中非类、函数定义的代码部分将在引用时被执行。

通过以下方式为程序设置一个入口，从而屏蔽引用模块中的非定义代码：

```python
if __name__ == '__main__':
	main_procedure
else:
	module_initialization
```

> 注意：上述代码并非常规的主函数（`main(args`))，只是一个普通的条件语句。可以自定义一个[常规的主函数](https://codingpy.com/article/guido-shows-how-to-write-main-function/)在上述结构中进行调用。

```python
def main(args=None):
    if args is None:
        args = sys.argv
    # main code

 	return value   
    
if __name__ == '__main__':
    sys.exit(main(sys.argv[1:]))
```



#### 命令行参数处理

`sys.argv[0]`表示程序名，其他元素为传入参数。

```python
for argi in sys.argv:
    print(argi)
```

##### getopt

`getopt`采用Linux Shell的参数声明规则设置参数。读取参数的方法：

> 与UNIX类系统不同，非选项参数后的所有参数都不会被视为选项解析。

```python
from getopt import getopt, GetoptError
options, args = getopt(cmd_args, short_opts, long_opts=[])
```

`short_opts`：代表短选项（命令行以`-`开头）的字母列表，如果一个选项还对应一个值，那么字母后添加`:`；

`long_opts`：代表长选项（命令行以`--`开头）的字符串列表，长选项如果需要一个值，则参数需要附加`=`。

> 命令行中，长选项的值可作为一个当都参数，或使用`=`附加在选项名后。

返回值：`options`为选项列表，包括选项名（包括前缀）和对应的值（没有值则为空字符串）；`args`为非选项参数。

典型用法：

```python
def usage():
    # print usage of the program, including options.

def main(sys_args):
    try: 
        optlist, args = getopt(sys_args, short_opts, long_opts=[])
    except GetoptError as err:
        print(err) # will print something like "option -a not recognized"
        usage()
        return(2)
    for option, value in optlist:
        if option == '--test' or option == '-t':
	        # do something
    	...        
```

##### [ArgumentParser](https://docs.python.org/3.7/library/argparse.html#module-argparse)

> `ArgumentParser`替代了`OptionParser`（从Python 3.2）。`ArgumentParser`对参数控制更加严格，如果出现未配置的参数将产生异常。==参数名如果没有前缀，则代表位置参数==；而`OptionParser`将位置参数存储到一个单独的返回参数中。

创建参数解析对象实例：

```python
from argparse import ArgumentParser
parser = ArgumentParser(
  prog=None,        # program name (default: sys.argv[0])
  description=None, # information before argument help
  usage=None,       # usage syntax (default: auto generate)
  epilog=None,      # information after argument help
  parent=[parent_parsers]  # 继承父解析器实例的解析方法
)
```

>  `parent`：指定继承的参数解析实例。

添加参数解析规则：

```python
parser.add_argument(
   names_or_flags,    # 可变参数列表：'--test', '-t', 'argname'
   required=False,
   dest='optname',    # attribute name in return options
   action='store',  
   nargs=1,       # number of option arguments
   const=0,       # 
   default=None,  # default value if not specified from command line
   type=str,      # type of option arguments
   help=None,     # option's usage information
   metavar='NAME' # 帮助信息中作为选项值的标识（默认为长选项名大写）
)
```

参数解析规则说明：

- `names_or_flags`：如果名称不含`-`或`--`，则表明该参数为位置参数（必须在运行时指定，且不需要添加`required`参数）。

- `dest`：解析参数列表后存储参数值的变量名（参看`parse_args()`方法）；如果没有指定字段，则会根据选项名生成合法的字段名；

- `nargs=N`：选项所需要的参数个数，消耗*N*个参数构成列表。当需要获取位置参数而非选项时，`nargs='?'`消耗一个位置参数，`nargs='*'`消耗所有位置参数构成列表，此时`names_or_flags`作为存储位置参数的变量名。

- `type`：值的类型，包括：`string`（默认）、`int`、`float`；整数值可以和短选项名组成一个参数，例如`-n42`等价于`-n 42`；`bool`类型参数使用`store_true, store_false`；

- `action`：检测到选项后的处理方式

  - `store`：（默认处理方式）储存值，值的类型通过`type`指定；
  - `store_true|store_false`：储存`bool`值`True`/`False`，对应的选项不需要另外参数设置值；
  - `store_const`：`'store_const'`与`const`关键字参数结合使用，用于为选项保存一个常量。
  - `append`：指定该类型的选项存储类型默认为数组，==可重复声明该选项==，以将每次声明的选项值追加到数组中。
  - `'append_const'`：与`const`关键字参数结合使用，用于没有值的参数。

- `default`：选项的默认值（选项未出现在命令行的情况下设置默认值）。

  > 选项如果不是`bool`类型，在命令行使用选项时必须提供值；

- `help`：选项的帮助信息。当解析参数时，遇到`-h`或`--help`（程序退出）或调用`parser.print_help()`时会自动输出所有帮助信息。

  > ` %default`可用于在帮助信息中表示输出变量的默认值。`ArgumentParser`自带`-h,--help`选项，命令行提供该选项时将输出帮助信息。

参数解析器默认会添加帮助选项（`-h,--help`），如果要禁用可在构造解析对象时设置`add_help=False`。

转换命令行参数（默认为`sys.argv[1:]`，==注意传入给`args`不要展开==）：

```python
options = parser.parse_args(args=None, namespace=None)
```

未在命令行提供的选项也会出现在`options`中，其值为默认值。返回值为[`NameSpace`类型](Python数据类型.md#NameSpace)。

自动命令补全：[argcomplete - Bash tab completion for argparse — argcomplete documentation (kislyuk.github.io)](https://kislyuk.github.io/argcomplete/)。

##### click

如果命令行嵌套子命令，可使用[click](Python编程应用.md#click)进行命令转发。

#### 程序运行信息

```python
from inspect import currentframe, getframeinfo
frameinfo = getframeinfo(currentframe())
frameinfo.filename  # 当前运行代码所在文件
frameinfo.lineno    # 当前运行代码所在行号
frameinfo.function  # 当前运行代码所在函数
```

### 模块(Module)

模块是一个文件，其中包含类，函数等的定义。==模块相当于一个命名空间，其中的定义与其他模块隔离==。

##### 导入模块

```python
import mod                    # modulename <- sys.modules['mod']   
import mod as alias           # aliasname <- sys.modules['mod']   
from mod import name          # objname <- sys.modules['mod'].name 
from module import (name1,name2,...,namen,)
from module import *          # 导入所有内容
from mod import name as alias # aliasname <- sys.modules['mod'].name
import importlib
path = importlib.import_module("os.path")  # => import os.path as path
file, pathname, desc = importlib.find_module('os')  # 查找模块
```

> *不能直接导入模块中的内容并设置别名*。

从模块所在目录导入其他模块：

```python
from . import module_name
```

> 相对路径是[根据导入声明的包名](https://stackoverflow.com/a/14132912/6571140)`package.subpackage.module`确定的，因此不可使用相对导入声明路径范围外的模块。由于`__main__`模块不包含任何路径信息，因此无法使用相对导入。

[5. The import system — Python 3.9.5 documentation](https://docs.python.org/3/reference/import.html)

##### 引用模块

当导入整个模块时，使用模块名称访问其中的内容（类、函数等）；也可以直接导入模块中的特定内容。

```python
{module|alias}.name     # use module contents via module name/alias
name       # use name/alias to refer an imported class,funcion,...
alias
```

##### 重载模块

检查模块是否导入：`sys.modules`记录了运行环境已[导入的模块](https://stackoverflow.com/questions/30483246/how-to-check-if-a-python-module-has-been-imported)，从而防止模块被重复导入。

```python
tf = 'modulename' in sys.modules  # dict
tf = 'importedname' in dir()      # 当前可见名称 
tf = 'importedname' in globals()  # 全局变量
```

重新加载修改过的包到当前正在运行的程序：

```shell
import importlib
importlib.reload(module)  # <class module> not str
```

##### 查看模块的文档

```python
help(len)  # doc for built-in functions
import pandas as pd
help(pd)	# module doc
help(pd.DataFrame) # class doc
help(pd.DataFrame.to_csv)# function doc
```

##### 相互引用

*注意引用对象顺序，避免先引用未初始化的对象。*

```python
# in module A
from module B import b
def a()
# in module B
from module A import a  
```

#### 包(Package)

Package是特殊的模块（包含`__path__`属性的模块），包含`subpackage`和`module`。

常规`package`通常是一个文件夹中的所有模块以及子文件夹（`sub-package`）组成。`__init__.py`文件用于**标识目录是一个package**。当包被导入时，会自动执行`__init__.py`文件

```shell
my_package/
├── __init__.py
├── subpackage1/
│   ├── __init__.py
│   ├── module_x.py
│   └── module_y.py
├── subpackage2/
│   ├── __init__.py
│   └── module_z.py
└── module_a.py
```

##### Namespace packages

>  *A [namespace package](https://docs.python.org/3/reference/import.html#namespace-packages) is a composite of various portions, where each portion contributes a subpackage to the parent package. Portions may reside in different locations on the file system.*

##### 导入包中的模块

导入包名（相对搜索路径的目录路径）后，通过包名访问其中的内容；或将包名作为前缀导入其中的模块。

```python
import pkg
import pkg.mod                # packagename <- sys.modules['pkg']
import pkg.mod as alias       # aliasname <- sys.modules['pkg.mod']  
from pkg import mod           # modulename <- sys.modules['pkg.mod']  
from pkg import mod as alias  # aliasname <- sys.modules['pkg.mod'] 
```

导入包名的时候会执行包目录下的`__init__.py`加载包中的模块。没有在`__init__.py`中指定加载的模块需要手动加载（`import pkg.mod`）。

##### 查找路径

`sys.path` 是一个保存了查找包的路径的列表。Python解释器会自动将==Python程序所在目录==加入路径，从而方便导入同一目录下的其他文件。

> *==不要在包内部编写测试脚本==，运行该测试脚本时会将该脚本所在路径加入查找路径，从而破坏其所在包的结构*。
>
> 程序的工作目录（`path.abspath(os.curdir)`）与程序所在目录可能不同，且不会加入搜索路径。
>
> **语法检查的查找路径**：在进行语法检查时（例如在VS Code中使用`pylint`检查器），检查器的会将当前工作目录加入搜索路径，因此跟程序运行时的情形不同。为了保证程序运行时的路径也在检查器中，可在项目配置文件（`settings.json`）中配置
>
> ```json
> "python.analysis.extraPaths": ["E:/Workspace/python"]
> ```

可以添加自定义搜索路径以导入第三方包：

```python
sys.path.insert(0, '/lib/path')  # 添加至头部
sys.path.append('/lib/path')  # 添加至尾部
```

查询包是否在本地可加载：

```python
import importlib
spec = importlib.util.find_spec("dask.dataframe")  # None if not find
```

##### 相对导入

```python
from . import module   # 导入同一包内同一层级的子包（或模块）
from .. import module  # 导入同一包内上一层级的子包（或模块）
from .package import module
from ..package import module
```

> 不要将子包所在路径加入搜索路径，否则不能识别完整包结构，导致相对导入失败。

[Python Modules and Packages – An Introduction](https://realpython.com/python-modules-packages/)。





## 标准库和应用

### 日期和时间

[pytz模块](http://www.twinsun.com/tz/tz-link.htm)

[dateutil模块](http://labix.org/python-dateutil)

### CGI

### 密码学

##### 消息摘要

Python内置`hashlib`（消息摘要算法），包括 FIPS 的 SHA1, SHA224, SHA256, SHA384, and SHA512 (定义于 FIPS 180-2) 算法，以及 RSA 的 MD5 算法( 定义于 Internet [**RFC 1321**](https://tools.ietf.org/html/rfc1321.html))。消息摘要算法的输入为字节序列

> 在 [`zlib`](https://docs.python.org/zh-cn/3/library/zlib.html#module-zlib) 模块中包括adler32 或 crc32 哈希函数。

```python
import hashlib
m = hashlib.sha256(b'init_data')
# m.update(a); m.update(b) => m.update(a+b)
m.update(b"Nobody inspects")          # 向该对象追加消息(字节序列)
m.update(b" the spammish repetition")
m.digest()   # 或hexdigest()
```

> 为了防止黑客通过彩虹表根据哈希值反推原始口令，在计算哈希的时候，不能仅针对原始输入计算，需要增加一个salt来使得相同的输入也能得到不同的哈希，这样，大大增加了黑客破解的难度。

##### 加密算法

`cryptography`包含加密、消息摘要以及密钥生成等方法。

> *The low-level cryptographic primitives are often dangerous and can be used incorrectly. They require making decisions and having an in-depth knowledge of the cryptographic concepts at work.*

`PyCryptodome `（代替[`pycrypto`](https://www.pycrypto.org/)）：Python Cryptography Toolkit提供加密算法（AES, DES, RSA, ElGamal, ...），同时也提供安全哈希函数（SHA256，MD5等）。

```python
from Crypto.Hash import SHA256
hash = SHA256.new()    # update, digest, hexdigest
from Crypto.Cipher import AES
obj = AES.new('This is a key123', AES.MODE_CBC, 'This is an IV456')
message = "The answer is no"
ciphertext = obj.encrypt(message)
obj2 = AES.new('This is a key123', AES.MODE_CBC, 'This is an IV456')
obj2.decrypt(ciphertext)
from Crypto.PublicKey import RSA
from Crypto import Random
random_generator = Random.new().read
key = RSA.generate(1024, random_generator)
signature = key.sign(hash, '')
public_key=key.publickey()
public_key.verify(hash_ver,signature)
```

[cryptography vs PyCrypto | LibHunt](https://python.libhunt.com/compare-cryptography-vs-pycrypto)

##### HMAC

[hmac — Keyed-Hashing for Message Authentication — Python 3.10.7 documentation](https://docs.python.org/3/library/hmac.html#module-hmac)

[RFC 2104 - HMAC: Keyed-Hashing for Message Authentication (ietf.org)](https://datatracker.ietf.org/doc/html/rfc2104.html)：通信双方使用共享密钥对消息的摘要（如MD5、SHA-1等）进行完整性进行校验。

```python
import hmac
h = hmac.new(key, message, digestmod='MD5')
```



## 常见问题

1. ModuleNotFoundError: No module named 'win32api'

   安装`pypiwin32`包。

## 参考文献

1. [Basic of Python Programming](https://nbviewer.org/github/Nyandwi/machine_learning_complete/blob/main/0_python_for_ml/intro_to_python.ipynb)
1. [Awesome Python | LibHunt](https://python.libhunt.com/)
