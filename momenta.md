### 1. 软件开发流程

规划需求、设计模式、开发、功能测试、端到端测试、用户验收测试、上线

### 2. 软件测试流程

需求分析、测试方案、测试用例、执行

### 3. ROS操作系统

ROS是一个用于开发机器人应用程序的、类似操作系统的机器人软件平台

### 4. Linux 用户授权的方法

chmod [参数] [文件名/目录名]

- root用户：root的权限是最高的，也被称为超级权限的拥有者
- 普通用户

每个用户都有一个ID号，称为UID，操作系统通过UID识别用户

### 5.Linux 软连接、硬链接




### 线程共享哪些资源，哪些不共享
共享的有：
- 堆
- 全局变量
- 静态变量
- 文件、内存等共用资源

独享的有：
- 栈
- 寄存器


### Python 线程安全的单例模式
```
    import threading

    class A(object):
        aa = None
        lock = threading.Lock()

        def __new__(cls, *args, **kwargs):
            with cls.lock:
                if not cls.aa:
                    cls.aa = object.__new__(cls)
            return cls.aa


    def task():
        obj = A()
        print(id(obj))


    if __name__ == '__main__':
        for i in range(3):
            t = threading.Thread(target=task)
            t.start()

```
### Python 匿名函数：lambda

不通过def来声明函数名字，而是通过lambda关键字来定义的函数称为匿名函数

### 生产者、消费者进程问题

进程 A 必须先生产了数据，进程 B 才能读取到数据，所以执行是有前后顺序的

那么这时候，就可以用信号量来实现多进程同步的方式，我们可以初始化信号量为 0
- 如果进程 B 比进程 A 先执行了，那么执行到 P 操作时，由于信号量初始值为 0，故信号量会变为 -1，表示进程 A 还没生产数据，于是进程 B 就阻塞等待
- 接着，当进程 A 生产完数据后，执行了 V 操作，就会使得信号量变为 0，于是就会唤醒阻塞在 P 操作的进程 B
- 最后，进程 B 被唤醒后，意味着进程 A 已经生产了数据，于是进程 B 就可以正常读取数据了

### 红黑树


