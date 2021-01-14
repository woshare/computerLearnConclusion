# lock & synchronized


## 区别
>1，用法：1）synchronized：类，方法，代码块。lock：需要显示指定起始位置和终止位置，一般使用ReentrantLock类做为锁
>2，性能：synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。Lock用的是乐观锁方式，乐观锁实现的机制就是CAS操作，其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁