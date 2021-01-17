---
title: Volatile
date: 2020-06-15 16:09:52
tags: JVM
---

# Java

**Java Memory Model**

>The Java Memory Model was the first attempt to provide a comprehensive threading memory
 model for a popular programming language.

**CPU写内存的机制**

- 同步写
    - 1. 将写内存的请求写入Store Buffer
    - 2. CPU马上将Store Buffer的写内存的请求执行，将数据刷回内存
- 异步写 (主流CPU都是这种机制)
    - 1. 将写内存的请求写入Store Buffer
    - 2. CPU空闲后将Store Buffer的写内存请求执行，将数据刷回内存，内存屏障让异步写达到了同步写的效果

step1: y:524 in L1 cache
step2: [r1=y load] y:524 in Load Buffer/168 in Store Buffer
step3: [x=168 store] 168 in C-0 L1/C-5 L1/Memory
![](/images/volatile/outoforder.png)

>通过volatile标记，可以解决编译器层面的可见性与重排序问题。而内存屏障则解决了硬件层面的可见性与重排序问题

1. 为什么有了MESI的缓存一致性协议，还需要加volatile关键字呢，是不是多此一举？
答： MESI协议为了提高性能，引入了Store Buffe和Invalidate Queues，还是有可能会引起缓存不一致，所以需要再引入内存屏障来确保一致性
2. 内存屏障怎么实现的呢？
   答：内存屏障是硬件层的. x86架构的处理器中允许处理器对Store-Load操作进行重排,与之对应有StoreLoad Barriers禁止其重排. StoreLoad Barriers同时具备其他三个屏障的效果,因此也称之为全能屏障,是目前大多数处理器所支持的,但是相对其他屏障,该屏障的开销相对昂贵.
   在x86架构的处理器的指令集中,lock指令可以触发StoreLoad Barriers.
  
   通过 #lock前缀实现 （导致其它的CPU也触发一定的动作来同步自己的Cache。#lock引脚链接到北桥芯片(North Bridge)的#lock引脚，当带lock前缀的执行执行时，北桥芯片会拉起#lock电平，从而锁住总线，直到该指令执行完毕再放开。   而总线加锁会自动invalidate所有CPU对 _该指令涉及的内存的Cache）
   LOCK用于在多处理器中执行指令时对共享内存的独占使用
   它的作用是能够将当前处理器对应缓存的内容刷新到内存，并使其他处理器对应的缓存失效
   另外还提供了有序的指令无法越过这个内存屏障的作用
   ` 
	The LOCK prefix ensures that the CPU has exclusive ownership of the appropriate cache line 
	for the duration of the operation, and provides certain additional ordering guarantees.              
	This may be achieved by asserting a bus lock, but the CPU will avoid this where possible. 
	If the bus is locked then it is only for the duration of the locked instruction.
`
3. Java volatile关键字的作用：
答：线程可见性，禁止指令重排序，不保证原子性。转换成指令以后，多了一个`lock addl $0x0, (%esp)`空操作，相当于一个内存屏障 

- 1） 可见性：lock前缀会强制将本处理器的写缓冲区/高速缓冲中的脏数据写入内存，该写入动作也会引起别的处理器无效化(invalidate)其缓存。
- 2） 禁止重排序： CPU流水线可能会对指令重排序。当`lock addl $0x0, (%esp)` 把修改同步到内存时，意味着所有之前的操作都已经完成，这样便形成了“指令重排序无法越过内存屏障”的效果

# as if serial
不管如何重排序，单线程执行结果不会变

# DCL (Double Check Lock) 单例 需要volatile

```
public class DclSingleton {
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

new一个对象半初始化,加volatile防止指令重排序

thread 1
```
NEW java/lang/String # 半初始化的对象指针入栈
DUP # 复制一个栈顶元素
INVOKESPECIAL java/lang/String.<init> ()V # 构造初始化，消耗掉一个栈顶元素
ASTORE 1 # 将栈顶元素赋值给对象
```
指令重排后
```
NEW java/lang/String # 半初始化的对象指针入栈
DUP # 复制一个栈顶元素
ASTORE 1 # 将栈顶元素赋值给对象t
INVOKESPECIAL java/lang/String.<init> ()V # 构造初始化，消耗掉一个栈顶元素
```

thread 2
    if (t != null) 使用了半初始化的对象


# CAS

确保了对同一个同一个内存地址操作的原子性, 在x86架构上，CAS被翻译为”lock cmpxchg...“，当两个core同时执行针对同一地址的CAS指令时,其实他们是在试图修改每个core自己持有的Cache line. 向ring bus发出invalidate操作，ring bus仲裁后core0和core1，胜者完成结果，输者接受结果。

Java Object Layout:
- markword : 8 bytes
- class pointer: 4 bytes
- 实例数据 + padding

锁的信息记录在markdword上，一开始默认偏向锁ID101

- 无锁态 -> 偏向锁 ， 第一个运行这个object的线程贴上标签
- 偏向锁 -> 自旋锁： 发生任意冲突，一个人已经占据这个坑了，另一个人要抢. 撤销偏向锁，在每个线程的线程栈里生成一个自己的对象，叫 Lock Record 锁记录， 尝试往obj贴上去，使用自旋锁（轻量级锁）谁贴成功了就是谁的，由偏向锁升级为自旋锁 （CAS compare and swap）
- 竞争激烈，自旋超过一定次数升级为重量级锁，在内核生成，有一定的资源限制，不消耗cpu，操作系统决定什么时候给你解冻 （为什么升级，比如1000个线程，有一个线程卡着不出来）

![](/images/volatile/lock.png)

# C++

C++ volatile 只意味着不做优化，禁止指令重排序 和 每次都从内存获取值, 语言级别的 memory barrier. `NOT suitable for concurrent programming`

- 1 volatile关键字可以禁止指令优化，其实这里发挥了编译器屏障的作用

```
int main() {
        int foo = 11;
        int a = 1;
        a = 2;
        a = foo + 10;
        int b = a + 20;
        return b;
}

g++ -O1 test.cpp -o test
objdump -d -M intel test | grep "<main>:" -A 30
```
![](/images/volatile/cpp1.png)

```
int main() {
        volatile int foo = 11;
        volatile int a = 1;
        a = 2;
        a = foo + 10;
        int b = a + 20;
        return b;
}
```
![](/images/volatile/cpp2.png)


- 2 对声明为volatile的变量操作时，每次都会从内存中取值，而不会使用原先保存在寄存器中的值。本测试开启了一级优化，即 -O1
```
int func(int a) {
    int x;
    scanf("%d",&x);
    if (a > x) return a;
    else return x;
}
int main() {
    volatile int a = 5;
    int b = 10;
    int c = 20;
    scanf("%d",&c);
    a = func(c);
    b = func(a);

    printf("%d\n", b);
    return 0;
}    

g++ -O1 test.cpp -o test
objdump -d -M intel test | grep "<main>:" -A 30

无 volatile
8d9:   e8 7c ff ff ff          call   85a <_Z4funci> 
 8de:   89 c7                   mov    edi,eax // a = func(c)
 8e0:   e8 75 ff ff ff          call   85a <_Z4funci> // b = func(a)


有volatile
 8e0:   e8 75 ff ff ff          call   85a <_Z4funci> // a = func(c)
 8e5:   89 04 24                mov    DWORD PTR [rsp],eax // 返回值a写回内存
 8e8:   8b 3c 24                mov    edi,DWORD PTR [rsp] // 从内存取出a
 8eb:   e8 6a ff ff ff          call   85a <_Z4funci> // b = func(a)
```

- 3 顺序性 
如果两个都声明为volatile，那么编译器就不会对指令进行重排
```
volatile int a,b;
int fun() {
        a = b + 1;
        b = 0;
        return b;
}

g++ -O2 test.cpp -o test
objdump -d -M intel test | grep 'funv>:' -A 10

无volatile
0000000000000780 <_Z3funv>:
 780:   8b 05 8e 08 20 00       mov    eax,DWORD PTR [rip+0x20088e]        # 201014 <b>
 786:   c7 05 84 08 20 00 00    mov    DWORD PTR [rip+0x200884],0x0        # 201014 <b> // b = 0
 78d:   00 00 00
 790:   83 c0 01                add    eax,0x1
 793:   89 05 7f 08 20 00       mov    DWORD PTR [rip+0x20087f],eax        # 201018 <a> // a = b + 1

有volatile
00000000000007a0 <_Z3funv>:
 7a0:   8b 05 6e 08 20 00       mov    eax,DWORD PTR [rip+0x20086e]        # 201014 <b>
 7a6:   83 c0 01                add    eax,0x1
 7a9:   89 05 69 08 20 00       mov    DWORD PTR [rip+0x200869],eax        # 201018 <a> // a = b + 1
 7af:   c7 05 5b 08 20 00 00    mov    DWORD PTR [rip+0x20085b],0x0        # 201014 <b> // b = 0
 7b6:   00 00 00
 7b9:   8b 05 55 08 20 00       mov    eax,DWORD PTR [rip+0x200855]        # 201014 <b>
 ```







