## 常用数据结构


### 
>1,LinkedList:经典的双链表结构, 适用于乱序插入, 删除. 指定序列操作则性能不如ArrayList   
>2,ArrayList:底层就是一个数组, 因此按序查找快, 乱序插入, 删除因为涉及到后面元素移位所以性能慢.    
>3,Stack:经典的数据结构, 底层也是数组, 继承自Vector, 先进后出FILO, 默认new Stack()容量为10, 超出自动扩容
>4,ArrayBlockingQueue:生产消费者中常用的阻塞有界队列, FIFO.
>5,HashMap: 内部通过数组 + 单链表+红黑树     
>6,LinkedHashMap:HashMap子类，再用一个双向链表维持数据有序   


![Alt text](./res/commonly-struct.png "常用数据结构")

![Alt text](./res/LinkedHashMap.gif "LinkedHashMap")

* [常用数据结构动图](https://blog.csdn.net/zzy7075/article/details/81099463)