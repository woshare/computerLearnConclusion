# CountDownLatch

```
public class CountDownLatchTest {
    public static void main(String[] args) throws Exception{

        /*创建CountDownLatch实例,计数器的值初始化为3*/
        final CountDownLatch downLatch = new CountDownLatch(3);

        /*创建三个线程,每个线程等待1s,表示执行比较耗时的任务*/
        for(int i = 0;i < 3;i++){
            final int num = i;
            new Thread(new Runnable() {
                public void run() {
                    try {
                        Thread.sleep(1000);

                    }catch (InterruptedException e){
                        e.printStackTrace();

                    }

                    System.out.println(String.format("thread %d has finished",num));

                    /*任务完成后调用CountDownLatch的countDown()方法*/
                    // **计数器N的值减1**
                    downLatch.countDown();
                }
            }).start();
        }

        /*主线程调用await()方法,等到其他三个线程执行完后才继续执行*/
        //**需要等待的线程会调用CountDownLatch的await方法，该线程就会阻塞直到计数器的值减为0**
        //主线程调用 CountDownLatch 的 await() 方法，所以会开始阻塞，直到 CountDownLatch 的 count 为 0 才继续执行
        downLatch.await();

        System.out.print("all threads have finished,main thread will continue run");
    }
}


```

```
public class CountDownLatchDemo {
    
    static class TaskThread extends Thread {
        
        CountDownLatch latch;
        
        public TaskThread(CountDownLatch latch) {
            this.latch = latch;
        }
        
        @Override
        public void run() {
            
            try {
                //等待latch=0
                latch.await();
                System.out.println(getName() + " start " + System.currentTimeMillis());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } 
        }
        
    }
    
    public static void main(String[] args) throws InterruptedException {
        
        int threadNum = 10;
        CountDownLatch latch = new CountDownLatch(1);
        
        for(int i = 0; i < threadNum; i++) {
            TaskThread task = new TaskThread(latch);
            task.start();
        }
        
        Thread.sleep(1000);
        //调用 CountDownLatch 的 countDown() 方法，调用后就会唤醒所有等待的线程，所有等待的线程就会同时执行
        latch.countDown();

    }

}
```



### 缺点：
>1，只能使用一次