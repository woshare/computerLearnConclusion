

* [yield,sleep,object.wait,Unsafe.park](https://juejin.cn/post/6844903971463626766)

## 

```
static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
};
```


## object.wait

```
JVM_ENTRY(void, JVM_MonitorWait(JNIEnv* env, jobject handle, jlong ms))
  JVMWrapper("JVM_MonitorWait");
  Handle obj(THREAD, JNIHandles::resolve_non_null(handle));
  JavaThreadInObjectWaitState jtiows(thread, ms != 0);
  if (JvmtiExport::should_post_monitor_wait()) {
    JvmtiExport::post_monitor_wait((JavaThread *)THREAD, (oop)obj(), ms);
  }
  ObjectSynchronizer::wait(obj, ms, CHECK);
JVM_END


```
>显而易见，Object::wait是配合synchronized使用的，对应的代码是在synchronizer.cpp中，其中的wait实现代码如下

```
void ObjectSynchronizer::wait(Handle obj, jlong millis, TRAPS) {
  if (UseBiasedLocking) {
    BiasedLocking::revoke_and_rebias(obj, false, THREAD);
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
  }
  if (millis < 0) {
    TEVENT (wait - throw IAX) ;
    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(), "timeout value is negative");
  }
  ObjectMonitor* monitor = ObjectSynchronizer::inflate(THREAD, obj()); // 膨胀为重量级锁
  DTRACE_MONITOR_WAIT_PROBE(monitor, obj(), THREAD, millis);
  monitor->wait(millis, true, THREAD); // 调用wait

  dtrace_waited_probe(monitor, obj, THREAD);
}

```
>首先撤销偏向锁，然后膨胀为重量级锁，再调用ObjectMonitor的wait函数。忽略ObjectMonitor复杂的实现机制，我们只看关键的地方，如下所示
>1. 添加到ObjectMonitor的等待队列_WaitSet中
>2. 释放java的monitor锁（也就是monitorexit）
>3. 等待，和Thread::sleep一样的  ，最后也都是pthread_cond_wait or pthread_cond_timewait,所以和sleep的差别就是会释放锁
```
void ObjectMonitor::wait(jlong millis, bool interruptible, TRAPS) {
  Thread * const Self = THREAD ;
  // ...
  if (interruptible && Thread::is_interrupted(Self, true) && !HAS_PENDING_EXCEPTION) {
     // ... 
     THROW(vmSymbols::java_lang_InterruptedException()); // 处理中断
     return ;
   }
  // ...
  AddWaiter (&node) ; // 1. 添加到ObjectMonitor的等待队列_WaitSet中
  // ...
  exit (true, Self) ; // 2. 释放java的monitor锁（也就是monitorexit）
  // ...
       if (interruptible && 
           (Thread::is_interrupted(THREAD, false) || 
            HAS_PENDING_EXCEPTION)) {
           // Intentionally empty
       } else if (node._notified == 0) {
         if (millis <= 0) {
            Self->_ParkEvent->park () ; 
         } else {
            ret = Self->_ParkEvent->park (millis) ; // 3. 等待，和Thread::sleep一样的  ，最后也都是pthread_cond_wait or pthread_cond_timewait
         }
       }
  //...
}

```

## notify

```
void ObjectSynchronizer::notify(Handle obj, TRAPS) {
 if (UseBiasedLocking) {
    BiasedLocking::revoke_and_rebias(obj, false, THREAD);
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
  }

  markOop mark = obj->mark();  //对象头
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    return;
  }
  ObjectSynchronizer::inflate(THREAD, obj())->notify(THREAD);
}

```