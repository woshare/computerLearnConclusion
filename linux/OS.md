# 操作系统


![](./res/linux-5-module.png "内核五大模块")

![](./res/linux-module-framework.png "内核模块架构")

![](./res/os-mem.png "物理内存")

>Linux内存管理中，段变换：将一个由段选择符和段内偏移构成的逻辑地址转换为一个线性地址。页变换：将线性地址转换为对应的物理地址

![](./res/mem-segment-page.png "内存管理-段页式")

![](./res/mem-segment-page-convert.png "内存管理-段页式-虚拟地址到物理地址转换")

>1、 全局描述符表（Global descriptor table---GDT）
>2、 局部描述符表（Local descriptor table---LDT）

![](./res/mem-segment-page-GDT-LDT.png "内存管理-段页式-GDT-LDT")



![](./res/mem-page.png "内存分页管理")
>Offset = 2^12=4K， table =2^10, directory = 2^10，所以线性地址空间为2^10*2^10*4k=4G。

## 进程
>由于0.11内核把每个进程的最大可用的虚拟内存空间定义为64M，因此每个进程的逻辑地址可以用任务号*64M，就可以转换到线性空间的地址。

![](./res/process-mem-0.11.png "linux0.11版进程地址占用")

![](./res/process-status-convert.png "进程状态转换")

![](./res/process-diaodu.png "进程调度")


## linux 0.11源码目录结构
![](./res/linux-0.11-source-code-struct.png "")



![](./res/linux-os-fs-module.png "")


![](./res/source-code-boot.png  "")

![](./res/source-code-fs.png "")

![](./res/source-code-fs-framework.png "")

![](./res/source-code-include-head.h.png "")

![](./res/source-code-asm-sys.png "")

![](./res/source-code-kernel-init.png "")

![](./res/source-code-kernel-struct.png "")

![](./res/source-code-diaoyong-level-struct.png "")

![](./res/source-code-kernel-blk-dev.png "")

![](./res/source-code-kernel-chr-dev.png "")

![](./res/source-code-kernel-math.png "")

![](./res/source-code-kernel-lib.png "")

![](./res/source-code-kernel-mm.png "")

![](./res/source-code-kernel-tool.png "")

![](./res/source-code-kernel-makefile.png "")