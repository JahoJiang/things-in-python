### 1. Python 是如何运行的 <div id="how-does-python-work"/>

>Python 的标准实现包括一个解释器和一套支持库。执行程序时，Python 内部会将源代码编译（翻译）成平台无关的字节码形式，（Python3.2 + 将其保存在 \__pycache__ 目录下，文件为 .pyc 格式。
>
>一旦程序编译成字节码，会交由 __PVM (Python Virtual Machine)__ 来执行。它本质上是一个迭代运行字节码指令的大循环，是所谓解释器的最后一步。
>
>因为不是采用二进制机器码，所以 Python 无法运行的像 C++ 之类一样快，因为 PVM 仍需解释字节码至机器指令。
>
>因此，Python 是一门动态语言，一切都在运行时发生，即我们只有 RunTime。



### 2. 常见数据类型及其操作 <div id="data-types"/>

#### 2.1 动态类型和共享引用 <div id="dynamic-types-and-sharing-reference"/>

Python 数据类型为动态类型，其核心类型分为可变类型和不可变类型。

不可变类型不能进行原地修改操作，有：

1. str
2. tuple
3. number

而：

1. set
2. dict
3. list

属于可变类型。

例如：

```python
a = 3 # 1
a = 5 # 2
b = a # 3
```

Python 实际上:
1. 先创建了一个 number 类型的对象，代表值 3，然后创建了一个变量 a，最后将这个变量和对象 3 进行连接。

2. 当我们修改 a 的值时，实际上，python 重新创建了一个对象 5 然后和变量 a 进行连接。所以，__类型实际上是属于对象而不是变量，变量名本身并没有类型。__

3. 运行 b = a 以后，两个变量就共享了引用，而如果对 b 重新赋值并不会影响到 变量 a，因为 number 不属于可变类型，会创建新的对象并连接到 b。

所以，__对于可变类型的共享引用要谨慎，因为会在原位置进行修改，从而影响到所有共享引用的变量。__

因此，对于判断两个变量相等：

1. ‘==’ 判断的是两个被引用的对象的值是否相同。
2. ‘is’ 判断的是是否引用同一个对象。



#### 2.2 str 字符串 <div id="str"/>

字符串是不可变类型。

1. 赋值

   ```python
   string = 'abcde'
   ```

2. 获取长度

   ```python
   len(string)
   ```

3. 字符串拼接

   ```python
   string + 'fg' # 拼接
   string * 2 # 字符串自拼接
   ```

4. 字符串遍历

   ```python
   for c in string:
     print(c, end=', ')
   ```

5. 查找

   ```python
   # 是否包含特定子串
   if 'ab' in string:
     pass
   
   # 是否以特定子串结尾
   if string.endswith('f', 0, -1):
     pass
   
   # 是否以特定子串开头
   if string.startswith('a', 0, -1):
     pass
   
   # 是否找到子串，并返回索引，找不到则返回 -1
   'abcabc'.find('abc', 0, -1)
   
   # 是否找到子串，并返回索引，找不到则引发异常
   'abcd'.index('abc', 0, -1)
   
   # 查找子串并计算出现次数 （‘子串’，‘起始查找位置’，‘结束查找位置’）
   string.count('a', 0, -1) 
   ```

6. 索引和分片

   ```python
   string[0] # 索引
   string[0:3] # 分片
   string[0::2] # 分片（带步长）
   ```

7. 字符编码转换

   ```python
   ord('a') # 获取字符 ascii 编码
   chr(97) # ascii 编码转字符
   ```

8. 非可变类型，不支持通过索引进行原地修改

   ```python
   >>> string[0] = '0' # 不能原地修改
   ---------------------------------------------------------------------------
   TypeError                                 Traceback (most recent call last)
   <ipython-input-16-ae5c72649fbc> in <module>
   ----> 1 string[0] = '0' 
   
   TypeError: 'str' object does not support item assignment # 不能原地修改
   ```

9. 对字符串进行修改

   由于字符串是不可变类型，所以这些API 都不是进行原位改动，而是返回一个新的改动后的字符串对象。

   ```python
   string.capitalize() # 字符串的首字母大写形式
   string.lower() # 字符串的全小写形式
   string.upper() # 字符串的全大写形式
   
   ' string \n'.strip() #  消除两端的指定的字符（默认为空格或换行符）或字符序列
   ' string \n'.lstrip() # strip() 的左端模式
   ' string \n'.rstrip() # strip() 的右端模式
   
   'string replace string'.replace('str', 'do') # 查找特定子串并替换为指定字符
   
   'a, b, c'.split(',') # 以指定子串对字符串进行分割 (默认)
   
   ','.join(['1', '2', '3']) # 以此字符串连接序列每个元素
   
   '%d...%-6d...%06d...%.4f...%+06.1f' % (12, 123, 123, 1.23456, 1.23456) # 字符串格式化
   ```

   



#### 2.3 set 集合 <div id="set"/>

set 是一个包含唯一的、不可变的对象的一个无序集合体，支持绝大多数 list 的操作。

```python
set1 = set() # 调用式创建集合并添加值
set1.add(1) # 添加

set2 = {1, 2, 3} 
1 in set2 # 查找（比list更快）

set1.intersection(set2) # 交集

set1.union(set2) # 并集

{c for c in 'string'} # 集合推导表达式
```



#### 2.4 list 列表 <div id="list"/>

list 是一个任意对象引用（存的是引用不是对象本身）的有序集合，长度可变，任意嵌套。

