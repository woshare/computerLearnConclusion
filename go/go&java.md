# go VS java 优劣


## 对比

  语言    |            优势             | 劣势                      
:--------------:|:-----------------------------|:-----------
 go           | 1，编译速度快于java，启动速度快于java  <br> 2，支持指针，更为灵活 <br> 3，支持函数式、也可实现面向对象编程。  <br> 4， 语言简洁些。没有烦人的大括号，那么多的class，使用首字母大小写来鉴别可见性，类型推断，struct相比java bean简洁等 <br> 5，并发更好。java的并发基于内存共享，通过Callable和Future或者netty的Future/promise模式返回结果，基于内存共享原理简单，并发场景复杂 时处理起来麻烦。go语言CSP模型,channel通知方式实现并发线程间的数据交互，使用更轻量级的协程，并发更有优势；java也有quasar这样的纤程 库、akka这样的actor库，但都不是官方支持，使用也不是很广泛。<br> 6，依赖可直接从github这些git库获取，不必非得搭建私服                   | 1，非常规的异常处理机制，大量的if err != nil <br> 2，生态不如java成熟。例如，go的ORM大都不够成熟，也很少对oracle支持。java的目前较为标准web开发是spring boot，go的web框架虽多，成熟性不好说，提供的功能也要比spring boot少 <br> 3，go自带的库已经很强大，依旧不如java，比如自带的map也不是线程安全的 <br>4，为了编译速度牺牲了很多现代语言普遍拥有的特性。不支持重载，不支持泛型，不支持方法默认参数等，也没有java的注解
 java          | 1，生态成熟 <br> 2，                     |  1，占用内存大 <br>2，启动速度慢     
 
## 总结
 >1，参考techempower的微基准测试, java、go性能相近，都不错, 尤其go的fasthttp，java的NIO框架vertx、netty、undertow。java使用最广泛spring，因为集成太多,比较臃肿，性能则比较一般。   
 >2，二者都是工程化不错的语言，语言可读性都不错，但go本身简洁些，也有go fmt这些工程化工具提高了可读性。   
 >3，JVM生态强大，成熟，稳定。还有kotlin、scala、groovy、clojure等语言可以替代使用，这点go还比不了。   
 >4，java受益于JVM，也局限于JVM，偏向调用底层的一些场景，go要比JAVA更合适，调用c库也更方便。比如docker，从这方便来讲，c/c++/rust这样的语言才是go的"竞争对手"。   