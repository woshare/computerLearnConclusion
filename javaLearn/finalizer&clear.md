# finalizer & cleaner 

* [finalizer & cleaner 可以看看](https://www.zhihu.com/question/62953438)


## finalizer

>由于GC只能管理自动内存资源而无法管理程序所需要的各种其它资源（例如GC堆外的native memory、file handle、socket、draw device handle啥的），程序员必须要自己手动来管理这些资源。只是纯粹为了避免在对象死了之后它原本持有的这样的资源泄漏，Java才提供了finalizer机制到让用户注册 finalize() 这么个回调方法来定制GC清理对象的行为。

>确实Java语言规范里是说底层实现可以自由选择当对象可以被回收之后何时调用 finalize() ；换句话说在无限远的未来才调用 finalize() 也是符合规范的，现实来说就是等价于永远不调用 finalize() 也是符合规范的。但**靠谱的JVM实现都还是会在维持GC效率的前提下尽快对可以回收的对象去调用 finalize() 方法。**

>**现实上说是会被执行的，但没办法抽象地、严格地有什么机制去保证 finalize() 一定被调用**




## Cleaner

>在DirectByteBuff中可以看到分配堆外内存，和推外内存回收的机制


```
private final Cleaner cleaner;

 DirectByteBuffer(int cap) {                   // package-private

        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = UNSAFE.allocateMemory(size);//堆外内存分配
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        UNSAFE.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));//配置cleaner回收堆外内存
        att = null;
    }


 private static class Deallocator
        implements Runnable
    {

        private long address;
        private long size;
        private int capacity;

        private Deallocator(long address, long size, int capacity) {
            assert (address != 0);
            this.address = address;
            this.size = size;
            this.capacity = capacity;
        }

        public void run() {
            if (address == 0) {
                // Paranoia
                return;
            }
            UNSAFE.freeMemory(address);//释放内存
            address = 0;
            Bits.unreserveMemory(size, capacity);
        }

    }
```