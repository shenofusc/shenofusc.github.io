---
title: "单例模式"
date: 2017-08-19
tags: 设计模式
---

如何保证一个对象只有一个实例并且易于访问呢？定义一个全局变量可以确保对象随时都可以被访问，但是不能防止我们实例化多个对象。一个更好的解决方法是让类自身负责保存它的唯一实例。这个类可以保证没有其他实例被创建，并且它可以提供一个访问该实例的方法。单例模式主要有五种实现方式，分别是饿汉模式、懒汉模式、双检锁（DCL）、静态内部类和枚举，下面分别展示了这几种模式的Java写法。

<!-- more -->

## 定义
单例模式确保某一个类只有一个实例，自行实例化并向整个系统提供这个实例，由它来提供全局访问的方法。单例模式有三个要点：一个类只能有一个实例；它必须自行创建这个实例；它必须自行向整个系统提供这个实例。

## Java代码实现
### 饿汉模式
饿汉模式基于类装载机制，可避免多线程的同步问题。饿汉模式的实例是在类加载的过程中被创建出来的，因此可以保证只会存在一份对象实例。
```java
	public class Singleton {
	    private static Singleton instance = new Singleton();
	    private Singleton() {
	    }
	    public static Singleton getInstance() {
	        return instance;
	    }
	}
```

### 懒汉模式
懒汉模式最重要的是实现了Lazy Loading的效果，但是下面这种写法在多线程情况下不能正常工作。
```java
	public class LazySingleton {
	    private static LazySingleton instance;
	    private LazySingleton() {
	    }
	    public static LazySingleton getInstance() {
	        if (instance == null) {
	            instance = new LazySingleton();
	        }
	        return instance;
	    }
	}
```

### 双检锁
下面的示例引自[朱小厮的博客](http://blog.csdn.net/u013256816/article/details/50966882)，关于双检锁问题的根源在InfoQ的[双重检查锁定与延迟初始化](http://www.infoq.com/cn/articles/double-checked-locking-with-delay-initialization)一文中有详细的介绍。
```java
	public class DoubleCheckedLockingSingleton {  
	        // java中使用双重检查锁定机制,由于Java编译器和JIT的优化的原因系统无法保证我们期望的执行次序。  
	        // 在java5.0修改了内存模型,使用volatile声明的变量可以强制屏蔽编译器和JIT的优化工作  
	        private volatile static DoubleCheckedLockingSingleton uniqueInstance;  
	
	        private DoubleCheckedLockingSingleton() {  
	        }  
	
	        public static DoubleCheckedLockingSingleton getInstance() {  
	                if (uniqueInstance == null) {  
	                        synchronized (DoubleCheckedLockingSingleton.class) {  
	                                if (uniqueInstance == null) {  
	                                        uniqueInstance = new DoubleCheckedLockingSingleton();  
	                                }  
	                        }  
	                }  
	                return uniqueInstance;  
	        }  
	}
```

### 静态内部类
静态内部类同样基于类装载机制来避免多线程的同步问题。它跟饿汉模式的区别在于，饿汉模式是Singleton类只要被加载就会马上实例化INSTANCE，而使用这种方式时，只有显式调用getInstance()时INSTANCE才会被实例化，从而达到Lazy Loading的效果。
```java
	public class Singleton {
	    private Singleton() {
	    }
	    private static class SingletonHolder {
	        static final Singleton INSTANCE = new Singleton();
	    }
	    public static final Singleton getInstance() {
	        return SingletonHolder.INSTANCE;
	    }
	}
```

### 枚举
枚举不仅能避免多线程同步问题，而且也能有效防范反射和序列化攻击。另外，这种方式也可以实现多例，例如两例或三例。
```java
	public enum Singleton {
		INSTANCE;
	}
```

## 如何防止反射攻击和序列化攻击
下面是基于饿汉模式的一种单例实现，可以有效防范反射和序列化攻击，还能防范调用克隆方法产生新实例。
```java
public class Singleton implements Serializable {
    private static final Singleton singleton = new Singleton();

    // 用于防范反射攻击
    private Singleton() {
        if (Singleton.singleton != null) {
            throw new InstantiationError("Creating of this object is not allowed.");
        }
    }

    public static Singleton getInstance() {
        return singleton;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException("Cloning of this class is not allowed.");
    }

    // 用于防范序列化攻击
    protected Object readResolve() {
        return singleton;
    }
}
```

## 总结
主要特点：
1、单例模式最主要的特点有三个：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。
2、单例模式的主要优点在于提供了对唯一实例的受控访问并可以节约系统资源。
3、如果处理不当，系统中可能会因为反射攻击或其他原因（例如同时存在多个类加载器）而同时存在多个实例。