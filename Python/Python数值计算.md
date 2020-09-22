# 数学运算

## 运算符

### 算术运算符

`/`：在python 2.x中对于整数除法，只返回整数部分；在python 3.x都返回浮点数 

`**` 幂 - 返回x的y次幂：x**y

`//` 取整除 - 返回商的整数部分

`%`   取模 - 返回除法的余数

### 比较（关系）运算符

| 运算符 | 描述                                                         | 运算符        | 描 述              |
| ------ | ------------------------------------------------------------ | ------------- | ------------------ |
| `==`   | 比较对象的值是否相等，<br />使用`is`运算符确定是否引用相同对象。 | `!=` 或`<>`： | 比较对象，不等于。 |
| `>`    |                                                              | `<`           |                    |
| `>=`   |                                                              | `<=`          |                    |

### 赋值运算符

| 运算符 | 描述       | 运算符 | 描述     | 运算符 | 描述     |
| ------ | ---------- | ------ | -------- | ------ | -------- |
| `=`    |            | `+=`   |          | `-=`   |          |
| `*=`   |            | `/=`   |          | `%=`   | 取模赋值 |
| `**=`  | 幂运算赋值 | `//=`  | 整除赋值 |        |          |

### 二进制数值运算符

整数位运算按`bit`进行，`bool`类型按逻辑关系运算。注意运算符优先级。

| 运算符 | 描述 | 运算符 | 描述   | 运算符 | 描述   |
| ------ | ---- | ------ | ------ | ------ | ------ |
| `&`    |      | `|`    |        | `~`    | 非     |
| `^`    | 异或 | `<<`   | 左移位 | `>>`   | 右移位 |

> 整数的二进制表示（补码）有符号前缀位（0表示整数，1表示负数）。前缀也会参与执行位运算。



## 矩阵

### 查找筛选

```python
indices = np.where(bool_expr[, x, y])
idx_pair = np.array(indices).transpose()   # 将返回值转换为每个元素对应的坐标对
idx_pair = np.argwhere(a)   # a.argwhere()
```

返回表达式`boo_expr`的值为真的元素的下标，按行评估元素的值。返回类型为`tuple`，`indices[i]`为所有值为真的元素的第`i`维下标构成的向量。

当提供三个参数时，根据`bool_expr`的真值决定返回`x`或`y`的元素。三个参数的维度需要兼容。

查找非零元素：

```python
indices = a.nonzero()	# np.where(a!=0)
```

```python
idx_pair = np.argmax()		# see argwhere()
idx_pair = np.argmin()		# see argwhere()
```


从矩阵中筛选满足条件的元素

```python
a0 = np.extract(bool_expr,a)  # b = a[bool_expr]
```




### 计算

### 矩阵运算

按元素计算或按矩阵运算规则（如矩阵乘法）计算。

```python
A += B 		# a.add(b); -: a.substract(b)
C = A * B # a.multiply(b)  element-wise
A @ B			# 矩阵乘法 A.dot(B)
B = -A    # a.negative()
/					# a.divide(b) element-wise
//				# 向下取整整除
divmod	  # 返回商和余数
%					# mod reminder
+= ...		# 自操作运算符
pow(a,x)		# exp(a), sqrt(a), square(a)
log(x)			# log2 log10
log1p(x)		# log(1+x)
logaddexp()	# log(sum(exp()))
absolute(a) # fabs(a)
```

> `& | ^ ~ >> <<`	与Python内置运算规则一致。对应的函数形式：
>
> `bitwise_and`、`bitwise_or`、`bitwise_xor`、`invert`、`leftshift`、`right_shift`。

#### 复数运算

```python
angle(c)
real(c)
imag(c)
conj(c)			# conjugate(x), complex conjugate
```

#### 比较运算

`<`，`<=`，`>`，`>=`，`==`，`!=`

```python
np.maximum(a,b)# minimum(a,b), find max/min among a,b; preserve nan
np.fmax(a,b)		# fmin(a,b), drop nan
```

> 上述方法用于比较两个对象，返回两者之中较大（小）的元素组成的对象。

`np.nan`与任何数值计算都返回`np.nan`；

且`np.nan!=np.nan`，可以使用`np.isnan(x)`判断一个数值（矩阵）是否为`np.nan`。

> `np.isnan()`不能用于类型为`object`的矩阵。

#### 近似

```python
round(decimal)
floor
fix
ceil
trunc
```

#### 三角和双曲线函数

```python
np.sin(a)
```

### 统计


```python
np.sum(a, axis) 
np.cumsum(a)
np.max(a, axis)   # a.min(), find max/min in a
np.mean(a)
np.var(a)
np.std(a)
np.prod(a)
np.cumprod(a)
np.all(a, axis)	# return true if all is true
np.any(a, axis) # return true if any is true
```

> 数值计算函数通常不仅能用于`ndarray`，也支持`list`等序列类型，因此使用`np.func(obj)`的调用方式比`array.func()`更加通用。

`axis=None`：默认对数组所有元素执行操作，`0`（行索引）代表按列分别执行操作，`1`（列索引）代表按行执行操作，$\cdots$。

由于Python自带`max`等函数，在使用`numpy`的`max`时，需要添加包名。

### 线性代数

### 其他

```python
convolve(a,b)		# 卷积
a.clip(min,max)		#将取值裁剪到[min,max]之间
```

## 随机数

### random

### `numpy.random`

```python
array = permutaiton(x)
```

随机排列一个序列，或`range`对象（如果`x`是整数）。



## 数据表运算

### 比较大小

按元素比较：

```python
df.gt(df2)    #  eq, ne, lt, gt, le, and ge
df == val			# 比较DataFrame/Series和标量值，或类似矩阵的数据类型（Index，ndarray）
np.nan == np.nan 		# result False, df.equals返回true
```

> 当两个表格比较时，其行列标签必须相等（否则出错）。

整体比较：

```python
df.equals(df2)  
(df1==df2).all(axis=None)  #判断两个表格的数据是否完全相等。
```

`equals()`比较两个数据表是否完全相同的元素（数据类型相同，`Nan`比较为相等，表头的数据类型不需要相同）。

**注意**：`==`和`!=`是`DataFrame`的元素比较，`df.equals`是元素比较后的汇总结果，而`is`和`is not`运算符则是判断是否引用为同一对象。

### 矩阵计算

数值计算函数名称与`numpy`保持一致。在元素处理规则方面：1) 由于表格每一列数据类型可能不同，因此沿`axis=1`方向的运算可能不可用。2) `numpy`默认对所有元素执行计算，`pandas`默认对列执行计算。

```python
df.add(other, fill_value=None) #  element-wise add, sub, mul, div, mod, pow
```

`other`：标量、矩阵、序列等数值运算规则兼容的类型。

### 统计

获取数据表中的统计特征。

`sum`、`mean`、 `cumsum`() 、`cumprod`、`max`、`min`、`idmax`、`idmin`等。

```python
df.sum(axis=0,skipna=False)  # 
df.<col_name>.mean()
df['col_name'].mean()
```

> 执行数值运算时（求和、均值等），缺失数据默认按`0`处理；`skipna=True`跳过`NaN`元素。

统计频数：

```python
pd.value_counts(data:ndarray,sort=True,ascending=False)  # require 1-D array
```

按列/行统计有效数值（`non-NA`）的数量：

```python
ss = df.count(axis=0)
```

#### 分组

根据列的值（或多列值的组合）进行分组：

```python
grouped = df.groupby(by=col_names)	# return <DataFrameGroupBy> object
grouped = ss.groupby(by=sequence)	# return <SeriesGroupBy> object
grouped = df.groupby(level=[l1,l2,...])	# by index level
```

`by=func|dict|series|ndarray|label|[labels,...]`：

- 函数，则作用于对象的索引；
- `dict`或`Series`，将其值作为对应数据行的分组依据；
- `ndarray`，将每行作为对应数据行的分组依据；
- 标签或标签列表，使用待分组对象的对应==数据列或索引列==的数值作为分组依据；数据和索引不能存在同名。

> 对`Series`分组时，根据传入`sequence`的值进行分组，并应用到`Series`上。因此，对自身的值进行分组：`ss.groupby(by=ss)`。

`level={0,1,...}|name`：指定级别的索引作为分组依据

分组对象的属性包括：

- `groups`：组名（由参考列的值构成）和**分组所属元素的行索引**组成的字典；

  ```python
  group_names = grouped.groups.keys()
  idx_rows = grouped.groups.values()
  ```

- `indices`：组名和分组所属元素的行下标组成的字典；

  ```python
  group_names = grouped.indices.keys()
  idx_rows = grouped.indices.values()
  ```

通过遍历分组对象迭代器，可迅速获得分组中的数据条目：

```python
for group_name, group_frame in gs:
	group_idx = gs.groups[group_name]
	sub_group = df.loc[group_idx]	# == group_frame
	group_iloc = gs.indices[group_name]
	sub_group = df.iloc[group_iloc] # == group_frame
```

##### 对分组的计算

通过`groupby`返回的分组结果`grouped`，可以同时对各个分组的每列进行统计计算或自定义运算（`apply`）：

```python
grouped.mean()   # count, max, avg, min, sum, ...
grouped.aggregate()  # customized reduce computation
grouped.apply()  # customized map computation
```

> 如果运算规约到标量（`reduce`），则返回结果以分组名作为索引；
> 如果运算不改变规模（`map`），则返回结果与原数据表结构一致。

或单独对某列进行统计：

```python
grouped.<col_name>.count()  #
```

### 重采样

```python
df.resample(rule, axis=0, closed=None, label=None, kind=None, 
            on, level, origin='start_day', offset = None)
```

`origin={'epoch'|'start'|'start_day'|timestamp|time_str}`：`epoch`代表起始时间点为1970-01-01，`start`表示起始点为数据的第一个点，`start_day`表示起始点为数据第一个点当天的的零点。
`closed={'left|right'}`：指定采样的闭合边界；
`label={'left'|'right'}`：指定采样后的数据标签为原始数据的左/右边界；
`on`：指定采样的列代替索引；
`level`：指定采样使用的索引列；
`kind={'timestample'|'period'}`；

采样返回`Resampler`对象，可对返回的数据做进一步计算。

汇总：

```python
df.resample('3T').sum() # .asfreq() .count()
```

填充上采样后缺失信息：

```sh
df.resample('3T').pad() # .bfill()
df.resample('3T').apply(func)
df.resample('3T',convention='start') # 指定上采样后数据分配给起点或终点
```



### 查询筛选

`isin()`查询每个元素是否在指定集合中。

```python
df_tf = df.isin(values:[iterable,Series,DataFrame,dict])
```

> `in`运算符返回前置运算数是否在后置运算数（集合）中。
>
> `==`要求运算对象的维度兼容，而isin不限制参数类型。。

查找满足条件的行列。

```python
ss_tf = df[col_name]==value # return Series
idx = df.index[ss_tf]		# return index as Index
idx = np.where(ss_tf)[0]    # as ndarray-vector
idx = np.argwhere(ss_tf.to_numpy()).reshape(2)
ss.argmin()   # ss.argmax() 仅适用于数值, see also np.argmin(a)
```

获取满足条件的子表格：

```python
s = df.loc[ss_tf,'colname']	# return filtered value: 
s = df.loc[idx, 'colname']
```

#### 查找唯一项

`Series`可以判断序列是否唯一，或返回唯一元素组成的矩阵（`nx1 ndarray`）。

```python
tf = ss.is_unique
array = df[col_name].unique()		# return 
```

`DataFrame`的`groupby`方法可以通过分组后的组名确定唯一项（参考列的值），同时确定唯一项对应的行索引/下标。

#### Boolean Reduction

```python
df.all(axis=[0,1,None])  # see also df.any()
df.isna().all(axis=0)  # 判断列是否全为Nan
```



### 自定义数值计算

#### Apply/Map

```python
x=df.apply(func, axis=0, raw=False, args=(), **kwds)
x=df.apply(np.sqrt)
x=df.apply(np.sum, axis=0)		# 将执作为计算单元
def f(x,a,b=0):
  return x + a + b
x=df.apply(lambda x: f(x, a, b), axis=1) # 将行作为计算单元
x=df.apply(f, axis=1, args=(a), b=10) # 将行作为计算单元
```

对数据表的每一列（`axis=0`）/行（`axis=1`）执行计算（`func`）,并返回汇总结果。如果计算函数按元素计算则返回`DataFrame`（`applymap`专门用于按元素计算，`Series`不支持`applymap`），如果计算函数按`reduce`方式计算则返回`Series`。

`args`和`kwds`可分别向计算函数传递除计算对象以外的位置参数和键值参数。

> `apply`/`applymap`会对第一行/列调用函数两次，以确定是否能快速执行代码。
>
> 通常存在向量化的函数，因此应该优先调用向量化函数，而非`apply`/`applymap`。
>

将一列数据变换为多列：

```python
import re
from collections import defaultdict
def val_to_dict(val):
    # do some thing to construct a dict 
    return d
df['col'].apply(val_to_dict).apply(pd.Series)
```

#### Reduce/Aggregate

```python
df = df.aggregate(func, axis=0,...)   # agg
```

使用给定的`reduce`函数`func`对每个分组进行汇总计算，并将计算结果汇聚到同一个表格。支持的函数参数：

- 函数，函数名或函数和函数名的列表（混合）：将函数应用到数据的每一列，返回各个函数的计算结果。

  ```python
  func = lambda x: reducefunc(x) # feducefunc, e.g. sum,count,mean
  ```

- 列名和函数/函数名/列表组成的字典，对不同列应用不同的函数，返回结果中没有应用某函数的列的值设置为`nan`。

如果计算对象为`Series`而汇聚函数只有一个，则返回标量；如果对象为`Series`而汇聚函数有多个或对象为`DataFrame`而汇聚函数只有一个，则返回`Series`；除此之外，每个函数计算结果构成返回数据表的一行（函数名作为`index`）。

## 加速运算

### 向量运算

安装`numexpr`和`bottleneck`库：

> `numexpr`库使用 smart chunking, caching, and multiple cores ；
>
> `bottleneck`库使用`cpython`库的方法加速具有`nan`值的矩阵。

支持整数/boolean类型数据的某些二元运算加速。

1. [Accelerating Python for scientific research - Optimize your Python code for research](https://developer.ibm.com/technologies/python/articles/ba-accelerate-python/).
2. 

### dask.dataframe

#### 应用场景

##### 简单并行运算

- 按元素算术运算

  `map_partitions(func, *args, **kwargs)`：以分区为单元执行`map`操作，再将返回结果合并。

- 按行运算：`df[df.x>0]`

- 切片：`df.loc[1:10]`

- 统计聚合：`df.max()`

- 查找：`df[df.x.isin([1,2,3])]`

- 时间日期：`df.timestamp.month`

##### 智能并行运算

- groupby-aggregate：`df.groupby('x').y.max()`

  `apply-concat-apply`

- groupby-apply on index：

  > common groupby operations like `df.groupby(columns).reduction()` for known reductions like `mean, sum, std, var, count, nunique` are all quite fast and efficient.

- 统计计数：`df.x.value_counts()`

- 去重： `df.x.drop_duplicates()`

  > 对整个表格去重会很慢，因为需要交换大量数据以及进行比较。

- Join on index, join with small Pandas DataFrames/single partition Dask DataFrame

  ```python
  # Large join small
  small = small.repartition(npartitions=1)
  result = big.merge(small)
  # Join on index
  left = left.set_index('id').persist()
  left.merge(right_one, left_index=True, ...)
  ```

- 分区/列之间的按元素计算：`df.x+df.y`

- 相关系数：`df[['col1', 'col2']].corr()`

##### 需要shuffle的运算（除非在index上，否则较慢）

- groupby-apply not on index：需要设置索引
- Join not on the index：需要设置索引

> https://docs.dask.org/en/latest/dataframe-groupby.html

#### 方法

##### 汇聚

``` python
df.apply(func, axis=1, meta=('name', type))  # 仅支持axis=1（按行计算）
```

仅支持`axis=1`，需要通过`meta`关键字提供输出的元信息。