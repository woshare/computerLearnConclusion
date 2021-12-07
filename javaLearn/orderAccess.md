# Unsafe 内存屏障

## 为什么要有内存屏障，它的作用是什么，原理是什么

##  java内存屏障使用介绍
>a. 通过 Synchronized关键字包住的代码区域,当线程进入到该区域读取变量信息时,保证读到的是最新的值.这是因为在同步区内对变量的写入操作,在离开同步区时就将当前线程内的数据刷新到内存中,而对数据的读取也不能从缓存读取,只能从内存中读取,保证了数据的读有效性.这就是插入了StoreStore屏障 
>b. 使用了volatile修饰变量,则对变量的写操作,会插入StoreLoad屏障. 
>c. 其余的操作,则需要通过Unsafe这个类来执行.    
        UNSAFE.putOrderedObject类似这样的方法,会插入StoreStore内存屏障      Unsafe.putVolatiObject 则是插入了StoreLoad屏障


```

UNSAFE_ENTRY(void, Unsafe_LoadFence(JNIEnv *env, jobject unsafe))
  UnsafeWrapper("Unsafe_LoadFence");
  OrderAccess::acquire();
UNSAFE_END

UNSAFE_ENTRY(void, Unsafe_StoreFence(JNIEnv *env, jobject unsafe))
  UnsafeWrapper("Unsafe_StoreFence");
  OrderAccess::release();
UNSAFE_END

UNSAFE_ENTRY(void, Unsafe_FullFence(JNIEnv *env, jobject unsafe))
  UnsafeWrapper("Unsafe_FullFence");
  OrderAccess::fence();
UNSAFE_END

inline void OrderAccess::acquire() {
  volatile intptr_t local_dummy;
#ifdef AMD64
  __asm__ volatile ("movq 0(%%rsp), %0" : "=r" (local_dummy) : : "memory");
#else
  __asm__ volatile ("movl 0(%%esp),%0" : "=r" (local_dummy) : : "memory");
#endif // AMD64
}

inline void OrderAccess::release() {
  // Avoid hitting the same cache-line from
  // different threads.
  volatile jint local_dummy = 0;
}

inline void OrderAccess::fence() {
  if (os::is_MP()) {
    // always use locked addl since mfence is sometimes expensive
#ifdef AMD64
    __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
    __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  }

}

inline void OrderAccess::loadload()   { acquire(); }
inline void OrderAccess::storestore() { release(); }
inline void OrderAccess::loadstore()  { acquire(); }
inline void OrderAccess::storeload()  { fence(); }
```