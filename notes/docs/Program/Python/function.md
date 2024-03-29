# 函数

```python
# 关键字def用于声明函数
# 函数在被调用前都是不存在的
def func_name(param1, param2):
    pass

    # 返回值
    # 可以参略，省略则默认返回None
    # 也同时返回多个，多个则以元组形式返回
    return param1, param2

# 调用函数
func(1,2)
```

## 参数类型

首先要区分形参和实参，形参就是定义函数时候的参数，实参是调用函数时传入的参数。

```python
# x,y,z都是形参
def f1(x,y,z):
    pass
    # 参数可以传递给内部函数
    def f2(x):
        pass

# 1,2,3是实参
func(1,2,3)
```

形参可分为固定的和动态的两种。

### 固定参数

固定形参的位置/顺序和个数都是确定的，必须要一对一的传入实参，所以也被叫做位置/顺序/必传参数。

```python
# x,y,z都是固定的
def func(x,y,z):
    pass

# 实参与形参的位置和个数都要一一对应
func(1,2,3)
# 如果以关键字方式传入，位置可以随意
func(y=2,z=3,x=1)
```

```python
# 可以给固定参数设置默认值
# 必须要位于没有默认值参数的后面
# 如果有多个时，建议将不易变的置后
# 且建议不要设置可变的默认值
def func(x,y,z=3):
    pass

# 设了默认值的形参，可不传实参
func(1,2)
# 设了默认值的形参，可传入其它实参覆盖默认值
func(1,2,8)
```

```python
# 可以让固定参数强制以关键字的方式传入，前面加一个：*,
def func(x,y,*,z=3):
    pass

# 必须以关键字的方式给参数z传值
func(1,2,z=8)
```

### 动态参数

动态参数的个数是不固定的，位置也随个数的变化而不确定。

有两种接受实参的形式：

#### `*args`

```python
# 以tuple的形式接受实参
def func(*args):
    pass

# 可以传入任意个数的实参，会被以元组的形式传入
func(1,2,3)  # args=(1,2,3)

# 还可以通过解包的方式传入任意可迭代对象
func(*X)

_str="123"  # args=('1', '2', '3')
_tuple=(1,2,3)  # args=(1, 2, 3)
_list=[1,2,3]  # args=(1, 2, 3)
_set={1,2,3}  # args=(1, 2, 3)
_dict={"a":1, "b":2, "c":3}  # args=('a', 'b', 'c')
```

#### `**kwargs`

```python
# 以dict的形式接受实参
def func(**kwargs):
    pass

# 可以用关键字的方式传入
func(a=1,b=2,c=3)  # kwargs={'a': 1, 'b': 2, 'c': 3}

# 还可以通过解包字典的方式传入
_dict={"a":1, "b":2, "c":3}
func(**_dict)  # kwargs={'a': 1, 'b': 2, 'c': 3}
```

### 参数混合使用

动态参数必须要在固定参数后面

```python
# *args后的固定参数，都被视作关键字参数，且可以省略*号
def func(x,*args,z=0):
    pass

func(1,2,3,4,z=8)
```

```python
# **kwargs必须在*args后面
# **kwargs之后不允许再设置任何参数
def func(x,*args,**kwargs):
    return x, args, kwargs

func(1,2,3)  # (1, (2, 3), {})
func(1,2,3,a=4,b=5)  # (1, (2, 3), {'a': 4, 'b': 5})
```

### 参数的多态性

由于Python函数的参数没有类型限制，所实际使用中可能会带来一些问题，所以必要时建议在函数开头加上数据类型检查

```python
def func():
    if not isinstance(args, args_type):
        pass
        return
    pass
```

## 作用域

### 局部变量

函数内定义的变量默认都是局部变量，在函数外部无法访问函数内部变量和函数

```python
# 全局变量
a = 1

def f1():
    # 局部变量
    a = 2  # 可以和全局变量名字相同，但不建议这样

print(a)  # 1，全局变量没有被重新赋值
```

### 全局变量

函数内部可以访问全局变量

```python
a = 1

def f1():
    print(a)  # a=1
    x = a
    return x  # x=1
```

但函数内无法对全局变量重新赋值

```python
# 全局变量
a = 1
b = {}

def f1():
    a = a + 1  # 定义局部变量a，"a" is unbound，然后+1赋值给自己，这样是有语法错误的

# 函数参数的传递是赋值传递，即形参copy了实参的值，并不会改变指向实参的变量
def f2(a):
    a = a + 1  # 定义局部变量a，形参a+1，赋值给局部变量a，没问题

def f3():
    b["k1"] = "v1"  # dict是可变的，增加元素不算重新赋值
```

函数内如果想改变全局变量，需要使用`global`关键字

```python
# 全局变量
a = 1

def f1():
    global a  # 这里声明了a是全局变量a
    a = a + 1  # 这样就没问题了
```

嵌套的内层函数如果想改变外层函数的变量，需要使用`nonlocal`关键字

```python
# 全局变量
a = 1

def f1():
    # 局部变量
    a = 2

    def f2():
        nonlocal a  # 这里声明了a是外层函数a，即局部变量a
        a = a + 1

    r = f2()
    return r  # a=3
```

## 匿名函数(lambda表达式)

`f = lambda 输入 : 输出`

```python
f = lambda x:x+1
f(2)  # 3

# 等价于
def f(x):
    return x+1

f(2)  # 3

# 也可以直接写成
(lambda x:x+1)(2)  # 3

# 没有参数时
f = lambda :2  # f() 返回2，可以用来返回另一个函数等
```

## 函数也是对象

函数式编程：

> 函数式编程是一种抽象程度很高的编程范式（如何编写程序的方法论），属于"结构化编程"的一种，鼻祖语言：Lisp。
>  
> 纯粹的函数式编程语言编写的函数是没有变量的，而Python允许使用变量，所以Python不是纯函数式编程语言，只是部分支持。
>  
> 参考：[函数式编程初探·廖雪峰](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)

### 函数可以赋值给变量

```python
def f1(x):
    print(x)

# 引用，而没有直接调用，有点像类的实例化
r = f1
r('hello world')
```

### 函数的内部可以是函数

函数内可以嵌套函数，使用嵌套函数，能保证数据的隐私性（不能被外界直接调用），提高程序运行效率。

```python
# 外层函数
def print_msg():
    msg = "zen of python"

    # 嵌套函数
    def printer():
        print(msg)

    printer()  # 调用内部函数

another = print_msg()  # 直接输出msg
```

- 递归函数

如果嵌套的是自己，则称之为递归函数。

不同性能的计算机允许的最大递归次数也不同。过深的调用会导致栈溢出。

### 函数的返回可以是函数

函数可以返回一个外部的函数，也可以返回一个内部的函数。

- 闭包

如果返回了内部的函数，且内部函数使用了外部函数的变量，那么称之为闭包。相当于间接实现了从外部访问函数的局部变量

可以简化程序的复杂度，提高可读性。如果类只有一个方法时，可以考虑使用闭包替代。

```python
# 外层函数
def func_wai():
    msg = "hello wold"

    # 内层函数
    def func_nei():
        # 使用了外部函数的变量，形成闭包
        print(msg)

    # 返回内部函数，不要加()
    return func_nei

r = func_wai()
r()
```

### 函数的参数可以是函数

> 被当作参数的函数通常被称之为回调函数

- `map(f, Iterable)`

用法：将可迭代对象中的每个元素，都作为参数传入函数执行，最后返回一个新的可迭代对象。

```python
_list = [1,2,3]

r = map(lambda x:x**2, _list)  # 返回一个可迭代对象：<map object at 0x032A0DD0>
print(list(r))  # [1, 4, 9]
```

- `filter(f, Iterable)`

用法：可迭代对象中的每个元素，经过函数判断，返回True或False，最后将返回True的元素组成一个新的可迭代对象。

```python
_list = [1, 2, 3, 4, 5]

def f(x):
    return x % 2 == 0

r = filter(f, _list)  # 返回一个可迭代对象：<filter object at 0x02FE0DD0>
print(list(r))  # [2, 4]
```

- `reduce(f, Iterable)`

用法：将一个集合传入函数做一些累积操作。

```python
# 需要先引入
from functools import reduce

# 集合的元素个数没有要求，可以是1个
_str = "123"  # "1234"
_list = [1,2,3]  # 6
_tupe = (1,2,3)  # 6
_set = {1,2,3}  # 6
_dict = {"a":1, "b":2}  # "ab"

r = reduce(lambda x,y:x+y, _list)  # 直接返回结果
```

## 装饰器 decorator

装饰器是一种语法糖，可以使代码松耦合，能够在不修改原有业务逻辑的情况下对代码进行扩展。

一个函数前面可以加若干装饰器，类也有装饰器

```python
# 装饰器函数，参数，内容，返回都是函数
def decorator(func):
    # 闭包
    def wrapper(*args, **kwargs):
        print("前置扩展内容，调用装饰器，实现一些权限校验、用户认证等功能")

        demo(*args, **kwargs)  # 使用装饰器的函数被传入到这里，使用可变参数让其更加灵活

        print("后置扩展内容，调用装饰器，实现一些日志记录、性能测试、事务处理、缓存等")

    return wrapper

# 调用装饰器
"""最本质的方式"""
def demo(x):
    pass

decorator(demo_func)(x)

"""语法糖"""
@decorator
def demo(x):
    pass

demo(x)
```

参考：[Python核心技术与实战-强大的装饰器（极客时间）](https://time.geekbang.org/column/article/100914?utm_source=infoq&utm_medium=sitenavigation)

## 迭代器

### 构建方式1：iter(Iterable)

```python
iter("123")  # <str_iterator object at 0x10de4aa10>
iter([1,2,3])  # <list_iterator object at 0x10de4aa10>
iter((1,2,3))  # <tuple_iterator object at 0x10de4a950>
iter({1,2,3})  # <set_iterator object at 0x10de4bf50>
iter({"a":1,"b":2})  # <dict_keyiterator object at 0x10de469b0>
```

### 构建方式2：生成器

生成器是一种特殊的迭代器，但是不需要像迭代器一样实现__iter__()和__next__()方法

- 构建方式1：列表推导式的[]换成()

```python
g = (x for x in range(5))
```

- 构建方式2：函数的return换成yield

```python
def fib(max):
    n = 0
    a,b = 0,1
    while n < max:
        if (counter > n): 
            return  # 函数执行完return语句就结束了
        yield b  # 但函数执行完yield语句只是暂停
        # 执行到此暂停，对象存储到迭代器数据流中
        # 调用一次next()，将生成器对象返回，然后继续执行后面的代码，循环到此，再一次暂停
        a, b = b, a + b
        n = n + 1

g = fib(5)  # <generator object demo at 0x100d55a50>
next(g)  # 获取生成器对象的值，['...']
```

```python
# 消费者
def consumer():
    message = "start"
    while True:
        receiver = yield message  # yield语句执行完，返回message值，然后暂停，可以接收send发送的消息，下一轮启动然后赋值给receiver
        print("receiver: ", receiver)  # 等待下一次启动才会执行赋值语句
        if not receiver:
            return "end"
        message = "200 OK"  # 执行完这里，会继续返回执行 yield message，也就是说此函数每一轮返回的都是message的值


if __name__ == '__main__':
    gen = consumer()  # <generator object consumer at 0x100d55a50>
    print(next(gen))  # 预激
    """
    在一个生成器函数未启动之前，是不能传递值进去
    也就是说在使用gen.send(x)之前，必须先使用 next(gen) 或者 gen.send(None) 来返回生成器的第一个值
    send() 类似 next()，但可以传递参数，发送消息：gen.send("message")
    """
    # 模拟生产者
    print(gen.send(1))  # 200 OK
    print(gen.send(2))  # 200 OK
    print(gen.send(3))  # 200 OK
    print(gen.send(None))  # StopIteration: end
```

### 调用方式

```python
# 每调用一次，则迭代一次，无可迭代最后会抛出StopIteration错误
r = next(g)

# 遍历
r = [i for i in g]

list(gen)  # 也可以将生成器转为list
```
