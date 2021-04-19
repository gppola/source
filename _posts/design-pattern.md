---
title: 设计模式
date: 2020-10-10 10:29:35
tags: JVM
---

# GOF: group of four 四人帮, 国外大牛总结出23种套路。
- 创建型模式：单例模式，工厂模式，抽象工厂模式，建造者模式，原型模式
- 结构型模式：适配器模式，桥接模式，装饰模式，组合模式，外观模式，享元模式，代理模式
- 行为型模式：模板方法模式，命令模式，迭代器模式，观察者模式，中介者模式，备忘录模式，解释器模式，状态模式，策略模式，职责链模式，访问者模式

# 单例模式
```
/**
 * 饿汉单例模式
 */
public class Singleton {

    // 类初始化时，立即加载这个对象 (由于加载类时，天然的线程安全, 不需要同步)
    private static Singleton instance = new Singleton();

    private Singleton() {}

    // 方法没有同步，调用效率高
    public static Singleton getInstance() {
        return instance;
    }
}

/**
 * 懒汉单例模式
 */
class Singleton2 {

    private static Singleton2 instance;

    private Singleton2() {}

    public static synchronized Singleton2 getInstance() {
        if (instance == null) {
            instance = new Singleton2();
        }
        return instance;
    }
}

/**
 * 静态内部类,也是一种懒汉单例模式
 */
class Singleton3 {

    private static class SingletonClassInstance {
        private static final Singleton3 instance = new Singleton3();
    }

    private Singleton3() {}

    public static Singleton3 getInstance() {
        return SingletonClassInstance.instance;
    }
}

/**
 * 枚举本身就是单例模式, JVM保证(没有延时加载)
 */
enum Singleton4 {

    // 这个枚举元素本身就是单例对象
    INSTANCE;

    public void singletonOperation() {

    }

}

/**
 * new一个对象半初始化,加volatile防止指令重排序
 * 如果不加volatile, thread 2 判断 if (instance != null) 后, 会使用了半初始化的对象
 */
class DclSingleton {
    private static volatile DclSingleton instance;
    public static DclSingleton getInstance() {
        if (instance == null) {
            synchronized (DclSingleton .class) {
                if (instance == null) {
                    instance = new DclSingleton();
                }
            }
        }
        return instance;
    }

    // private constructor and other methods...
}
```

## 通过反射方式直接调用私有构造器

```
	Class<Singleton> clazz = (Class<Singleton>) Class.forName("Singleton");
	Constructor<Singleton> cons = clazz.getDeclaredConstructor(null);
	cons.setAccessible(true);
	Singleton s3 = cons.newInstance();
	Singleton s4 = cons.newInstance();
	System.out.println(s3);
	System.out.println(s4);
	// Singleton@610455d6
	// Singleton@511d50c0
```

## 防止利用反射

```
 private Singleton() { // 第二次调用就抛出异常
        if (instance != null) throw new RuntimeException();
    }
```
