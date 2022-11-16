# Python高级编程

Python语言的高级特性。

## 协程和任务

> *`asyncio` is used as a foundation for multiple Python asynchronous frameworks that provide high-performance network and web-servers, database connection libraries, distributed task queues, etc.*

asyncio适用于IO密集型任务，提供**high-level** APIs：

- [run Python coroutines](https://docs.python.org/3/library/asyncio-task.html#coroutine) concurrently and have full control over their execution;
- perform [network IO and IPC](https://docs.python.org/3/library/asyncio-stream.html#asyncio-streams);
- control [subprocesses](https://docs.python.org/3/library/asyncio-subprocess.html#asyncio-subprocess);
- distribute tasks via [queues](https://docs.python.org/3/library/asyncio-queue.html#asyncio-queues);
- [synchronize](https://docs.python.org/3/library/asyncio-sync.html#asyncio-sync) concurrent code;

以及底层API用于开发库和框架。

### 异步方法

基于`async/await`语法定义异步函数：异步方法中可嵌套调用异步方法，也可仅是普通耗时IO操作。

```python
import asyncio
async say_after(length, msg):
   await asyncio.sleep(length)
   print(msg*length)
async def main():
   print('start')
   await asyncio.sleep(1)
   task1 = asyncio.create_task(say_after(1, 'hello'))
   task2 = asyncio.create_task(say_after(2, 'hello'))
   await task1
   await task2
   print('end')
```

#### 运行异步方法

##### 入口函数

执行异步方法，管理异步事件循环和线程池。

```python
asyncio.run(main())
```

##### 嵌套调用异步方法

直接访问异步方法返回的是`coroutines`对象，需要通过`await`关键字来启用协程。

```python
await async_method()
await function_that_returns_a_future_object()  # await Future object
```

***awaitable*** objects: **coroutines**, **Tasks**, and **Futures**.

##### 并发调用异步方法

使用`asyncio.create_task`创建并调度多个协程并返回`Task`对象，可在后续等待一批协程执行完成。



https://docs.python.org/3/library/asyncio-task.html

https://docs.python.org/3/library/asyncio.html



## 反射

### 获取动态属性

```python
value = getattr(obj, name) # exception if not exist
tf = hasattr(obj, name)
```

### 动态增删属性

```python
setattr(obj, name, value)  # obj.attr = value
delattr(obj, name)         # del obj.attr
```

> `object.__set_attr__()`：子类内部可覆盖该方法，以改变设置属性时的行为。只有在没有找到属性的情况下，才调用`__getattr__`，已有的属性，比如`name`，不会在`__getattr__`中查找。
>
> 注意：如果存在成员函数名`name`，则不能设置同名属性，否则产生（`AttributeError`）。

[定制类 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/1016959663602400/1017590712115904)

## 装饰器

装饰器（*Decorator*）可以基于函数的闭包构建，为已有函数`some_func`增加前置/后置处理。

```python
def decorator(some_func):
   def wrapper(*args,**kwargs):  # warpper for some_func
      print "before some_func"
      ret = some_func(*args,**kwargs) 
      return ret + 1
   return wrapper
# 等效于回调函数
def decorator(some_func, *args, **kwargs):
   print "before some_func"
   ret = some_func(*args,**kwargs) 
   return ret + 1
```

使用装饰器：

```python
@decorator
def some_func(*args,**kwargs):
  pass
# equivalent form
some_func = decorator(som_func)(*args,**kwargs)
```

向装饰器传递参数需要再定义一层装饰器：

```python
def decorator_with_args(deco_args):
   def decorator(some_func):
      def wrapper(*args,**kwargs): 
         # use deco_args in the the wrapper function.
         return some_func(*args,**kwargs) 
      return wrapper
   return decorator
# define function with decorator
@decorator_with_args(deco_args)
def some_func(*args, **kwargs)
```

##### 基于类的装饰器

装饰器实际是`callable`对象。除函数外，实现` __call__ `方法的类也是`callable`对象（等效于将基于函数定义的内容放到了`__call__`方法中）。带参数的基于类的装饰器定义（在构造函数中定义传递给装饰器的参数，装饰目标传递给`__call__`方法）：

```python
class logger(object):
  def __init__(self, level='INFO'):
    self.level = level
  def __call__(self, func): # 实现callable接口
    def wrapper(*args, **kwargs):
      print("[{level}]: the function {func}() is running..."\
            .format(level=self.level, func=func.__name__))
      func(*args, **kwargs)
    return wrapper # 返回包装后的函数对象

@logger(level='WARN')
def somefunc(*args, **kwargs): # => logger(level='WARN')(somefunc,...)
  pass
```

如果装饰器不带参数（类比于基于函数的装饰器，比带参数的少了一层函数定义；装饰目标传递给装饰器的构造方法）：

```python
class logger(object):
   def __init__(self, func):
      self.func = func
      def __call__(self, *args, **kwargs): # 实现callable接口
         print("[INFO]: the function {func}() is running..."\
               .format(func=self.func.__name__))
         return self.func(*args, **kwargs)
@logger
def somefunc(*args,**kwargs): # => logger(somefunc)(...)
  pass
```



##### 装饰类的装饰器

装饰器的参数由函数名变为类。

```python
class singleton:    # <== 基于类的装饰器，用于装饰Singleton Class
   _instances = {}  # <== 记录所有Singleton实例
   def __init__(self, cls):
      self._cls = cls
   def __call__(self, *args,**kwargs):
      if self._cls.__name__ not in singleton._instances:
         singleton._instances[self._cls.__name__] = cls(*args, **kwargs)
      return singleton._instances[self._cls.__name__]
@singleton
class User:
   def __init__(self, name):
      self.name = name
user = User('gary')   # => singleton(User)('gary')
```



## 跨语言编程

### Matlab

#### Python调用Matlab

```shell
cd "matlabroot\extern\engines\python"
python setup.py install   # conda activate
```

> [查看支持的Python版本](https://www.mathworks.com/content/dam/mathworks/mathworks-dot-com/support/sysreq/files/python-support.pdf)。

```python
import matlab.engine
eng = matlab.engine.start_matlab()
eng.addpath(r'path\to\work\dir')
```

##### 在Python程序中创建Matlab矩阵类型

```python
matlab.type(initializer=None,size=None,is_complex=False)
```

`type`包括：`double`、`single`、`intXX`、`uintXX`、`logical`。

> `matlab.int64`在Matlab中转换为`int32`；`matlab.uint64`在Matlab中转换为`uint32`；`logical`类型不能用于构造复数（无`is_complex`参数）。

`initializer`为矩阵初始值，即Python序列类型（`list`和`tuple`）；`size`为矩阵维度，需要与初始化值的维度匹配。未指定`size`时，矩阵维度默认为$1\times N$；如果初始化数据中元素为序列类型，矩阵维度为$m\times N$。未指定初始化值时，元素值设为默认值（`0`）。

`matlab.type.size`：获取矩阵的维度；
`matlab.type.reshape(size)`：修改矩阵维度；
`pyarray[i]`：索引矩阵的行。Matlab矩阵默认为二维矩阵。
`pyarray[i][j]`:索引矩阵元素。Matlab中索引起始为1，在Python代码中索引起始为0；
`pyarray[i][j:k]`：切片；

> 如果使用Matlab矩阵自身元素更新自身可能得到造成非预期结果。
>
> ```python
> mlarray[0] = mlarray[0][::-1]
> ```

将数据从Python程序传递至Matlab程序：

| Python Input Argument Type                                   | Resulting MATLAB Data Type         |
| :----------------------------------------------------------- | :--------------------------------- |
| `matlab.type` ([数值矩阵](#在Python程序中创建Matlab矩阵类型)) | 数值矩阵                           |
| `float`                                                      | `double`                           |
| `complex`                                                    | Complex `double`                   |
| `int`                                                        | `int32`(Windows)`int64`(Linux/Mac) |
| `float('nan'\|'inf')`                                        |                                    |
| `bool`                                                       | `logical`                          |
| `str`                                                        | `char`                             |
| `bytearray`/`bytes`                                          | `uint8` array                      |
| `dict`                                                       | 结构体（key必须为字符串）          |
| `list`/`set`/`tuple`/`range`                                 | Cell array                         |
| `memoryview`/`None`                                          | Not supported                      |

从Matlab程序返回至Python的数据：

| MATLAB Output Argument Type             | Resulting Python Data Type |
| :-------------------------------------- | :------------------------- |
| 数值矩阵                                | `matlab` 数值矩阵          |
| `double` `single`                       | `float`                    |
| 复数(any numeric type)                  | `complex`                  |
| `int8` `uint8` `int16` `uint16` `int32` | `int`                      |
| `uint32` `int64` `uint64`               | `int` `long`               |
| `NaN`                                   | `inf`                      |
| `logical`                               | `bool`                     |
| `char` array (1-by-`N`, `N`-by-1)       | `str`                      |
| 结构体                                  | `dict`                     |
| 1D cell array                           | `list`                     |

### 在Matlab中调用Python

在Matlab会话中启用Python环境：

```matlab
pe = pyenv('Version','C:/tools/miniconda3/envs/data/python.exe')
pe = pyenv   # read curent python environment
```

> 在整个会话中只能导入一次Python环境，可根据`pe.Status`来判断是否已经导入。

调用python的包无需使用`import`语句

```matlab
syspath = py.sys.path
syspath.append(pwd)
import py.textwrap.wrap
S2 = wrap('another string');
mm = py.importlib.import_module('mymod');
% Use mm as an alias to access functionality in mymod
py.help('object')
```

### Java

[Welcome to Py4J — Py4J](https://www.py4j.org/)



## Python项目

##### Basic Layout

```shell
projectname/
├── .gitignore
├── projectname.py
├── LICENSE           # if you’re distributing code.
├── README.md
├── requirements.txt  # Python dependencies and versions
├── setup.py          # installation
└── tests.py
```

##### modulized layout

```shell
helloworld/
├── bin/              # 可编写脚本调用模块的主程序
├── docs/
│   ├── hello.md
│   └── world.md
├── helloworld/       # package contains sub-modules
│   ├── __init__.py   # indicator of package
│   ├── helloworld.py # 可包括多层内部package进行功能划分
│   └── helpers.py
├── tests/            # 对应功能模块结构编写测试
│   ├── helloworld_tests.py
│   └── helpers_tests.py
└── ...   # other files in basic layout
```

### 测试

#### 性能测试

代码片段的运行时间统计。

```python
from timeit import timeit
print(timeit('[n**2 for n in range(10) if n%2==0]', number=10000))
```

#### 单元测试

> *A unit is a specific piece of code to be tested, such as a function or a class. Unit tests are then other pieces of code that specifically exercise the code unit with a full range of different inputs, including boundary and edge cases.*

`unittest`用于单元测试。`setUp`（由`__init__()`调用）和`tearDown`用于为每个测试用例初始化和回收资源，与`test_case()`组成一个测试配置。`unittest.main()`为测试类中的每个测试用例构造一个测试类实例，并调用该测试用例方法（因此在一个测试类中`setUp`等方法调用次数与测试样例数一致）。

```python
import unittest
class MyTestCases(unittest.TestCase):
  def setUp(self):
    pass
  def tearDown(self):
    pass
  def test_case(self): # 测试项
    self.assertTrue(True, msg='Test Pass.')  # 使用assert语句判断并记录测试结果
if __name__ == '__main__':
  unittest.main()   # 使用Python运行该脚本自动执行所有测试
```

执行测试。测试项的执行顺序依据测试方法名称排序。

```shell
python test/components/test_operators.py
```

```shell
python -m unittest test.components.test_operators 
python -m unittest test/components/test_operators.py
# 需要相应目录下有__init__.py，否则无法将对应文件夹视为模块
```

##### 构造测试组

```python
if __name__ == '__main__':
  suite = unittest.TestSuite()
  suite.addTests([
    TestCases('test_scalar_operator'),  # 参数为测试类中的测试方法名
  ])
  runner = unittest.TextTestRunner()
  runner.run(suite)
```

`TextTestRunner`以文本形式显示测试结果，包括：

- 测试名称；
- 测试过程中的错误信息；
- 测试结果汇总。



### 编译打包

```shell
python -m compileall pysrc.py # compile to .pyc file
python -m zipfile \
       -c demo.zip demo/*   # 将源文件打包为zip => 执行：python demo.zip
python3 -m zipapp demo -m "main:main"
```



### 分发和安装

```python
from setuptools import setup, find_packages
setup(
   name="mytest",
   version="1.0",
   author="wangbm",
   author_email="wongbingming@163.com",
   description="Learn to Pack Python Module",
   url="http://iswbm.com/", 
   packages=find_packages(),
   python_requires='>=2.7, <=3',
	install_requires=['docutils>=0.3'],  # 自动下载安装依赖包
   setup_requires=['pbr'],              # 当前环境必须存在依赖包
   entry_points={
        'console_scripts': [
            'foo = foo.main:main'  # 生成/usr/bin/foo -> foo.main模块中的main函数
        ]
    },
   scripts=['bin/foo.sh', 'bar.py'] # 将脚本复制到系统路径中
)
```

[花了两天，终于把 Python 的 setup.py 给整明白了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/276461821)

[setuptools Quickstart - setuptools 60.5.2 documentation (pypa.io)](https://setuptools.pypa.io/en/latest/userguide/quickstart.html)
