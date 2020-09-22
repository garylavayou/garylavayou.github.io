# 数据类型

`string`, `tuple`，`bytes`和 `number `是**不可更改的对象**，而 `list`,`dict`等则是**可以修改的对象**。

## 数值（Numbers）

- 整数（`int`）：Python3不区分32bit/64bit整数，可以表现任意位数的整数。

  - 二进制前缀：`0b`；

  - 八进制前缀：`0o`；

  - 十六进制前缀：`0x`；

- 浮点型（`float`）：有小数点或科学记数法表示

- 逻辑（`bool`）：值为`True`或`False`。

- 复数（`complex`）：实部和虚部都是浮点型。

  ```python
  a + bj
  complex(a,b)
  ```

- 非数值

  `x=float('nan')`

## 时间

### 时间类型

#### 时间戳

Python使用浮点数表示时间戳（单位：秒，浮点数精度$10^{-7}$）。

```python
import time
ts = time.time()  # return timestamp (seconds)
```

#### struct_time

`localtime()`获取本地时间，返回时间元组`struct_time`；`mktime`做逆向变换，
`tup_time`为表示时间的九元组，或`time_struct`。

```python
st_time = localtime(ts) # to struct_time 
ts = mktime(tup_time)   # to timestamp
```

### 格式化时间

`strftime`用于自定义时间格式，格式声明[参考](../Linux and UNIX/Linux配置和管理#日期时间)。
`strptime`将格式化的时间字符串按格式转换为时间元组。

```python
str_time = asctime(st_time) # 格式化输出时间。
str_time = strftime(format, st_time)
st_time = strptime(str_time, format)
```

字符串表示的时间与时间戳相互转换：

```python
from time import mktime,strptime  
ts = mktime(strptime('2019090109', '%Y%m%d%H'))
ts_micro = int(ts*1000)   # 毫秒表示的时间戳
from time import strftime, localtime
str_time = strftime('%Y%m%d%H', localtime(ts))
```

### 时间对象

时间对象封装了时间戳/时间字段。[Python](https://docs.python.org/3/library/datetime.html)在`datetime`模块中提供多个类处理时间。

```python
import datetime as dt
t = dt.time(hour, minute, second, microsecond)  # time object
d = dt.date(year, month, day)  # date object
day = dt.date.today() # fromtimestamp, fromisoformat
tt = dt.datetime(year, month, day, hour, ...) # datetime object  
now = datetime.datetime.now()  # today, fromtimestamp, fromisoformat, strptime
day = day + dt.timedelta(days=1,hours=0,...)
t = dt.datetime.strptime('%Y%m%d')
dt.timezone
```

Pandast的`Timestamp`类为`datetime.datetime`的子类，提供了时间转换和提取时间字段等方法。

```python
import pandas as pd
dt = pd.to_datetime(secs, unit='s')   # unit = 's'|'ms'|'ns'
dt.tz_localize(UTC).tz_convert('Asia/Shanghai')  # timezone
secs = dt.timestamp()
```

### 时间段

表示由开始和结束时间确定的时间区间。

`Pandas.Period`可对时间跨度进行单位转换。

> `pandas.Interval`表示浮点数区间。




## 容器

容器包括序列类型（列表、元组、字符串）、映射类型（字典）、集合类型等。

##### 统计函数

```python
x = max(a)			# built-in max/min in Python, min
l = len(a)			# length of container
```

### 序列类型

#### 字符串（String）

```python
s = 'ilovepython'
```

Python可以使用引号(`'`)、双引号(`"`)、三引号(`'''`或 `"""`)来表示字符串，引号的开始与结束必须的相同类型的。其中三引号可以由多行组成，可以直接包含换行、制表等特殊字符（而无需使用转义字符）；常用于[文档字符串](#注释和文档)，在文件的特定地点，被当做注释。

##### 转义字符

| 转义序列 | 实际字符                                 |
| -------- | ---------------------------------------- |
| `\\`     | \                                        |
| `\"`     | "                                        |
| `\' `    | '                                        |
| `\000`   | `NULL`（等价于`\x00`，类似地`\x01`,...） |
| `\n`     | 换行                                     |
| `\t`     | 横向制表符                               |

Unicode 字符串：`u'Hello World !'`（在Python3中，所有的字符串都是Unicode字符串）。

原始字符串，不解释转义字符：`r'Hello\nWorld!'`（或`R`前缀）。

变量替换：`s = f'{arg1}, {arg2}'`

可以采用[对象运算符](#对象运算符)对字符串进行拼接、重复和截取等操作。

##### 查找统计

`idx=str.find(sub_str, beg=0, end=len(str))`：`rfind`、`index`、`rindex`，以及[序列类型公共方法](#序列类型公共方法)（`index`）

`tf=str.startWith(suffix, beg=0,end=len(str))`：`endWith`

`n=str.count(sub_str, beg=0,end=len(str))`

`is*()`方法：

##### 变换

`new_str = str.replace(old, new[, max])`

`new_str=center(width, fillchar)`：类似地`ljust`、`rjust`、`zfill(width)`。

`new_str=str.expandtabs(tabsize=8)`：将tab符号替换为空格。

`new_str=strip(chars='')`：移除字符串两端`chars`中指定的任意字符，默认为空白字符。`lstrip`、`rstrip`。

`list_strs=split(sep='')`：默认分隔符为所有空字符，包括空格、换行、制表符等。`sep`指定的字符串被视为一个分隔符，而非任意字符。若需要提供多个分隔符，使用`re.split`。

`list_strs=str.splitlines(keepends=False)`

`new_str=title()`：单词首字母大写，`capitalize()`、`swapcase()`、`lower()`、`upper()`。

##### 字符串格式化

```python
"... {} ... {:format} ...".format(args...)
"... {idx} ... {idx.member} ... {idx:format} ...".format(args...)
"... {idx.member} ... {idx[0]} ...".format(args...)
"... {var_name} ...".format(var_name=value)
```

`idx`代表参数位置，从`0`开始，没有指定则使用默认参数顺序；位置及关键字参数可以任意的结合。`format`代表格式化参数；`args`为需要被代入字符串的参数； 

##### 正则表达式

`re.match`：尝试从字符串的**起始位置**匹配一个模式，否则返回`None`。
`re.search`：扫描整个字符串并返回**第一个**成功的匹配。

```python
import re
m=re.match(pattern, string, flags=0)
m=re.search(pattern, string, flags=0)
m.span()     # 匹配位置(m.start(),m.end())
m.group()    # 匹配内容：m.group(0)
m.groups()   # 捕获组：m.group(1,2,...)
```

`m.groups() `数量与表达式中声明的捕获组`(pattern)`对应。未成功匹配的捕获组，其捕获内容返回`Null`。

> ```python
> text=r'试图打印下列文档：D:\Users\gary\Documents\Python数据类型.md'
> pattern=r'(文档：)(.*)(\.\w*)|(文档：)(.*)'
> m = re.search(pattern, text)
> ```

`re.sub`：替换字符串中的匹配项。

```python
str=re.sub(pattern, replace, string, count=0, flags=0)
```

> `replace`可以是一个函数，用于更新匹配内容，即
>
> ```python
> def replace(matched):
> to_transform
> return trans
> ```

`re.compile`编译正则表达式并检查语法，生成一个正则表达式` re.Pattern `对象，提供`match`、`search`等方法（与静态方法相同）。

```python
pat=re.compile(pattern[,flag])
```

`pattern.findall`返回所有匹配的子串并组成列表。`pattern.finditer`返回一个字串匹配对象`Match`的迭代器。

```python
pattern.findall(string[, pos[, endpos]])
for m in pattern.finditer(text):
  print(m.group(0))
```

https://www.runoob.com/python3/python3-reg-expressions.html.

字符串分割：

```python
list_strs = re.split('.|,|:|;| ', string)
```



##### JSON

编码和解码

```python
json.dumps(obj)     # 将Python类型转换为JSON字符串
json.loads(string)  # 将JSON字符串转换为Python对象
```



#### 元组（Tuple）

```python
tup = ( 'runoob', 786 , 2.23, 'john', 70.2 )
```

元组的元素不能二次赋值（`tup[i]=value`），相当于只读列表。如果元素是可变对象，可以对对象内容修改。

> `(x,)`表示只有一个元素的元组，使用`,`区分元组与普通括号运算符。

#### 列表（List）

列表可以动态添加和删除元素。

```python
list = [ 'runoob', 786 , 2.23, 'john', 70.2 ]
list = [ func(x) for x in iterable if cond(x) ] # substitute to for loops
																								# cond(x) is optional.
i = 0, list = []
for x in iterable:
  if cond(x):
    list[i] = func(x)
    i += 1
list = [ func1(x) if cond1(x) else func2(x) for x in iterable ]
```

> 使用`[]`构造列表[性能更好](https://www.datacamp.com/community/tutorials/python-list-comprehension)。

`range`类型表示不可变的数字序列，通常用于在 `for`循环中循环指定的次数。

```python
class range(start, stop[, step])
```

> 包含`start`，不包含`stop`。

##### 修改列表

`l.append(e)`：追加元素。

`l.remove(e)`：删除元素（必须在列表中的）。

##### 合并序列

```python
# Method1: def concat function with for-loop and concat operator (+)
def concat(nest_list):
	list = []
  for l in nest_list:
    list = list + l			# 合并两个列表
	return list
l_nest = [[1,2,3], ['a', 'b'], [4]]
ll = concat(l_nest)
# Method2: List constructor, append by elements
ll = [y for l in l_nest for y in l] 
# Method3: Substitute concat() with reduce()
from functools import reduce
ll = reduce(lambda x,y: x+y, l_nest)		
# Method4:
import itertools			# use itertools
flat=itertools.chain.from_iterable(l_nest)
```

> 使用对象运算符`*`复制序列，使用`+`进行序列拼接。

##### 排序

`list.sort( key=None, reverse=False)`为列表类型特有的[排序函数](#序列类型公共方法)，对原列表进行操作。

##### Get unique values from a list

https://www.geeksforgeeks.org/python-get-unique-values-list/

1.  Traversal of list `if x not in unique_list`

2. Using `set` `unique_list = (list(set(list1))) `

3. Using `numpy.unique`

   

#### 序列类型的索引

访问元素：

```python
s[idx]
```

取子串：从左到右索引默认0开始（***zero-based***）的，最大范围是字符串长度少1；从右到左索引默认-1开始的，最大范围是数组开头。（包含开始不包含结束位置的值）

```python
a[start:end+1]	# 读取从第start到第end的元素
a[start:end+1:step]# i缺省为0 j缺省为L s缺省为1
a[-j:-i]# 读取从倒数第(j-1)到倒数第i的元素: a[-i:-j:1]
a[-i:-j:-s] # 步长为负，逆序读取
a[:]	# all 
a[:-1]  # 从位置0到位置-1之前的数 a[-L:-1:-1]
a[::-1] # 反转操作：-n 读取倒数第n个元素之前的内容
```

> 超出范围的下标自动被忽略。

> 序列类型不能通过整数序列或`bool`序列选取元素（`numpy.ndarray`支持）。

#### 序列类型公共方法

##### 排序

排序结果作为新的对象返回。

```python
sorted(iterable, key=None, reverse=False)
```

`key`是一个排序元素的函数，用于排序的参考值。`sorted`不支持自定义比较函数（Python3）。

为了支持多个字段的排序，可对序列进行多次排序，即按优先级最低到最高的字段分别进行排序：

```python
def multisort(xs, specs):
    for key, reverse in reversed(specs):
        xs.sort(key=attrgetter(key), reverse=reverse)
    return xs
multisort(list(objects), (('field1', True), ('Field2', False)))
```

##### 查找（find）

```python
idx = list.index(value)  # retrun the first occurence
```

> `==`运算符不能用于序列与元素的比较，返回`bool`掩码；`==`直接比较两个对象。
>
> 如果元素不存在，返回异常。

查找多个元素：

```python
L = [1,2,3,2,1,4,5]
R = [1,3]
indices = [i for i in range(len(L)) if L[i] in R]
```

#### 复合类型

list of tuples:

```python
arrays = [['bar', 'bar', 'baz', 'baz', 'foo'],
          ['one', 'two', 'one', 'two', 'one']]
tuples = list(zip(*arrays))
```



### 映射类型

#### 字典（Dictionary）

字典当中的元素是通过键来存取的，而不是通过偏移存取。

定义字典：

```python
d = {'name': 'john','code':6734, 'dept': 'sales'}
d = dict.fromkeys(keys, value) # 初始化所有keys的值为value
d = dict(key1=val1, key2=val2, ...)
d = dict([(key1, val1), (key2,val2), ...])
d = dict(zip(keys, values))	# 使用对应的values初始化keys
```

增删字典元素：

```python
d['key'] = value
d.update(another_dict)
del(d['key'])	# 删除字典元素 
```

`dict.clear()`：清空字典元素。

遍历字典元素：

```python
# by ietrator
for key in dict:		# equals to dict.keys()
    print(key, dict[keys])
for key,value in dict.items(): # dict.items() return a tuple each time 
    print(key, value)
for key in dict.keys():
    print(key, dict[key])
for value in dict.values():
    print(value)
```

键是不可变数据类型，可以采用数值、字符串，元组； 值可以是任意类型。

```python
keys = d.keys()      # return Type <dict_keys>
vals = d.values()    # return type <dict_values>
key in dict    # 判断key是否在字典中
```

`keys()`和`values()`无法通过下标访问，可通过`list()`转换为列表读取其中的元素，或通过迭代访问，其顺序为元素加入字典的顺序。

### 集合

`set`对象是由具有唯一性的 [hashable](https://docs.python.org/zh-cn/3.7/glossary.html#term-hashable) 对象所组成的无序多项集。 常见的用途包括**成员检测、从序列中去除重复项以及数学中的集合类计算**，例如交集、并集、差集与对称差集等等。 

```python
s = {'xyz', 123, True}  # 空集合为set()，{}创建的是空字典
s = set('abcdefg')   #将字符串的字符转换为集合元素。
```

> 集合类型是无序的，其元素顺序与构造时提供的顺序不一定一致。
>
> 使用`in`/`not in`判断集合元素是否存在。

#### 修改集合

```python
s.add(x)
s.update(x)   # 参数可以是列表，元组，字典等
s.remove(x)   # 不存在会发生错误
s.discard(x)
s.pop()       # 设置随机删除集合中的一个元素
s.clear()
```

`frozenset`为对应的不可变类型。

#### 集合运算

```python
s.intersection() # 交集
s.union()  # 并集
s.difference() # 差集
s.isdisjoint() # 集合是否有没有交集
s.issubset(A)   # 判断s是否为A的子集
s.issubset(A)   # 判断s是否为A的子集
```



[Python 标准库](https://docs.python.org/zh-cn/3.7/library/index.html)

## 数组

存储同类型元素的类型，区别于list、tuple等序列类型。

- Python 标准库提供`array`类用于构造[一维数值数组](https://docs.python.org/zh-cn/3.7/library/array.html?highlight=append#array.array.append)。
- numpy提供`ndarray`(`array`)类用于构建数值数组。

### 二进制序列类型

`bytes`, `bytearray`

[`memoryview`](https://docs.python.org/zh-cn/3.7/library/stdtypes.html?highlight=list append#memoryview) 对象允许 Python 代码访问一个对象的内部数据，只要该对象支持 [缓冲区协议](https://docs.python.org/zh-cn/3.7/c-api/buffer.html#bufferobjects) 而无需进行拷贝。

#### 编码和解码

```python
str.encode()
bytes.decode()
```

##### Base64编解码

```python
import base64
x = 'this is a string'
x_bytes = x.encode('utf-8')
x_b64 = base64.b64encode(x_bytes)
x_bytes = base64.b64decode(x_b64)
x = str(x_bytes, 'utf-8')
```

#### URL编码

```python
from urllib.parse import quote,unquote
text = "丽江"
url_code_str = quote(text,'utf-8')
text_orgin = unquote(url_code_str,'utf-8')
```



### 多维矩阵

`ndarray`（`array`）类型可表示多维同构（*homogeneous*）矩阵。

#### 属性

| 名称       | 用途                                                         |
| ---------- | ------------------------------------------------------------ |
| `ndim`     | 数组的维数，例如向量为1，矩阵为2矩阵。                       |
| `shape`    | 数组的形状。使用矩阵表示向量：列向量$N\times 1$，行向量默认为$1\times N$，与仅有一个维度的向量类型不同。 |
| `size`     | 数组的元素总数                                               |
| `dtype`    | [数据类型](https://numpy.org/devdocs/user/basics.types.html)，矩阵元素不仅可以使数值类型，也可以是字符串等类型。 |
| `itemsize` | -                                                            |
| `data`     | 数据的内存空间                                               |

> `len(array)`返回数组第一维的长度。
>
> 数据类型定义包括：平台相关类型（`np.int`，`np.float`，...）以及定长类型（`np.int32`,`np.int64`, `np.float32`,`np.float64`,...,`np.bool`）。

#### 创建数组

##### 快捷方法

```python
numpy.full(shape, fill_value, dtype=None, order='C')
```

创建指定形状`shape`的数组，并指定初始值`fill_value`和元素类型`dtype`（例如`bool`）。`shape`指定数组的维度，可以是**整数标量**、**列表**或**元组**。

- 标量：为`0`表示空数组；为`1`相当于标量组成的数组，其他值则为向量；
- 长度为`2`的列表或元组：生成矩阵；
- 大于2的列表或元组：生成高阶数组。

`order`指定数据存储方式： ***row-major (C-style) or column-major (Fortran-style)***。

```python
np.ones(shape, dtype=None, order='C')
np.zeros(...)
```



```
np.empty(shape, dtype=Float, order='C')
```

创建数组，不执行初始化。

###### 数值序列

```python
np.arange(start=0, stop, step=1, dtype=None)
np.linspace(start, stop, num=50,...)
np.logspace(start, stop, num,...)
```

`arange`创建序列**不包含**`stop`，使用`step`指定步长；`linspace`包含`stop`，使用`num`指定序列元素个数。

```python
np.meshgrid(x,y,...,sparse=False,...)
```

###### 对角矩阵

`identity`，`eye`，`diag|diagonal`，`diag_flat`、

###### 三角矩阵

`tri`，`tril`，`triu`。

##### 从已有数据对象构造

```python
np.array(array_like, dtype=None, ndmin=0,...)
```

从已有矩阵或序列对象创建矩阵。嵌套序列对象可初始化为多维矩阵，`dtype`可以为每列指定一个名称和类型。

```python
np.array([(1,2,3), (4,5,6)], dtype='i4')
data = np.array(
  [(3, 'a'), (2, 'b'), (1, 'c'), (0, 'd')],
  dtype=[('col_1', 'i4'), ('col_2', 'U1')]
)
```

> 另有其他采用固定选项对`array`方法的封装方法：
>
> - `asarray`：避免兼容类型复制（`copy=False`）；
> - `asanyarray`：避免兼容类型及其子类的复制（`copy=False,subok=True`）；
> - `copy`：总是复制；
> - ...

##### 从数据源创建

###### 读取缓存数据

```python
numpy.frombuffer(buffer, dtype=float, count=-1, offset=0)
```

从字节数组读取数据并创建一维数组。`count`为读取的字节数，`offset`为跳过起始字节数。

###### 读取文件数据

使用`loadtxt`、`genfromtxt`。

##### 其他特殊数组

随机数：`np.random.randn(m, n,...)`

#### 数组对象操作

##### 复制对象

1. 不复制，直接引用

   ```
   a = b
   a = b[:,idx]
   ```

   通过[切片](#索引和切片)方式获得的数组子集仍然直接引用原数组。

2. 浅拷贝

   对象之间共享底层存储数据的内存，两个对象可对底层数据有不同的表达。

   ```python
   a = b.view()
   b = a.reshape()
   ```

   多数数组操作函数，不必要情况下都不会复制数组内容，而是使用浅拷贝。

3. 深拷贝

   ```python
   a = b.copy()
   ```

##### 初始化

```python
a.fill(value)
```

##### 数组变形

```python
a.reshape((m,n,...), order='C')
```

参数为`-1`表示给维度的长度由其他维度和数组元素数量决定。

```python
a.resize((m,n,...))
```

根据新的形状重新分配存储空间，如果数组空间减小则舍弃靠后的元素，反之将新增元素置为0。根据底层存储方式（`C`或`F`）决定元素在内存中的位置。

```python
a.ravel()		# reshape to (n,) vector
a.flatten()
a.reshape(-1)
```

返回数组元素组成的向量。

```python
a.squeeze
```

移除长度为1的维度。

```python
np.repeat(a, n_repeat, axis)
```

在给定方向`axis`，对矩阵`a`进行重复。

转置：

```python
a.transpose
```

##### 拼接

将多个数组进行垂直/水平拼接。

```python
np.concatenate((a1,a2,...),axis=0)
np.vstack(list_array)	# vertical concatenate
np.hstack(list_array) # horizontal concatenate
```

`list_array`是多个数组组成的序列（**列表或元组**），这里的数组可以是标量、Python序列以及`ndarray`。

拼接方向上各数组长度应该相同。

> 一维数组（向量）与标量只能进行水平拼接。

##### 分割

```
np.hsplit
```

##### 排序

```python
a.sort(axis,algorithm)
np.flip(a)				# sort 不提供升降序选项，默认升序，使用flip转换为降序。
a.argsort(axis,algorithm)			# 返回排序的下标
```

#### 访问数组

##### 索引

访问数组时的下标个数应与声明时的维数（`ndim`）一致。

```python
a[i] # vector
a[i,j], a[i][j] # matrix
a[(i,j)]		# matrix
```

> 元组作为下标将被展开至每个维度，因此长度需要与矩阵维数相同。

##### 切片

```python
a[I]	# sub-array, equals to a[i,;] for 2d array or a[i,:,:] for 3d array
A[I,:]  
A[:,J] 
A[I,J] # I/J is a sequece or ndarray-vector
```

> 下标可以是单个元素，也可以是列表，或`ndarray`向量。

##### 迭代

```python
for row in array:  # iterate by rows
  print(row)
for col in array.T: # iterate by columns
    print(col)
for a in array1d:  # iterate by elements
    print(a)
```



### 稀疏矩阵

`scipy.sparse`提供稀疏矩阵。

```python
As = lil_matrix(D)	# construct from 2D ndarray
As = lil_matrix(As) # construct from a sparse matrix
As = lil_matrix((M,N), dtype=None) # construct empty matrix with size and type
```

稀疏矩阵类型：

| 类型         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| `lil_matrix` | Row-based linked list sparse matrix。<br/>用于高效地逐步构建稀疏矩阵，但数值运算效率低，在数值运算前转换为其他类型。 |
| `dok_matrix` | Dictionary Of Keys based sparse matrix.<br/>用于高效地逐步构建稀疏矩阵，但数值运算效率低，在数值运算前转换为其他类型。<br/>支持$O(1)$复杂度访问元素，可以高效转换为`coo_matrix`。<br />支持矩阵元素运算。 |
| `coo_matrix` | `ijv`三元组表示稀疏矩阵，不支持算数运算，用于稀疏矩阵类型间的快速转换。 |
| `csr_matrix` | 同类型矩阵高效地数值运算；高效地行切片。                     |
| `csc_matrix` | 同类型矩阵高效地数值运算；高效地列切片。                     |

特殊稀疏矩阵：

#### 保存和加载

```python
save_npz(file_path, matrix)
matrix = load_npz(file_path)
```



## 表格

### 构造对象

#### DataFrame

##### 以序列/矩阵初始化

```python
df = pd.DataFrame(data=array, index=index_label, columns=label_list, dtype=type)
df = pd.DataFrame.from_records(data, columns)
```

以矩阵数据构造数据表，`data`可以是`list`、`tuple`及其复合类型（如`list_of_list`、`list_of_tuple`、`tuple_of_list`等）以及`ndarray`。当数据表示向量时，构造的表格为`Nx1`类型的。

`pd.DataFrame`的`dtype`参数只能为整个表格指定一种数据类型，可在创建后调用`astype`分别设置各列的数据类型。`pd.DataFrame.from_records(...)`可以指定每列的类型。


##### 以字典初始化

```python
df = pd.DataFrame({'col1':values1, 'col2':values2, ...}) 
df = pd.DataFrame.from_dict(
  {'col1': values1, 'col2': values2, ...}, 
  orient='columns'
)
df = pd.DataFrame.from_records(
  [
    {'col1': v1, 'col2': v2}, 
    {'col1': v1, 'col2': v2}
  ]
)
```

字典的键作为表格的列名，值可以是**序列类型**（`list`、`tuple`、`Series`等），作为表格对应列的数据。

`DataFrame.from_dict(...)`将字典的值（序列类型）转换为表格的行（`orient='columns'`，默认）或列（`orient='index'`）。当字典的值为标量，可使用`from_dict(d, orient='index')`得到单列的表格。



##### 读取数据源

表格通常通过从数据源导入数据的方式初始化，例如`read_csv`、`read_parquet`等。

> `read_parquet`需要`pyarrow`或`fastparquet`支持。
>
> ```python
> conda install -c conda-forge pyarrow=1.0.0
> ```
>
> 使用`conda`安装时，可能默认不会安装最近版本，导致兼容问题。

#### Series

```python
ss = pd.Series(data=None, index=None, dtype=None, name=None, copy=False)
ss = pd.Series(values, index=arrays)
```

一维序列。`copy`表示是否复制输入数据。

> 迭代`Series`对象返回每个元素。`Series`没有列索引，因此不需要传入字典进行初始化。
>
> ```python
> ss = pd.Series([1, 3, 5, np.nan, 6, 8])
> ```

### 数据属性

`shape`：行列数。

`columns`:列索引名称。

`index`：行索引名。

`empty`：数据表是否为空。

`dtypes`：每列的数据类型。

`values`：推荐使用 `.array()` or `.to_numpy()`

##### `Series`属性

```python
ss.is_monotonic  # => is_monotonic_increasing
ss.is_unique
ss.hasnans
ss.name = 'Name'  # 获取或修改Series的名称（列名）
```


#### 修改行列索引名

```python
df.columns = ['A', 'B', 'C', ...]  # 整体修改
df.rename(columns={"A": "a", "B": "c"}) # 替换修改
df.rename(index={0: "x", 1: "y", 2: "z"})
```

设置行列名称：

```python
df.rename_axis(index='index_axis_name', columns='col_axis_name')
```

> 当存在多个行列标签时，以字典给出新旧名称的映射。

### 查看数据

可以直接打印（`print`），Pandas根据窗口控件对部分内容进行省略。

```python
df.head(n=5)		# return the first n rows
df.tail()
df.describe()		# summary
print(df.to_string(index=True, header=True))  # remove dtype/header/index info
```

设置打印宽度等参数：

```python
pd.set_option('display.width', 80)
pd.set_option('display.max_rows', 60)
pd.set_option('display.max_columns', 20)
```

`pd.DataFrame.style`：*Contains methods for building a styled HTML representation of the DataFrame.*

### 索引

#### Index

表示（行/列）索引的数据类型。

```python
index = pd.Index(['e', 'd', 'a', 'b'])
index = pd.Index(list(range(5)), name='rows')
columns = pd.Index(['A', 'B', 'C'], name='cols')
```

`Index`可以被视为特殊类型的`Series`，因此`Series`多数属性、方法也适用于`Index`。

```python
df.index						# 获取DataFrame的索引
index.name
index.has_duplicates
index.is_all_dates
```


```python
loc = index.get_loc(key)
```

> Return *int if unique index, slice if monotonic index, else mask*. 

`nan`不影响*unique index*的统计（包含多个`nan`的索引调用`get_loc()`时仍返回的是标量）。


[Index索引的用途总结](https://zhuanlan.zhihu.com/p/86042616)：

- 更方便的数据查询（`df.loc[index]`）；
- 使用index可以获得性能提升：对于唯一的index使用hash表查找，对于有序index使用二分法查找；
- 自动的数据对齐功能（进行序列或表格运算时，非公共的index对应的数据计算结果为`Nan`）；
- 更多更强大的数据结构支持，包括`Categoricallndex`、`Multilndex`、`Datetimeindex`等；

#### 索引类型

##### 数值索引

##### 类别索引

##### 时间索引

`DateTimeIndex`以时间点为数据点。

```python
datetime_index = pd.date_range(
  start, end, periods, freq, tz, normalize, name, closed)
```

`start, end`：起始和结束时间；
`periods`：生成的点数；
`freq`：时间间隔，单位标识如下：

| 年   | 季   | 月   | 周   | 日   | 时   | 分   | 秒   | 毫秒 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| `Y`  | `Q`  | `M`  | `W`  | `D`  | `H`  | `T`  | `S`  | `L`  |

> 前缀`B`：仅计算工作时间，默认单位为天（`+MQY`）；`CB`：自定义工作时间；
> 后缀`S`：以周期开始作为记录点（无后缀代表以结束为记录点，`+MQY`）；
> 前缀`S`(semi)：以半周期为记录点（只用于`M`，可与其他前后缀组合使用）。

`tz`：时区，格式为`Asia/Shanghai`。
`normalize=True|False`：将起点和终点对齐当天午夜零点。
`name`：索引的列名。
`closed=None|'left'|'right'`：索引区间的开闭，默认为全闭。

`PeriodIndex`以[时间段](#时间段)为数据点。

```python
pd.period_range(start, end, periods, freq, ame)
```

`InterIndex`以区间为数据点（起始点对，可以为实数）。

```python
pd.interval_range(start, end, periods, freq, name, closed)
```

#### MultiIndex

创建`MultiIndex`：

```python
# construct from list of tuples
index=pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])
index.names		# index.name = None for MultiIndex
# from list of lists(): cross product
index=pd.MultiIndex.from_product(iterables, names=['first', 'second'])
# from DataFrame
MultiIndex.from_frame()
```

创建`DataFrame`时，可通过`index`参数提供原始数据直接创建`MultiIndex`，或通过`set_index`来创建。

##### 多级访问

当传入单个索引时，可仅指定条目的部分索引（编号最小的索引）。

```python
df.loc[l1_key, cols]   # access data with only level-1 index labels
df.loc[(l1_key, l2_key)] # access data with both 2-level index labels
df.loc[(l1_key, l2_key), cols]  # with column labels
# .loc does not accept access directly via level-2 index labels
df.xs(idx, level=idx_name)  # direct access any index level
```

当传入单个索引时，取回内容会**自动去掉传入层级的索引**，因此可以使用链式访问方式：

```python
df.loc[l1_key][cols]
df.loc[l1_key].loc[l2_key]  # chaining access
```

如果要同时访问多个条目，则必须提供每个层级完整的索引。可将索引封装为元组序列，或者将每一级索引单独提供：

```python
df.loc[[(id1,id2,...),(id1,id2,...),...],cols] # list of tuple index
df.loc[l1_idx, l2_idx, :, ...][cols] # separate index
```

当单独提供每一级索引时，不能同时包含列名，因此需要链式访问方式提供列名。

> 可重设索引（`set_index`）去掉不要的索引层级，以使用部分索引来访问内容。

The [`swaplevel()`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.MultiIndex.swaplevel.html#pandas.MultiIndex.swaplevel) method can switch the order of two levels:

https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html

获取部分索引：

```python
idx = df.index.get_level_values(level_name)  # get index of specified level
idx = df.index.droplevel(level_name)  # drop index of specified level
```

获取某一级索引对应的下标：

```python
df.index.get_loc_level(idx)  # return (tf_values, loc)
```



#### 表格索引操作方法

##### 设置索引

使用指定的一列/或多列数据设置为索引。

```python
df.set_index(keys, drop=True, inplace=False)		# 添加新的索引项
```

`keys`：如果为列名，则使用对应列移至索引列，索引列名称为对应列名；如果为与表格长度相同的列（包括`Series`、`Index`、`np.ndarray`、`Iterator`），则设置默认的索引列（无索引列名）；也可以是以上两者组合的列表（设置多重索引）。

`drop`：默认移除作为索引的列，反之移保留为索引的列；

##### 重置索引

设置为数字索引。

```python
df.reset_index(drop=False, inplace=False)				
```

`drop`：默认将原始的索引设置为新的列（**首列**），反之移除索引；

`inplace`：默认返回新的对象，反之对源对象进行修改。

##### 重排索引

按照给定的索引标签顺序重排表格内容。未给出的索引对应的表格行列将被丢弃，新增的索引对应的行列将填充指定的值`fill_value`

```python
df_new = df.reindex(labels, axis={'index'|'columns'}, fill_value=nan, ...)
df_new = df.reindex(index=idx_labels, columns=colnames, copy=True, ...)
```

> `copy=True`将返回新的表格对象。
>
> `ss_new = ss.reindex(new_index)`



### 访问数据

#### 索引或下标

**下标运算符`[]`和成员运算符`.`**：接受标签或切片对象作为参数。

> 使用`.`时，标签/列名需要是合法的Python标识符。

```python
ss = df[colname]			# return Series corresponding to colname
ss = df.<column_name>
```

> 可以使用列表传递多个列的名称。
>
> 返回新的数据对象，不能用该表达式修改原对象，而应该使用`.loc`。

`Series`的参数为行索引，而非列名（`Series`无列名）。

```python
x = ss[index_labels]	# return sca lar/Series value via index labels
x = ss.<index_name>
```

**对行进行切片**

```python
s1 = ss[i:j:s]
d1 = df[i:j:s]		# slicing rows
```

> 仅支持`:`操作符构造的参数（与Python规则一致），不支持列表作为参数（会被视为列标签）

**`loc`属性**

参数为标签，或`boolean`向量。

> 提供的参数与数据表标签类型不一致会导致报错。

```python
s.loc[index_labels]				# Series
df.loc[row_labels,column_names] # DataFrame
```

标签表示方法包括：

- 单个标签：例如`5`、`a`等；

- 序列：`['a', 'b', 'c']`；

- 切片对象：`'a':'f'`、`'d':`等，首尾包含在内；

- `boolean`向量

  可以使用`query(expr)`代替`boolean`表达式。

> `loc`属性和`[]`运算符可以增大数据表，方便增添数据。
>
> `df.loc[row_labels,:]`等价于`df.loc[row_labels]`。
>
> 返回值为单个元素时，如果参数为标量对象则返回标量，如果参数包含列表类型（即使仅有一个元素），则返回值为`Series`。
>
> 返回值为向量时，如果参数为标量，则返回值构成`Series`，反之（即使仅有一列）构成`DataFrame`。

**`iloc`属性**

参数基于整数位置（下标，和Python，Numpy的语法一致），或`boolean`向量。

> 参数表示方式对返回值的类型影响与`.loc`一致。

```python
s0 = s.iloc[indices]
df.iloc[row_indices, col_indices]
```

下标表达方式包括：

- 标量整数；

- 整数列表；

- 切片对象；

- `boolean`向量：**查询数据表返回的表达式（例如`df['A']>0`）为`Series`，不能直接用于`iloc`的下标参数**，代替方法可以先将返回值转换为矩阵：

  ```python
  df['A'].to_numpy()>0   # or
  ((df['A'])>0).to_numpy()
  ```

> `ix` is deprecated.
>
> 若返回对象是原始数据的一个切片，则其是一个浅拷贝。对返回对象的修改等效于修改原始数据。对切片赋值时推荐使用`.iloc()`，否则产生警告（*A value is trying to be set on a copy of a slice from a DataFrame.*）。
>
> 以切片索引访问表格时总是返回表格类型，而当行列索引为标量时，返回值类型会转换为`Series`；以当给定的切片索引范围不在表格内时，会返回空表格；给定单个索引不在表格内则会报错。
>
> ```python
> x = df.loc[i:j]  # x always be DataFrame
> x = df.loc[i]    # x becomes Series if i is scalar
> ```


#### 迭代

按列/行以此访问表格。

```python
for col_name in df:			# iterate column names
  print(df[col_name])
for label,series in df.iteriterms():  # df.iterms(), df.iterrows()
  print(label)
  print(s)
```

#### 快速访问单个元素

通过简化下标的判断和处理，加快访问速度。

```python
s.at[label]		# label based
s.iat[index]	# integer based
df.at[label, column_name]
df.iat[row_idx, col_idx]
```

> 

#### 随机抽样

```python
df.sample(n=None, frac=None, random_state=None, axis=None)
```

`n`和`frac`：采样个数或比列，两者不能同时设置。当`frac`未设置时，`n=1`。


### 编辑

#### 替换

```python
df.where(df_mask, new_value=nan)# replace with new_value where df_mask is true 
df.replace(value, new_value)
```

> 不像`np.where()`返回下标。

#### 排序

按照索引标签对表格的行/列排序：

```python
df.sort_index(axis=0, ascending=True, inplace=False)
```

`axis`指定按行/列排序，`ascending`指定升序/降序。

按照行/列的值对表格排序：

```python
df.sort_values(by=index, axis=0, ascending=True, inplace=False)
```

`by`指定排序的参考行/列（可指定多个）。

判断数据是否有序：

```python
pd.Index(data).is_monotonic   // or Series
```



#### 新增行列

```python
df.assign(colname=values,...)  # 添加新列。
df[new_col] = values           # scalar or vector
df.loc[index,:]=values         # scalar or vector
df.at[row_idx,col_idx] = value # scalar
```

> 只能一次新增一行或一列，如果同时新增多列会提示`KeyError: '[label] not in index'`。
>
> `iloc`、`iat`无法新增数据（`out-of-bounds`）。



#### 移除行列

```python
pd.drop(labels=None, axis=0, index=None, columns=None,inplace=False)
```

`labels`和`axis`组合使用，以确定`labels`代表的是行索引或列名。

可以同时指定`index`和`columns`以删除指定行列。

##### 查找移除重复行列

重复行：

```python
df.duplicated(subset_cols, keep)
ss.duplicated(keep)
df.drop_duplicates(...,inplace=False,ignore_index=False)
```

`keep='first'|'last'|False`：`first`将第一行以外的重复项标记为True。

移除重复列：将底层数据转职并检查其中的重复行。

```python
df.loc[:,~df.T.duplicated(keep='first')]
```



### 缺失数据

Pandas读取数据源时，默认使用`np.nan`代替缺失数据。

判断数据是否为有效数据（ 数值矩阵中`NaN` , 对象矩阵中`None` 或 `NaN`，日期中`NaT`）：

```python
df_tf = pd.isna(df)   # ==> df.isna()
df_tf = pd.notna(df)  # ==> df.notna()
```

> 静态方法还支持除Panda内置数据以外的类型，例如`ndarray`、`list`等，且返回类型与输入类型。实例类型返回值类型与实例一致。
>
> `isnull`是`isna`的别名；`notnull`是`notna`的别名。

丢弃缺失数据的行：

```python
df1.dropna(how='any')
```

填充缺失数据：

```python
df1.fillna(value=5)
```

如果某一列数据为`category`，则需要将填充的值添加到类别中。

```python
df.col_name = df.col_name.cat.add_categories("D")
df.col_name.fillna("D")
```



### 表格对象操作

表格操作只能返回新的对象。

```python
df_t = df.T		# 返回表格的转置
```



#### 拼接

拼接数据表或序列。

```python
df_new = pd.concat([df1, df2, s1, ...], axis=0, join='outer', ignore_index=False,...)
```

`axis`：`0`表示沿行索引方向拼接，`1`表示按列索引方向拼接；==当指定`axis=1`时，`DataFrame`与`Series`可按列拼接==。

`join`：`'inner'`表示取列的**交集**，`‘outer`表示取列的**并集**（缺失数据填充为`NaN`）；

`ignore_index`：如果为`True`则忽略在拼接方向上原有的索引，以`0,1,2,...`代替；

`keys`：追加最外层的索引（每个组成表格对应相同的索引元素）。当沿列索引方向拼接时，`keys`将替换相应数据表的列标签。

> 性能问题：创建新的数据表对象设计大量数据复制，因此在进行拼接多个数据表时，首先将数据表组成列表。
>
> ```python
> frames = [ process_your_file(f) for f in files ]
> result = pd.concat(frames)
> ```

`DataFrame.append`方法仅用于**在行索引方向**进行拼接。

```python
df = df1.append(df2, ignore_index=True)
```

> `DataFrame.append`仍**返回拼接后的新对象**，而不像Python列表的`append`方法对原对象进行修改。

#### 数据库方式合并

```python
result = pd.merge(left_df, right_df, how='inner', on=None, sort=True, 
                  suffixes=('_x', '_y'), validate=None,...)
```

`how`：合并方法

- `'left'`：使用左侧表的关键字。如果右侧表不存在对应的关键字值，则合并后右表列填充`NaN`；如果右表中存在多条关键字值与坐标匹配，则进行**一对多**合并；如果右表条目的关键字不匹配坐标，则丢弃相应行。
- `'right'`：使用右侧表的关键字；
- `'outer'`：使用两侧表的关键字的并集，两个表格中关键字未匹配的行填充`Nan`；
- `'inner'`：使用两侧表的关键字的交集，仅保留关键字匹配的行。

`on`：拼接参考行/列的标签，在两个表中均存在；可以使用多个标签的组合（`on=['key1','key2']`）作为参考，如果未指定参考标签，则两个表的列的交集将被所谓合并参考关键字。
`sort`：按参考标签值进行字典序排序；
`'validate'`：验证合并参考关键字是否唯一。

`DataFrame.join`方法

```python
df_new = df.join(other, on=None, how='left', ...)
```



## Dask分布式数据结构

提供并行化的Numpy数组和Pandas DataFrame对象，支持本地化并行计算。

### DataFrame

https://docs.dask.org/en/latest/dataframe.html

https://docs.dask.org/en/latest/dataframe-api.html

<img src="Python数据类型.assets/dask-dataframe.svg" alt="Dask DataFrames coordinate many Pandas DataFrames" style="zoom:25%;" />

`dask.dataframe`不支持排序操作（分布式运行耗时）。

```python
import dask.dataframe as dd
df = dd.read_parquet("/path/sub*/file*.parquet")
df = dd.from_pandas(df, nparts, nrow_perpart)

```

> `dask`的读取函数可以使用通配符匹配文件夹和文件。如果数据是从HDFS导出的分片数据，则传入父文件夹将以子文件夹名重建分片参考列。



> `nparts`指定分片数量，实际数量可能小于该值；

```python
df.to_parquet(path,
              write_index=True, # 写入索引
              partition_on=None, # 对每个分区按指定列拆分为多个文件
              write_metadata_file=True # 导出元数据(_metadata)到文件
             ) 
```

将每个分区单独写入一个文件。



#### 数据表操作

多数可执行操作为Pandas API的子集，但行为有所不同。

`df.reset_index`：与pandas不同，索引在每个分区中单独设置，因此非单调。

dask不支持为行赋值（`df.loc[i]=0`出错）；`iloc`仅支持取整列：`df.iloc[:,col_idx]`

`head(n=5, npartitions=1, compute=True)`：

`groupby`仅支持对分组后的数据进行汇聚统计，不支持获取分组子表格，不支持以列索引级别分组（`level=`）。

`to_datetime`：仅支持表格类型的数据转换（需要包含index），如Pandas/dask的`DataFrame`和`Series`等。

`df.any(axis=None).compute()`：如果`df`是列为1的表，返回的是长度为1的`Series`，而非标量（`pd.DataFrame`: `axis=None`计算所有元素返回标量）；代替方法转换为`pandas.DataFrame`或转换为`Series`。

判断是否[缺失数据](#缺失数据)：dask仅提供`notnull()`和`isnull()`方法，未实现`isna()`和`notna()`。

##### 赋值

不支持`df.loc[:,cols]=values`赋值，使用`df.loc[idx]=values`为行赋值。支持行索引和真值访问数据表的一行或多行。

```python
df1 = df.loc[df[col]<10]  # truth value必须为dask类型
```

##### 数据分区

`df = df.partitions[start:end:step]`

`n = df.npartitions`

`df.get_partition(n)`





## 数据类型转换

值类型：

```python
y = int(x,[base])		# x 为数值或字符串
y = float(x)			# x 为数值或字符串
```

, complex(real, imag), ord(x)

### 序列

字符/字符串： `str(x)`,

 repr(x), eval(str), unichr(x), chr(x), hex(x), oct(x)

元组：`tuple(s)`

列表：`list(s)`

### 非序列类型

集合：set(s), frozenset(s)

字典：dict(d)

### ndarray

```python
ndarray.astype(dtype, ...)
```

>  复制数组并作元素类型转换。

```python
list(a)
a.tolist()
```

将矩阵转换为嵌套的列表。

### 稀疏矩阵

稀疏矩阵可以在各类型间相互转换，根据面向的运算类型，选择合适的稀疏矩阵类型。

稀疏矩阵可以转换为常规矩阵。

```python
A = As.todense()
```



### 表格

##### DataFrame元素类型转换

```python
y = df.astype(dtype, copy=True)		# 转换整个数据表
y = df.astype({col1:dtype1,col2:dtype2,...}, copy=True)  # 转换指定列的类型
```

##### 序列数据转换

Pandas提供方法将标量、列表、元组一维数组或`Series`的数据类型转换，返回`ndarray`或`Series`（输入为`Series`时）

```python
pd.to_numeric(data)
index.to_series()
```

##### 转换为矩阵

```python
a = df.to_numpy() # return numpy.ndarray
a = df.lookup(indices)
a = ss.array		# get data from Index or Series, return ExtensionArray supercede .values
```

> 将数据对象转换为矩阵。

`DataFrame`对象如果保存的是同一类型数据，则`to_numpy`将返回对象的底层数据对象，可**通过底层数据对象修改**`DataFrame`对象。对于异构的数据类型，则返回矩阵的数据类型将适当调整（例如包括整数和`string`类型则返回类型为`object`，如果包含浮点数和整数，则返回类型为浮点型，**返回的数据不是`DataFrame`的底层数据对象**。

##### dask.DataFrame类型转换

```python
df.to_dask_array()
dd.to_datetime()
df.values   # ss.values, idx.values, recommend using to_numpy() and df.array
```

#### Index

`index`类型可转换为`set`。

