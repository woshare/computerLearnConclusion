
* [yield,sleep,object.wait,Unsafe.park](https://juejin.cn/post/6844903971463626766)
* [操作系统进程调度](https://zhuanlan.zhihu.com/p/148632752)
  
## yield

```
JVM_ENTRY(void, JVM_Yield(JNIEnv *env, jclass threadClass))
  // ...
  if (os::dont_yield()) return;
  // ...
  if (ConvertYieldToSleep) {
    os::sleep(thread, MinSleepInterval, false); // 使用sleep替代
  } else {
    os::yield(); // 默认调用os的yield实现
  }
JVM_END


void os::yield() {
  sched_yield();//这是一个linux的系统调用
}

```
>这是一个linux的系统调用，下面是相关的内核代码

```
SYSCALL_DEFINE0(sched_yield)
{
  do_sched_yield();
  return 0;
}

static void do_sched_yield(void)
{
  // ...
  current->sched_class->yield_task(rq);
  // ...
  schedule();
}

```


## thread.sleep
>其实最后还是调用的pthread_cond_timedwait

```
JVM_ENTRY(void, JVM_Sleep(JNIEnv* env, jclass threadClass, jlong millis))
  // ...
  if (millis == 0) {
    if (ConvertSleepToYield) { // 默认是false
      os::yield();
    } else {
      ThreadState old_state = thread->osthread()->get_state();
      thread->osthread()->set_state(SLEEPING);
      os::sleep(thread, MinSleepInterval, false); // 小睡一下
      thread->osthread()->set_state(old_state);
    }
  } else {
    ThreadState old_state = thread->osthread()->get_state();
    thread->osthread()->set_state(SLEEPING);
    if (os::sleep(thread, millis, true) == OS_INTRPT) {
      // 处理中断
    }
    thread->osthread()->set_state(old_state);
  }
  // ...
JVM_END

```

>调用了os::sleep函数（JVM实现的os，并不是操作系统的sleep），linux平台的实现代码如下

```
int os::sleep(Thread* thread, jlong millis, bool interruptible) {
  ParkEvent * const slp = thread->_SleepEvent ;
  if (interruptible) {
    jlong prevtime = javaTimeNanos();

    for (;;) {
      if (os::is_interrupted(thread, true)) { 
        return OS_INTRPT;
      }

      jlong newtime = javaTimeNanos();

      if (newtime - prevtime < 0) {
        // ...
      } else {
        millis -= (newtime - prevtime) / NANOSECS_PER_MILLISEC;
      }
      
      if(millis <= 0) {
        return OS_OK;
      }
      // ...
      {
        // ...
        slp->park(millis);  // 调用的是os::PlatformEvent::park
        // ...
      }
    }
  } else {
    // ...
  }
}

```
>最终是调用ParkEvent的park函数，实现如下


```
int os::PlatformEvent::park(jlong millis) {
  int v ;
  for (;;) {
      v = _Event ;
      if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ; // cas设置_Event
  }
  if (v != 0) return OS_OK ;  // os::PlatformEvent::unpark的时候会设置_Event=1，这里就会提前跳出
  struct timespec abst;
  compute_abstime(&abst, millis);  // 0. 计算绝对时间

  int ret = OS_TIMEOUT;
  int status = pthread_mutex_lock(_mutex);  // 1. 加mutex锁
  // ...
  ++_nParked ;

  while (_Event < 0) {
    status = os::Linux::safe_cond_timedwait(_cond, _mutex, &abst); // 2. 等待
    if (status != 0 && WorkAroundNPTLTimedWaitHang) {
      pthread_cond_destroy (_cond);
      pthread_cond_init (_cond, os::Linux::condAttr()) ;
    }
    if (!FilterSpuriousWakeups) break ;                 // previous semantics
    if (status == ETIME || status == ETIMEDOUT) break ;
  }
  --_nParked ;
  if (_Event >= 0) {
     ret = OS_OK;
  }
  _Event = 0 ;
  status = pthread_mutex_unlock(_mutex); // 3. 释放mutex锁
  // ...
  return ret;
}

```
>熟悉的味道吧，上面是一个典型的Mesa Monitor条件等待代码了，其中os::Linux::safe_cond_timedwait的代码比较简单，就是调用了pthread_cond_timedwait函数

```
int os::Linux::safe_cond_timedwait(pthread_cond_t *_cond, pthread_mutex_t *_mutex, const struct timespec *_abstime)
{
   if (is_NPTL()) {
      return pthread_cond_timedwait(_cond, _mutex, _abstime);
   } else {
      int fpu = get_fpu_control_word();
      int status = pthread_cond_timedwait(_cond, _mutex, _abstime);
      set_fpu_control_word(fpu);
      return status;
   }
}

```