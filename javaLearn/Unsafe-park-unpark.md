
# LockSupport.park和unpark源码解读

>1，简化版park和unpark，方便快速理解
>2，每个线程都有一个Parker对象
>3，LockSupport.park调用逻辑和jvm底层park源码
>4，jvm底层unpark源码

## JVM中park和unpark源码稍复杂，先简化逻辑如下

## park
```
pthread_mutex_trylock(_mutex);//pthread_mutex_lock的非阻塞版本
status = pthread_cond_wait (&_cond[_cur_index], _mutex) ;//1,将mut互斥量解锁 ，再将cond条件变量加锁，线程挂起 ，2，当执行了status = pthread_cond_signal (&_cond[_cur_index]);会唤醒第一个挂起的线程，并重新获取互斥锁
status = pthread_mutex_unlock(_mutex);
_counter = 0 ;
```
 
## pthread_cond_wait (&_cond[_cur_index], _mutex) 等同如下

```
  pthread_mutex_unlock(mtx);
    pthread_cond_just_wait(cv);  //需要第一句unlock和第二句just_wait是原子的
    pthread_mutex_lock(mtx);
```

* [pthread_cond_wait原理](https://www.zhihu.com/question/24116967)


## unpark
```
status = pthread_mutex_lock(_mutex);
s = _counter;
  _counter = 1;
  if (s < 1) {
       status = pthread_cond_signal (&_cond[_cur_index]);//he pthread_cond_signal() call unblocks at least one of the threads that are blocked on the specified condition variable cond
        status = pthread_mutex_unlock(_mutex);
  }
```

## 每个线程都有一个Parker对象
>1，在Linux系统下，是用的Posix线程库pthread中的mutex（互斥量），condition（条件变量）来实现的。
>2，mutex和condition保护了一个_counter的变量，当park时，这个变量被设置为0，当unpark时，这个变量被设置为1


```
class Parker : public os::PlatformParker {
private:
  volatile int _counter ;
  Parker * FreeNext ;
  JavaThread * AssociatedWith ; // Current association

public:
  Parker() : PlatformParker() {
    _counter       = 0 ;
    FreeNext       = NULL ;
    AssociatedWith = NULL ;
  }
protected:
  ~Parker() { ShouldNotReachHere(); }
public:
  // For simplicity of interface with Java, all forms of park (indefinite,
  // relative, and absolute) are multiplexed into one call.
  void park(bool isAbsolute, jlong time);
  void unpark();

  // Lifecycle operators
  static Parker * Allocate (JavaThread * t) ;
  static void Release (Parker * e) ;
private:
  static Parker * volatile FreeList ;
  static volatile int ListLock ;

};

class PlatformParker : public CHeapObj<mtInternal> {
  protected:
    enum {
        REL_INDEX = 0,
        ABS_INDEX = 1
    };
    int _cur_index;  // which cond is in use: -1, 0, 1
    pthread_mutex_t _mutex [1] ;
    pthread_cond_t  _cond  [2] ; // one for relative times and one for abs.

  public:       // TODO-FIXME: make dtor private
    ~PlatformParker() { guarantee (0, "invariant") ; }

  public:
    PlatformParker() {
      int status;
      status = pthread_cond_init (&_cond[REL_INDEX], os::Linux::condAttr());
      assert_status(status == 0, status, "cond_init rel");
      status = pthread_cond_init (&_cond[ABS_INDEX], NULL);
      assert_status(status == 0, status, "cond_init abs");
      status = pthread_mutex_init (_mutex, NULL);
      assert_status(status == 0, status, "mutex_init");
      _cur_index = -1; // mark as unused
    }
};
```

## LockSupport.park->Unsafe.park

>1,JUC/locks LockSupport.park->sun.misc.Unsafe.park->hotspot/src/os/linux/vm/os_linux.cpp Parker::park

```
//LockSupport
 public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```

```
//hotspot/src/os/linux/vm/os_linux.cpp 
//这个应该是有不同的操作系统有其不同的实现，我们暂时就只看linux这个
void Parker::park(bool isAbsolute, jlong time) {
  // Ideally we'd do something useful while spinning, such
  // as calling unpackTime().

  // Optional fast-path check:
  // Return immediately if a permit is available.
  // We depend on Atomic::xchg() having full barrier semantics
  // since we are doing a lock-free update to _counter.
  if (Atomic::xchg(0, &_counter) > 0) return;

  Thread* thread = Thread::current();
  assert(thread->is_Java_thread(), "Must be JavaThread");
  JavaThread *jt = (JavaThread *)thread;

  // Optional optimization -- avoid state transitions if there's an interrupt pending.
  // Check interrupt before trying to wait
  if (Thread::is_interrupted(thread, false)) {
    return;
  }

  // Next, demultiplex/decode time arguments
  timespec absTime;
  if (time < 0 || (isAbsolute && time == 0) ) { // don't wait at all
    return;
  }
  if (time > 0) {
    unpackTime(&absTime, isAbsolute, time);
  }


  // Enter safepoint region
  // Beware of deadlocks such as 6317397.
  // The per-thread Parker:: mutex is a classic leaf-lock.
  // In particular a thread must never block on the Threads_lock while
  // holding the Parker:: mutex.  If safepoints are pending both the
  // the ThreadBlockInVM() CTOR and DTOR may grab Threads_lock.
  ThreadBlockInVM tbivm(jt);

  // Don't wait if cannot get lock since interference arises from
  // unblocking.  Also. check interrupt before trying wait
  if (Thread::is_interrupted(thread, false) || pthread_mutex_trylock(_mutex) != 0) {
    return;
  }
 //pthread_mutex_trylock() 是 pthread_mutex_lock() 的非阻塞版本
  //如果 mutex 所引用的互斥对象当前被任何线程（包括当前线程）锁定，则将立即返回该调用。否则，该互斥锁将处于锁定状态，调用线程是其属主。
  //返回值：pthread_mutex_trylock() 在成功完成之后会返回零。其他任何返回值都表示出现了错误
  
  int status ;
  if (_counter > 0)  { // no wait needed
    _counter = 0;
    status = pthread_mutex_unlock(_mutex);//// 可释放 mutex 引用的互斥锁对象,在成功完成之后会返回零
    assert (status == 0, "invariant") ;
    // Paranoia to ensure our locked and lock-free paths interact
    // correctly with each other and Java-level accesses.
    OrderAccess::fence();
    return;
  }

#ifdef ASSERT
  // Don't catch signals while blocked; let the running threads have the signals.
  // (This allows a debugger to break into the running thread.)
  sigset_t oldsigs;
  sigset_t* allowdebug_blocked = os::Linux::allowdebug_blocked_signals();
  pthread_sigmask(SIG_BLOCK, allowdebug_blocked, &oldsigs);
#endif

  OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
  jt->set_suspend_equivalent();
  // cleared by handle_special_suspend_equivalent_condition() or java_suspend_self()

  assert(_cur_index == -1, "invariant");
  if (time == 0) {
    _cur_index = REL_INDEX; // arbitrary choice when not timed
    status = pthread_cond_wait (&_cond[_cur_index], _mutex) ;
  } else {
    _cur_index = isAbsolute ? ABS_INDEX : REL_INDEX;
    status = os::Linux::safe_cond_timedwait (&_cond[_cur_index], _mutex, &absTime) ;
    if (status != 0 && WorkAroundNPTLTimedWaitHang) {
      pthread_cond_destroy (&_cond[_cur_index]) ;
      pthread_cond_init    (&_cond[_cur_index], isAbsolute ? NULL : os::Linux::condAttr());
    }
  }
  _cur_index = -1;
  assert_status(status == 0 || status == EINTR ||
                status == ETIME || status == ETIMEDOUT,
                status, "cond_timedwait");

#ifdef ASSERT
  pthread_sigmask(SIG_SETMASK, &oldsigs, NULL);
#endif

  _counter = 0 ;
  status = pthread_mutex_unlock(_mutex) ;
  assert_status(status == 0, status, "invariant") ;
  // Paranoia to ensure our locked and lock-free paths interact
  // correctly with each other and Java-level accesses.
  OrderAccess::fence();

  // If externally suspended while waiting, re-suspend
  if (jt->handle_special_suspend_equivalent_condition()) {
    jt->java_suspend_self();
  }
}

//thread.cpp
int JavaThread::java_suspend_self() {
  int ret = 0;

  // we are in the process of exiting so don't suspend
  if (is_exiting()) {
     clear_external_suspend();
     return ret;
  }

  assert(_anchor.walkable() ||
    (is_Java_thread() && !((JavaThread*)this)->has_last_Java_frame()),
    "must have walkable stack");

  MutexLockerEx ml(SR_lock(), Mutex::_no_safepoint_check_flag);

  assert(!this->is_ext_suspended(),
    "a thread trying to self-suspend should not already be suspended");

  if (this->is_suspend_equivalent()) {
    // If we are self-suspending as a result of the lifting of a
    // suspend equivalent condition, then the suspend_equivalent
    // flag is not cleared until we set the ext_suspended flag so
    // that wait_for_ext_suspend_completion() returns consistent
    // results.
    this->clear_suspend_equivalent();
  }

  // A racing resume may have cancelled us before we grabbed SR_lock
  // above. Or another external suspend request could be waiting for us
  // by the time we return from SR_lock()->wait(). The thread
  // that requested the suspension may already be trying to walk our
  // stack and if we return now, we can change the stack out from under
  // it. This would be a "bad thing (TM)" and cause the stack walker
  // to crash. We stay self-suspended until there are no more pending
  // external suspend requests.
  while (is_external_suspend()) {
    ret++;
    this->set_ext_suspended();

    // _ext_suspended flag is cleared by java_resume()
    while (is_ext_suspended()) {
      this->SR_lock()->wait(Mutex::_no_safepoint_check_flag);
    }
  }

  return ret;
}
```
## unpark

```
void Parker::unpark() {
  int s, status ;
  status = pthread_mutex_lock(_mutex);//该互斥锁已被锁定。调用线程是该互斥锁的属主。如果该互斥锁已被另一个线程锁定和拥有，则调用线程将阻塞，直到该互斥锁变为可用为止。
  assert (status == 0, "invariant") ;
  s = _counter;
  _counter = 1;
  if (s < 1) {
    // thread might be parked
    if (_cur_index != -1) {
      // thread is definitely parked
      if (WorkAroundNPTLTimedWaitHang) {
        status = pthread_cond_signal (&_cond[_cur_index]);
        assert (status == 0, "invariant");
        status = pthread_mutex_unlock(_mutex);
        assert (status == 0, "invariant");
      } else {
        status = pthread_mutex_unlock(_mutex);
        assert (status == 0, "invariant");
        status = pthread_cond_signal (&_cond[_cur_index]);
        assert (status == 0, "invariant");
      }
    } else {
      pthread_mutex_unlock(_mutex);
      assert (status == 0, "invariant") ;
    }
  } else {
    pthread_mutex_unlock(_mutex);
    assert (status == 0, "invariant") ;
  }
}
```