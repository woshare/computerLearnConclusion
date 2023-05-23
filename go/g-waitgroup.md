


Go中的指针可以分为三类：1.普通类型指针*T，例如*int；2. unsafe.Pointer指针；3. uintptr。

*T：普通的指针类型，用于传递对象地址，不能进行指针计算。
unsafe.Pointer指针：通用型指针，任何一个普通类型的指针*T都可以转换为unsafe.Pointer指针，而且unsafe.Pointer类型的指针还可以转换回普通指针，并且它可以不用和原来的指针类型*T相同。但是它不能进行指针计算，不能读取内存中的值（必须通过转换为某一具体类型的普通指针才行）。
uintptr：准确来讲，uintptr并不是指针，它是一个大小并不明确的无符号整型。unsafe.Pointer类型可以与uinptr相互转换，由于uinptr类型保存了指针所指向地址的数值，因此可以通过该数值进行指针运算。GC时，不会将uintptr当做指针，uintptr类型目标会被回收



## WaitGroup
type WaitGroup struct {
	noCopy noCopy

	// 64-bit value: high 32 bits are counter, low 32 bits are waiter count.
	// 64-bit atomic operations require 64-bit alignment, but 32-bit
	// compilers only guarantee that 64-bit fields are 32-bit aligned.
	// For this reason on 32 bit architectures we need to check in state()
	// if state1 is aligned or not, and dynamically "swap" the field order if
	// needed.
	state1 uint64
	state2 uint32
}

// noCopy may be added to structs which must not be copied
// after the first use.
//
// See https://golang.org/issues/8005#issuecomment-190753527
// for details.
//
// Note that it must not be embedded, due to the Lock and Unlock methods.
type noCopy struct{}

### 《no copy机制》
noCopy字段是空结构体，它并不会占用内存，编译器也不会对其进行字节填充。它主要是为了通过go vet工具来做静态编译检查，防止开发者在使用WaitGroup过程中对其进行了复制，从而导致的安全隐患


![](./res/waitgroup.awebp "")







## 在使用WaitGroup时，有几点需要注意

通过Add()函数添加的counter数一定要与后续通过Done()减去的数值一致。如果前者大，那么阻塞在Wait()调用处的goroutine将永远得不到唤醒；如果后者大，将会引发panic。
Add()的增量函数应该最先得到执行。
不要对WaitGroup对象进行复制使用。
如果要复用WaitGroup，则必须在所有先前的Wait()调用返回之后再进行新的Add()调用
