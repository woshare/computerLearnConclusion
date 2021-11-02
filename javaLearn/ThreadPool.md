# ThreadPoolExecutor线程池源码解析

* [线程池原理-美团](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

## 四种常用线程池



## 线程池基本调度逻辑：核心线程，非核心线程，任务队列

## 线程池五种状态


## 线程池源码解析

### 线程池对象的重要参数

#### Worker解析：AQS，不可重入公平锁，反应线程现在的执行状态 

### 线程池submit和execute执行逻辑
>1，和FutureTask进行了关联：AbstractExecutorService.submit{execute(futureTask)}->ThreadPoolExecutor.execute->thread.start->worker.run->runWork->task.run->futureTask.run->futureTask.call->selfDefineTask.run
>2，直接execute：ThreadPoolExecutor.execute->thread.start->worker.run->runWork->task.run->>selfDefineTask.run



### 线程池回收