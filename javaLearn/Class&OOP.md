# 类与面向对象编程

## 类

### 抽象类
>1，抽象类中，至少有一个抽象方法（通过关键字 abstract 定义的方法，并且没有方法体  
>2，需要注意的是，抽象类是不能实例化的！ 它需要被一个子类继承   
>3，抽象类的子类必须给出父类中的抽象方法的具体实现，除非该子类也是抽象类    

```
abstract class CC {
	public abstract void MM();

	public  void XX(){

    }
}
```

### 接口:一个类可以实现多个接口，java类是单继承


```
/ 隐式的abstract
interface INF {
	//成员变量隐式为 static final
    // 隐式的public
	void PMM();
	void PNN();
}
```

#### 接口在应用中常见的三种模式：策略模式、适配器模式和工厂模式

### extends (延伸) & impliment （实现）
>1，Extends可以理解为全盘继承了父类的功能
>2，implements可以理解为为这个类附加一些额外的功能
>3，interface定义一些方法,并没有实现,需要implements来实现才可用
>4，extend可以继承一个接口,但仍是一个接口,也需要implements之后才可用
>5，对于class而言，Extends用于(单)继承一个类（class），而implements用于实现一个接口(interface)
>6，JAVA中不支持多重继承，继承只能继承一个类，但implements可以实现多个接口，比如 class A extends B implements C,D,E
>7，implements，实现父类，子类不可以覆盖父类的方法或者变量。即使子类定义与父类相同的变量或者函数，也会被父类取代掉   
>8，implements一般是实现接口。extends 是继承类