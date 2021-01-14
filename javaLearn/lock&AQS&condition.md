# lock & AQS & Condition

* [lock&await&signal-不错](https://juejin.cn/post/6844903665128439815#heading-6)

>1，void await() throws InterruptedException 当前线程进入等待状态，直到被通知（signal）或者被中断时，当前线程进入运行状态，从await()返回
>2，void signal() 唤醒一个等待在Condition上的线程


```
public final void await() throws InterruptedException {
    // 1.如果当前线程被中断，则抛出中断异常
    if (Thread.interrupted())
        throw new InterruptedException();
    // 2.将节点加入到Condition队列中去，这里如果lastWaiter是cancel状态，那么会把它踢出Condition队列。
    Node node = addConditionWaiter();
    // 3.调用tryRelease，释放当前线程的锁
    long savedState = fullyRelease(node);
    int interruptMode = 0;
    // 4.为什么会有在AQS的等待队列的判断？
    // 解答：signal操作会将Node从Condition队列中拿出并且放入到等待队列中去，在不在AQS等待队列就看signal是否执行了
    // 如果不在AQS等待队列中，就park当前线程，如果在，就退出循环，这个时候如果被中断，那么就退出循环
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 5.这个时候线程已经被signal()或者signalAll()操作给唤醒了，退出了4中的while循环
    // 自旋等待尝试再次获取锁，调用acquireQueued方法
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

```
>1，将当前线程加入Condition锁队列。特别说明的是，这里不同于AQS的队列，这里进入的是Condition的FIFO队列。 
>2，释放锁。这里可以看到将锁释放了，否则别的线程就无法拿到锁而发生死锁。 
>3，自旋(while)挂起，直到被唤醒（signal把他重新放回到AQS的等待队列）或者超时或者CACELLED等。 
>4，获取锁(acquireQueued)。并将自己从Condition的FIFO队列中释放，表明自己不再需要锁（我已经拿到锁了）。


```
public final void signal() {
            if (!isHeldExclusively())
              //如果同步状态不是被当前线程独占，直接抛出异常。从这里也能看出来，Condition只能配合独占类同步组件使用。
                throw new IllegalMonitorStateException(); 
            Node first = firstWaiter;
            if (first != null)
                //通知等待队列队首的节点。
                doSignal(first); 
        }

private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&   //transferForSignal方法尝试唤醒当前节点，如果唤醒失败，则继续尝试唤醒当前节点的后继节点。
                     (first = firstWaiter) != null);
        }

    final boolean transferForSignal(Node node) {
        //如果当前节点状态为CONDITION，则将状态改为0准备加入同步队列；如果当前状态不为CONDITION，说明该节点等待已被中断，则该方法返回false，doSignal()方法会继续尝试唤醒当前节点的后继节点
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        Node p = enq(node);  //将节点加入同步队列，返回的p是节点在同步队列中的先驱节点
        int ws = p.waitStatus;
        //如果先驱节点的状态为CANCELLED(>0) 或设置先驱节点的状态为SIGNAL失败，那么就立即唤醒当前节点对应的线程，线程被唤醒后会执行acquireQueued方法，该方法会重新尝试将节点的先驱状态设为SIGNAL并再次park线程；如果当前设置前驱节点状态为SIGNAL成功，那么就不需要马上唤醒线程了，当它的前驱节点成为同步队列的首节点且释放同步状态后，会自动唤醒它。
        //其实笔者认为这里不加这个判断条件应该也是可以的。只是对于CAS修改前驱节点状态为SIGNAL成功这种情况来说，如果不加这个判断条件，提前唤醒了线程，等进入acquireQueued方法了节点发现自己的前驱不是首节点，还要再阻塞，等到其前驱节点成为首节点并释放锁时再唤醒一次；而如果加了这个条件，线程被唤醒的时候它的前驱节点肯定是首节点了，线程就有机会直接获取同步状态从而避免二次阻塞，节省了硬件资源。
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }

```
>signal就是唤醒Condition队列中的第一个非CANCELLED节点线程，而signalAll就是唤醒所有非CANCELLED节点线程，本质是将节点从Condition队列中取出来一个还是所有节点放到AQS的等待队列。尽管所有Node可能都被唤醒，但是要知道的是仍然只有一个线程能够拿到锁，其它没有拿到锁的线程仍然需要自旋等待，就上上面提到的第4步(acquireQueued)


>1，线程1调用reentrantLock.lock时，尝试获取锁。如果成功，则返回，从AQS的队列中移除线程；否则阻塞，保持在AQS的等待队列中。
>2，线程1调用await方法被调用时，对应操作是被加入到Condition的等待队列中，等待signal信号；同时释放锁。
>3，锁被释放后，会唤醒AQS队列中的头结点，所以线程2会获取到锁。
>4，线程2调用signal方法，这个时候Condition的等待队列中只有线程1一个节点，于是它被取出来，并被加入到AQS的等待队列中。注意，这个时候，线程1 并没有被唤醒，只是被加入AQS等待队列。
>5，signal方法执行完毕，线程2调用unLock()方法，释放锁。这个时候因为AQS中只有线程1，于是，线程1被唤醒，线程1恢复执行。所以：发送signal信号只是将Condition队列中的线程加到AQS的等待队列中。只有到发送signal信号的线程调用reentrantLock.unlock()释放锁后，这些线程才会被唤醒

