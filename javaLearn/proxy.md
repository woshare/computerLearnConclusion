# 代理

* [静态代理-动态代理（JDK proxy，CGLIB）](https://juejin.cn/post/6844904164787486727)


## JDK Proxy

```
public class MyHandler implements InvocationHandler{
    // 标识被代理类的实例对象
    private Object delegate;   
    // 构造器注入被代理对象
    public MyHandler(Object delegate){
       this.delegate = delegate;
    }
    
    // 重写invoke方法
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("被代理方法调用前的附加代码执行~ ");
        // 真实的被代理方法调用
        method.invoke(delegate, args);
        System.out.println("被代理方法调用后的附加代码执行~ ");
    } 
}

public class MainTest{
    public static void main(String[] args) {    
        // 创建被代理对象
        ImplA A = new ClassA();
        // 创建处理器类实现
        InvocationHandler myHandler = new MyHandler(A);
        // 重点！ 生成代理类， 其中proxyA就是A的代理类了
        ImplA proxyA = (ImplA)Proxy.newProxyInstance(A.getClass().getClassLoader(), A.getClass().getInterfaces(), myHandler);
        // 调用代理类的代理的methoda方法， 在此处就会去调用上述myHandler的invoke方法区执行，至于为什么，先留着疑问，下面会说清楚~
        proxyA.methoda();
    }
}

```

### 优点：
>1，动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。
>2，动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。
>3，接口增加一个方法，除了所有实现类需要实现这个方法外，动态代理类会直接自动生成对应的代理方法。


### 缺点：
>1，JDK proxy只能对有实现接口的类才能代理，也就是说没有接口实现的类，jdk proxy是无法代理的




