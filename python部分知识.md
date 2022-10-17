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

python中的类可以由传统方式（平时写的）和type方法创建（没咋看懂语法）

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
