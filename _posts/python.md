---
title: Python
date: 2021-06-13 11:12:50
tags: Linux
---

# draft

python 每个线程都是python interpreter程序，
独立的task_struct(see linux-thread post)
task_struct共享mm (point to mm_struct)
共享如下maps，有着相同的结构


## 全局解释器锁(GIL)

　无论你启多少个线程，你有多少个cpu, Python在执行的时候会淡定的在同一时刻只允许一个线程运行
Python的线程就是操作系统的线程(POSIX thread || Win thread)，线程调度也直接使用操作系统的线程调度。

因为python的线程是调用操作系统的原生线程，这个原生线程就是C语言写的原生线程。因为python是用C写的，启动的时候就是调用的C语言的接口。
每个线程在执行的过程中，python解释器是控制不了的，因为是调的C语言的接口，超出了python的控制范围，python的控制范围是只在python解释器这一层，所以python控制不了C接口，它只能等结果。所以它不能控制让哪个线程先执行，因为是一块调用的，只要一执行，就是等结果，这个时候4个线程独自执行，所以结果就不一定正确了。有了GIL，就可以在同一时间只有一个线程能够工作。虽然这4个线程都启动了，但是同一时间我只能让一个线程拿到这个数据。

这也是Cpython的一个缺陷，其他语言没有，仅仅只是Cpython有。


因为你python调用的所有线程都是原生线程。原生线程是通过C语言提供原生接口，相当于C语言的一个函数。你一调它，你就控制不了了它了，就必须等它给你返回结果。只要已通过python虚拟机，再往下就不受python控制了，就是C语言自己控制了。你加在python虚拟机以下，你是加不上去的。同一时间，只有一个线程穿过这个锁去真正执行。其他的线程，只能在python虚拟机这边等待。

需要明确的一点是GIL并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。就好比C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如GCC，INTEL C++，Visual C++等。Python也一样，同样一段代码可以通过CPython，PyPy，Psyco等不同的Python执行环境来执行。像其中的JPython就没有GIL。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把GIL归结为Python语言的缺陷。所以这里要先明确一点：GIL并不是Python的特性，Python完全可以不依赖于GIL。

···

```
// * 此处for循环是在执行OPCode序列
    for (;;) {

        // ...(只显示关键代码，其他忽略)

        // * 防止每个OP（非fast OP）都处理一次GIL，此处会限制OP次数。默认是100次。可修改。
        if (--_Py_Ticker < 0) {
            _Py_Ticker = _Py_CheckInterval;
            
            // ...
 
#ifdef WITH_THREAD
            // * 超过100次，则进入GIL处理部分
            if (interpreter_lock) {
                /* Give another thread a chance */

                // ...

                // * 如果GIL已经初始化，则释放它
                PyThread_release_lock(interpreter_lock);

                /* Other threads may run now */
                // *  再获取它 （此处会存在多线程竞争，只有一个线程能获取成功。其他线程只能在此处等待）
                PyThread_acquire_lock(interpreter_lock, 1);

                // ...
            }
#endif
        }
        

```
``` cat /proc/{pid}/maps
5649e51a6000-5649e51a7000 r--p 00000000 fe:01 1458454                    /usr/local/bin/python3.9
5649e51a7000-5649e51a8000 r-xp 00001000 fe:01 1458454                    /usr/local/bin/python3.9
5649e51a8000-5649e51a9000 r--p 00002000 fe:01 1458454                    /usr/local/bin/python3.9
5649e51a9000-5649e51aa000 r--p 00002000 fe:01 1458454                    /usr/local/bin/python3.9
5649e51aa000-5649e51ab000 rw-p 00003000 fe:01 1458454                    /usr/local/bin/python3.9
5649e522c000-5649e52b8000 rw-p 00000000 00:00 0                          [heap]
7f9d78000000-7f9d78021000 rw-p 00000000 00:00 0 
7f9d78021000-7f9d7c000000 ---p 00000000 00:00 0 
7f9d80000000-7f9d80021000 rw-p 00000000 00:00 0 
7f9d80021000-7f9d84000000 ---p 00000000 00:00 0 
7f9d8516f000-7f9d85170000 ---p 00000000 00:00 0 
7f9d85170000-7f9d85970000 rw-p 00000000 00:00 0 
7f9d85970000-7f9d85971000 ---p 00000000 00:00 0 
7f9d85971000-7f9d86316000 rw-p 00000000 00:00 0 
7f9d86316000-7f9d86348000 r--p 00000000 fe:01 1190428                    /usr/lib/locale/C.UTF-8/LC_CTYPE
7f9d86348000-7f9d8634b000 rw-p 00000000 00:00 0 
7f9d8634b000-7f9d8636d000 r--p 00000000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d8636d000-7f9d864b5000 r-xp 00022000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d864b5000-7f9d86501000 r--p 0016a000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d86501000-7f9d86502000 ---p 001b6000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d86502000-7f9d86506000 r--p 001b6000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d86506000-7f9d86508000 rw-p 001ba000 fe:01 1058295                    /lib/x86_64-linux-gnu/libc-2.28.so
7f9d86508000-7f9d8650e000 rw-p 00000000 00:00 0 
7f9d8650e000-7f9d8651b000 r--p 00000000 fe:01 1058320                    /lib/x86_64-linux-gnu/libm-2.28.so
7f9d8651b000-7f9d865ba000 r-xp 0000d000 fe:01 1058320                    /lib/x86_64-linux-gnu/libm-2.28.so
7f9d865ba000-7f9d8668f000 r--p 000ac000 fe:01 1058320                    /lib/x86_64-linux-gnu/libm-2.28.so
7f9d8668f000-7f9d86690000 r--p 00180000 fe:01 1058320                    /lib/x86_64-linux-gnu/libm-2.28.so
7f9d86690000-7f9d86691000 rw-p 00181000 fe:01 1058320                    /lib/x86_64-linux-gnu/libm-2.28.so
7f9d86691000-7f9d86692000 r--p 00000000 fe:01 1058374                    /lib/x86_64-linux-gnu/libutil-2.28.so
7f9d86692000-7f9d86693000 r-xp 00001000 fe:01 1058374                    /lib/x86_64-linux-gnu/libutil-2.28.so
7f9d86693000-7f9d86694000 r--p 00002000 fe:01 1058374                    /lib/x86_64-linux-gnu/libutil-2.28.so
7f9d86694000-7f9d86695000 r--p 00002000 fe:01 1058374                    /lib/x86_64-linux-gnu/libutil-2.28.so
7f9d86695000-7f9d86696000 rw-p 00003000 fe:01 1058374                    /lib/x86_64-linux-gnu/libutil-2.28.so
7f9d86696000-7f9d86697000 r--p 00000000 fe:01 1058305                    /lib/x86_64-linux-gnu/libdl-2.28.so
7f9d86697000-7f9d86698000 r-xp 00001000 fe:01 1058305                    /lib/x86_64-linux-gnu/libdl-2.28.so
7f9d86698000-7f9d86699000 r--p 00002000 fe:01 1058305                    /lib/x86_64-linux-gnu/libdl-2.28.so
7f9d86699000-7f9d8669a000 r--p 00002000 fe:01 1058305                    /lib/x86_64-linux-gnu/libdl-2.28.so
7f9d8669a000-7f9d8669b000 rw-p 00003000 fe:01 1058305                    /lib/x86_64-linux-gnu/libdl-2.28.so
7f9d8669b000-7f9d866a1000 r--p 00000000 fe:01 1058354                    /lib/x86_64-linux-gnu/libpthread-2.28.so
7f9d866a1000-7f9d866b0000 r-xp 00006000 fe:01 1058354                    /lib/x86_64-linux-gnu/libpthread-2.28.so
7f9d866b0000-7f9d866b6000 r--p 00015000 fe:01 1058354                    /lib/x86_64-linux-gnu/libpthread-2.28.so
7f9d866b6000-7f9d866b7000 r--p 0001a000 fe:01 1058354                    /lib/x86_64-linux-gnu/libpthread-2.28.so
7f9d866b7000-7f9d866b8000 rw-p 0001b000 fe:01 1058354                    /lib/x86_64-linux-gnu/libpthread-2.28.so
7f9d866b8000-7f9d866bc000 rw-p 00000000 00:00 0 
7f9d866bc000-7f9d866bd000 r--p 00000000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866bd000-7f9d866c3000 r-xp 00001000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866c3000-7f9d866c5000 r--p 00007000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866c5000-7f9d866c6000 ---p 00009000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866c6000-7f9d866c7000 r--p 00009000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866c7000-7f9d866c8000 rw-p 0000a000 fe:01 1058303                    /lib/x86_64-linux-gnu/libcrypt-2.28.so
7f9d866c8000-7f9d866f6000 rw-p 00000000 00:00 0 
7f9d866f6000-7f9d86756000 r--p 00000000 fe:01 1458618                    /usr/local/lib/libpython3.9.so.1.0
7f9d86756000-7f9d86967000 r-xp 00060000 fe:01 1458618                    /usr/local/lib/libpython3.9.so.1.0
7f9d86967000-7f9d86a5a000 r--p 00271000 fe:01 1458618                    /usr/local/lib/libpython3.9.so.1.0
7f9d86a5a000-7f9d86a60000 r--p 00363000 fe:01 1458618                    /usr/local/lib/libpython3.9.so.1.0
7f9d86a60000-7f9d86a9a000 rw-p 00369000 fe:01 1458618                    /usr/local/lib/libpython3.9.so.1.0
7f9d86a9a000-7f9d86abe000 rw-p 00000000 00:00 0 
7f9d86ac0000-7f9d86ac7000 r--s 00000000 fe:01 1190707                    /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
7f9d86ac7000-7f9d86ac8000 r--p 00000000 fe:01 1058281                    /lib/x86_64-linux-gnu/ld-2.28.so
7f9d86ac8000-7f9d86ae6000 r-xp 00001000 fe:01 1058281                    /lib/x86_64-linux-gnu/ld-2.28.so
7f9d86ae6000-7f9d86aee000 r--p 0001f000 fe:01 1058281                    /lib/x86_64-linux-gnu/ld-2.28.so
7f9d86aee000-7f9d86aef000 r--p 00026000 fe:01 1058281                    /lib/x86_64-linux-gnu/ld-2.28.so
7f9d86aef000-7f9d86af0000 rw-p 00027000 fe:01 1058281                    /lib/x86_64-linux-gnu/ld-2.28.so
7f9d86af0000-7f9d86af1000 rw-p 00000000 00:00 0 
7ffe3979f000-7ffe397c0000 rw-p 00000000 00:00 0                          [stack]
7ffe397c6000-7ffe397ca000 r--p 00000000 00:00 0                          [vvar]
7ffe397ca000-7ffe397cc000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
```




