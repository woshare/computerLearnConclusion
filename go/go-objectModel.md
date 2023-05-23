


* [对象模型](http://r12f.com/posts/learning-golang-object-model-struct/)


### nil Vs null

* [nil详细介绍](https://zhuanlan.zhihu.com/p/531802029)

>初学Go语言的时候常将nil看成是其它语言中的null或者NULL。 这种看法只是部分上正确的，但是Go中的nil和其它语言中的null或者NULL也是有很大的区别的    
>预声明标识符nil没有一个默认类型，尽管它有很多潜在的可能类型。 事实上，预声明标识符nil是Go中唯一一个没有默认类型的类型不确定值     
>两个不同类型的nil值可能不能相互比较:  var _ = (*int)(nil) == (*bool)(nil) // error: 类型不匹配
>不同种类的类型的nil值的尺寸很可能不相同
>nil被设计成一个可以表示成很多种类型的零值的预声明标识符。 换句话说，它可以表示很多内存布局不同的值，而不仅仅是一个值
>并不是所有的类型能够赋值 nil，并且和 nil 进行对比判断。只有 slice、map、channel、interface、指针、函数 这 6 种类型
>以上 6 种类型和 nil 进行比较判断本质上都是和变量本身做判断，slice 是判断管理结构的第一个指针字段，map，channel 本身就是指针，interface 也是判断管理结构的第一个指针字段，指针和函数变量本身就是指针   
```
var p *struct{} = nil
fmt.Println( unsafe.Sizeof( p ) ) // 8

var s []int = nil
fmt.Println( unsafe.Sizeof( s ) ) // 24
```

#### nil本质


>nil 本质上是一个 Type 类型的变量而已；
>Type 类型仅仅是基于 int 定义出来的一个新类型
>nil 其实更准确的理解是一个触发条件，编译器看到和 nil 值比较的写法，那么就要确认类型在这 6 种类型以内，如果是赋值 nil，那么也要确认在这 6 种类型以内，并且对应的结构内存为全 0 数据。 所以，记住这句话，nil 是编译器识别行为的一个触发点而已，看到这个 nil 会触发编译器的一些特殊判断和操作

```
// nil is a predeclared identifier representing the zero value for a
// pointer, channel, func, interface, map, or slice type.
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type

// Type is here for the purposes of documentation only. It is a stand-in
// for any Go type, but represents the same type for any given function
// invocation.
type Type int

```

#### nil赋值和判断
// slice 赋值 nil
slice2 = nil
发生什么事？

事情在编译期间就确定了，就是把 slice2 变量本身内存块置 0 ，也就是说 slice2 本身的 24 字节的内存块被置 0。

nil 值判断
编译器认为 slice 做可以做 nil 判断，那么什么样的 slice 认为是 nil 的？

指针值为 0 的，也就是说这个动态数组没有实际数据的时候

```
package main

import (
   "unsafe"
)

type sliceType struct {
   pdata unsafe.Pointer
   len   int
   cap   int
}

func main() {
   var a []byte

   ((*sliceType)(unsafe.Pointer(&a))).len = 0x3
   ((*sliceType)(unsafe.Pointer(&a))).cap = 0x4

   if a != nil {
      println("not nil")
   } else {
      println("nil")
   }
}
```
答案是：输出 nil。

#### var 和 make 这两种方式有什么区别？

>第一种 var 的方式定义变量纯粹真的是变量定义，如果逃逸分析之后，确认可以分配在栈上，那就在栈上分配这 24 个字节，如果逃逸到堆上去，那么调用 newobject 函数进行类型分配。
>第二种 make 方式则略有不同，如果逃逸分析之后，确认分配在栈上，那么也是直接在栈上分配 24 字节，如果逃逸到堆上则会导致调用 makeslice 函数来分配变量