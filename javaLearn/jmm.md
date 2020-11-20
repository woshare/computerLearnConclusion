# JMM

JMM是定义程序中变量的访问规则,线程对于变量的操作只能在自己的工作内存中进行,而不能直接对主
内存操作.由于指令重排序,读写的顺序会被打乱,因此JMM需要提供原子性,可见性,有序性保证.

![线程拷贝独享进程共享变量](./res/jvm-thread-mem.png "")

![JMM-原子性-可见性-有序性](./res/jmm-atomic-visibility-orderly.png "")