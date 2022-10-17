### 作用域

- 全局变量
- 局部变量  

### 生成器函数（yield）

 
### 闭包

- 闭包可以先封装一个函数，而不是立刻执行
- 扩展函数内部的变量的生命周
  - 一般在函数内部的变量，会随着函数执行结束而销毁，但如果return inner函数，即inner函数还在使用，那么与inner同级的变量们也不会被销毁
  - 因为inner可能还会调用它的上级outer函数中的变量，所以这些变量是不能销毁的，这也就延续了这些变量的生命周期
- 应用场景：线程池、装饰器等

### 线程池

目的：实现并发

PS：在线程池中，硬要不使用闭包也行，但为了功能更好的拆分，使用闭包进行（https://www.bilibili.com/video/BV14m4y1D7gW?p=50&spm_id_from=pageDriver&vd_source=07999cb1d010fa9357b6650a0ee711a7）

- 在如上例子中，为了让下载和保存这两个功能分开，用线程池去实现下载，然后将线程池执行结果的返回值再去执行done函数，实现保存功能

```
from concurrent.futures import ThreadPoolExecutor
import threading

lock = threading.Lock()

def task(n, *args, **kwargs):
    return n

def outer(index, *args, **kwargs):
    def done(res, *args, **kwargs):
        print(index)  # 闭包此时的作用显示出来了，如果不加，那么无法将i这种外部变量注入进来使用

    return done


if __name__ == '__main__':
    pool = ThreadPoolExecutor(10)
    for i in range(50):
        with lock:
            future = pool.submit(task, i)  # 把返回值保存到某个变量中
            future.add_done_callback(outer(i))  # add_done_callback方法将这个变量自动当做参数，传入done函数中，执行


```
### 装饰器

- 用于拓展原来函数功能的一种函数，即：在不用更改原函数的代码前提下给函数增加新的功能
- 装饰器本身也是个函数
- 特殊之处在于它的返回值也是一个函数

```
def add(func,*args,**kwargs):
  def inner(*args,**kwargs):
    start=time.time()
    func()
    end=time.time()
    return 
  return inner
```

单例模式装饰器：
```
def single(cls,*args,**kwargs):
  instance={}
  def inner(*args,**kwargs):
    if cls not in instance:
      instance[cls]=cls(*args,**kwargs)
    return instance[cls]
  return inner
  
@single
class A:
  def __init__(self,x=0,y=0):
    self.xx=x
    self.yy=y
    
a1=A() 
a2=A()
 
```

### yield

- 与return类似，都可以返回值
- 不一样在于，yield可以返回多个值而且可暂停，再次执行可继续下一步操作
- return到了就停止不在继续运行

### 迭代器类 

可以直接作用于for循环的对象统称为可迭代对象：Iterable


- iter：创建迭代器，此方法需要返回对象本身，即：self
- next：访问迭代器

生成器是特殊的迭代器，包含yield


### 可迭代对象

一个类中有iter方法且返回一个迭代器对象，那么这个类创建的对象称为可迭代对象


### python元类

源码底层用的多

在python中可以基于类创建对象
- 先执行类的new方法，内部去撞见一个对象（此时是空对象）
- 再执行init初始化对象

```
class A(object):
    def __init__(self):
        pass

    def __new__(cls, *args, **kwargs):
        data = object.__new__(cls)
        return data

```

python中的类可以由传统方式（平时写的）和type方法创建（没咋看懂语法），默认由type创建，即只要是按传统方法写的，都是底层走type方式（？）
```
obj = type("Foo", (object,), {"v1": 123, "func": lambda self: 666})

# 分别表示类名、继承关系、成员

# 等价于

def Foo(object):
    v1 = 123

    def func(self):
        return 666

```

当不想默认由type创建时，用元类：元类的作用是指定类由谁来创建

```
class mytype(type):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        print(self)  # <class '__main__.foo'>

    def __new__(cls, *args, **kwargs):
        # 创建类对象

        print(cls)  # <class '__main__.mytype'>

        new_cls = super().__new__(cls, *args, **kwargs)

        print(new_cls)  # <class '__main__.foo'>

        return new_cls

    def __call__(self, *args, **kwargs):
        # 1.调用自己的那个类的new方法

        # print(self)  # <class '__main__.foo'>

        empty_obj = self.__new__(self)  # 在正常调用时候，这里的参数应该是类本身cls
        # 注意，self指的是创建出来的对象，在此时就是foo类
        # 即self.__new相当于foo.__new

        # 2.调用自己的那个类的init方法
        self.__init__(empty_obj, *args, **kwargs)  # 这里的empty_obj是foo创建的对象
        print("empty_obj=" + str(empty_obj))
        return empty_obj


class foo(object, metaclass=mytype):
    # foo 由mytype创建的，相当于foo是mytype的一个对象
    # 类中的call方法，会在类创建的实例对象加括号时候被调用
    # foo相当于一个mytype的对象
    # 那么当foo()时，相当于mytype这个类的对象加了个括号
    # 就会去调用mytype的call方法

    def __init__(self, name, *args, **kwargs):
        # print(self)  # <__main__.foo object at 0x0000026BA383C908>
        self.name = name


v1 = foo("123", "456")
print(v1)
print(v1.name)
```

### python的单下划线和双下划线

- 以双下划线开头：表示它是类中的私有变量/方法名，我们不能直接访问
  - 全保护，只有类对象自己能访问，连子类对象也不能访问到这个数据

- 以单下划线开头：一种约定，用来指定变量私有
  - 以单下划线开头_foo的代表不能直接访问的类属性，需通过类提供的接口进行访问
  - 不能用```“from xxx import *”```而导入
  - 半保护，只有类对象和子类对象自己能访问到这些变量
```

class A(object):
    def __init__(self):
        self.__name='aa'

x=A()
print(x.__name)  # 报错
print(x._A__name)

```


### python子类调用父类方法

在子类初始化方法中，通过super()函数来调用父类（超类）的初始化方法

当我们实例化一个对象时，是先执行了类的__new__方法（我们没写时，默认调用object.new）

```
比如单例模式中：super().__new__(cls)
```

### 元类实现单例模式

```
class mytype(type):
    def __init__(self,*args,**kwargs):
        print(args)
        print(kwargs)
        self.instance = None

    def __call__(self, *args, **kwargs):
        if not self.instance:
            self.instance = self.__new__(self)

        self.__init__(self.instance, *args, **kwargs)  # 这里的empty_obj是foo创建的对象

        return self.instance


class foo(object, metaclass=mytype):
    pass


class singleton(foo):
    # 直接继承用了metaclass的类，可以每次不用写metaclass
    pass


v1 = singleton()
v2 = singleton()
```

### 进程、线程、协程上下文切换的代价

- 进程
  - 进程id、状态、优先级、权限、内存（数据段和堆）、栈、指向可执行程序文件的描述符、打开的文件描述符、寄存器、程序I/O状态
  - 用户态 → 内核态 → 用户态
- 线程
  - 状态、优先级、栈、寄存器
  - 用户态 → 内核态 → 用户态
- 协程
  - 栈、寄存器


### 上下文切换为什么要陷入内核？

操作系统保持跟踪进程运行所需的所有状态信息（称为上下文）

- 上一个进程的上下文信息还在内存和处理器当中，我们要保存这些信息的话，就必须陷入到内核态才可以
- 创建一个新的进程，以及它的上下文信息，并且将控制权交给这个新进程，这些都只有在内核态才能实现

在当今互联网高并发场景下，为每个任务都创建一个线程是不现实的，因为会消耗大量的内存
- 进程虚拟内存会占用 4GB [32 位操作系统]
- 线程也要大约 4MB

即：系统线程会占用非常多的内存空间，过多的线程切换会占用大量的系统时间

```
协程运行在线程之上

当一个协程执行完成后，可以选择主动让出，让另一个协程运行在当前线程之上

协程并没有增加线程数量，只是在线程的基础之上通过分时复用的方式运行多个协程

而且协程的切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多
```

一个 “用户态线程”（协程） 必须要绑定一个 “内核态线程”，但是 CPU 并不知道有协程的存在，它只知道它运行的是一个 “内核态线程”

### 协程

被称为用户线程，或者说是一种用户态上下文切换的技术

用一个线程（？）实现代码块相互切换执行：先执行某个函数，然后切换到另一个函数执行一点，再切换回去执行一点，实现在函数之间来回切换，而不是顺序执行完一个函数再执行另一个函数

python实现协程方法：
- 通过第三方模块：greenlet（早期模块）
- 用yield关键字
- asyncio模块：通过装饰器来实现
- 目前用的多的：用async/await关键字等进行实现


```
import greenlet


# 实现协程

def func1():
    print(1)
    gr2.switch()  # 切换到协程2，即func2函数
    print(2)
    gr2.switch()  # 切换到协程2，从上一次执行的位置继续执行


def func2():
    print(3)
    gr1.switch()  # 切换到协程1，从上一次执行的位置继续执行
    print(4)


gr1 = greenlet.greenlet(func1)
gr2 = greenlet.greenlet(func2)

gr1.switch() # 执行协程1

```
