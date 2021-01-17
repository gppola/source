---
title: 深入JVM
date: 2020-12-02 20:17:31
tags: JVM
---

## Learn about JVM internals

https://www.youtube.com/watch?v=UwB0OSmkOtQ
look inside an interpreter
```
while(true) {
	bytecode b = bytecodeStream[pc++]
	switch(b) {
		case iconst_1: push(1); break;
		case iload_0: push(local(0)); break;
		case iadd: push(pop() + pop()); break;
		...
	}
}
```

## JVM结构

![](/images/jvm_intro/jvm.png)

## GC
Division of Heap into Eden, Survivor and Tenured/Old spaces
- Minor GC: Collecting garbage from Young space (consisting of Eden and Survivor spaces) is called a Minor GC. 
- Major GC is cleaning the Tenured space.
- Full GC is cleaning the entire Heap – both Young and Tenured spaces.
![](/images/jvm_intro/gc.png)


## 可达性分析算法（根可达）

![](/images/jvm_intro/root.png)


python: 计数器的方式，没有办法对相互引用对象进行回收


### 案例

```
public class B {
	public static void main(String[] args) {
		A a1 = new A();
		A a2 = new A();
		A a3 = new A();
		a3 = a1;
		a1 = a2;
		a2 = null;
		a3 = a1;
	} // 当代码运行这的时候，有多少对象符合垃圾回收的条件
}

class A {
	long l = 1200L; // 不是基本数据类型--包装类型 类 字节码 long.valueOf(1200)
	// 如果 l = 12L, 正确答案是2个，因为包装类型有缓存
	public void a2() {
		System.out.println("hello");
	}
}
```

![](/images/jvm_intro/case1_ans.png)