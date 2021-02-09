# atomic



## AtomicIntegerArray
>1，其中array 是final int[],那如何保证线程安全呢？如下举例解答

```
1，数组寻址 数组寻址[i]位置地址 = 数组初始偏移+元素大小*i;(数组是连续的内存空间)
2，先获取元素地址指向的value
3，通过CAS的方式，去设置新value

 public final int getAndAdd(int i, int delta) {
        return unsafe.getAndAddInt(array, checkedByteOffset(i), delta);
    }
    private long checkedByteOffset(int i) {
        if (i < 0 || i >= array.length)
            throw new IndexOutOfBoundsException("index " + i);

        return byteOffset(i);
    }
    //数组寻址 数组寻址[i]位置地址 = 数组初始偏移+元素大小*i;(数组是连续的内存空间)
    //a[i]地址=i<<偏移量 + a[0]首地址
    private static long byteOffset(int i) {
        return ((long) i << shift) + base;
    }

//Unsafe.java：
 public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!compareAndSwapInt(o, offset, v, v + delta));
        return v;
    }
```

## Unsafe
>通常，我们在Java中创建的对象都处于堆内内存（heap）中，堆内内存是由JVM所管控的Java进程内存，并且它们遵循JVM的内存管理机制，JVM会采用垃圾回收机制统一管理堆内存。与之相对的是堆外内存，存在于JVM管控之外的内存区域，Java中对堆外内存的操作，依赖于Unsafe提供的操作堆外内存的native方法。

### 使用堆外内存的原因
>1,对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在GC时减少回收停顿对于应用的影响。
>2,提升程序I/O操作的性能。通常在I/O通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存
>3,典型应用：DirectByteBuffe
![Alt text](./res/java-unsafe-struct.png "")

* [unsafe-讲解的不错](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)