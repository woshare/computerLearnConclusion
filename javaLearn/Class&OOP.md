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