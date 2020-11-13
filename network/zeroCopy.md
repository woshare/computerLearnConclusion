# 零拷贝

## 发送数据时传统的拷贝实现方式是：
>1. `File.read(bytes)`
>2. `Socket.send(bytes)`

### 这种方式需要四次数据拷贝和四次上下文切换
>1. 数据从磁盘读取到内核的read buffer
>2. 数据从内核缓冲区拷贝到用户缓冲区
>3. 数据从用户缓冲区拷贝到内核的socket buffer
>4. 数据从内核的socket buffer拷贝到网卡接口（硬件）的缓冲区

## 零拷贝
>明显上面的第二步和第三步是没有必要的，通过java的FileChannel.transferTo方法，可以避免上面两次多余的拷贝（当然这需要底层操作系统支持）    

>1，调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer
>2，接着DMA从内核read buffer将数据拷贝到网卡接口buffer上面的两次操作都不需要CPU参与，所以就达到了零拷贝。


* [netty，reactor，network](https://juejin.im/post/6844903703183360008)
