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

- 本身是个函数
