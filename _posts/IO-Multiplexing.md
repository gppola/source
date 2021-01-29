---
title: I/O 多路复用
date: 2021-01-29 20:02:23
tags: Linux
---

https://www.bilibili.com/video/BV11K4y1C7rm?p=2

![](/images/IO_multiplexing/intro.png)

```
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class TestSocket {
    public static void main(String[] args) throws Exception {
        ServerSocket server = new ServerSocket(8090);
        System.out.println("step1: new ServerSocket(8090)");
        while (true) {
            Socket client = server.accept();
            System.out.println("step2: client\t" + client.getPort());
            new Thread(() -> {
                try {
                    InputStream in = client.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                    while (true) {
                        System.out.println(reader.readLine());
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

-ff 对fork出来的线程/进程 也进行跟踪     系统调用
```
strace -ff -o ./ooxx java TestSocket
```

server | client
```
ck@test104:~/SocketTest$ strace -ff -o ./o│ck@test104:~/SocketTest$ nc localhost 8090
oxx java TestSocket                       │hello from client
step1: new ServerSocket(8090)             │
step2: client   45520                     │
hello from client                         │
```

```
ck@test104:~/SocketTest$ ls
ooxx.2494  ooxx.2496  ooxx.2498  ooxx.2500  ooxx.2502  ooxx.2505         TestSocket.java
ooxx.2495  ooxx.2497  ooxx.2499  ooxx.2501  ooxx.2503  TestSocket.class
ck@test104:~/SocketTest$ grep 'write' *
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(5, "\0", 1)                       = 1
ooxx.2495:write(1, "step1: new ServerSocket(8090)", 29) = 29
ooxx.2495:write(1, "\n", 1)                       = 1
ooxx.2495:write(1, "step2: client\t45520", 19)    = 19
ooxx.2495:write(1, "\n", 1)                       = 1
ooxx.2505:write(1, "hello from client", 17)       = 17
ooxx.2505:write(1, "\n", 1)                       = 1
ck@test104:~/SocketTest$ jps
2494 TestSocket
2527 Jps
ck@test104:~/SocketTest$ netstat -atp | grep 8090
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 localhost:45520         localhost:8090          ESTABLISHED 2504/nc
tcp6       0      0 [::]:8090               [::]:*                  LISTEN      2494/java
tcp6       0      0 127.0.0.1:8090          127.0.0.1:45520         ESTABLISHED 2494/java
```

socket shared in all threads
```
ck@test104:~/SocketTest$ ll /proc/2494/fd
total 0
dr-x------ 2 ck ck  0 Jan 29 12:27 ./
dr-xr-xr-x 9 ck ck  0 Jan 29 12:21 ../
lrwx------ 1 ck ck 64 Jan 29 12:27 0 -> /dev/pts/0
lrwx------ 1 ck ck 64 Jan 29 12:27 1 -> /dev/pts/0
lr-x------ 1 ck ck 64 Jan 29 12:27 10 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/icedtea-sound.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 11 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/cldrdata.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 12 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/dnsns.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 13 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/jaccess.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 14 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/zipfs.jar
lrwx------ 1 ck ck 64 Jan 29 12:27 15 -> 'socket:[33882]'
lrwx------ 1 ck ck 64 Jan 29 12:27 16 -> 'socket:[33884]'
lrwx------ 1 ck ck 64 Jan 29 12:27 17 -> 'socket:[33893]'
lrwx------ 1 ck ck 64 Jan 29 12:27 2 -> /dev/pts/0
lr-x------ 1 ck ck 64 Jan 29 12:27 3 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/rt.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 4 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/jfr.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 5 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/sunjce_provider.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 6 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/sunec.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 7 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/localedata.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 8 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/sunpkcs11.jar
lr-x------ 1 ck ck 64 Jan 29 12:27 9 -> /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext/nashorn.jar
ck@test104:~/SocketTest$ ps -aL | grep 2494
   2494    2494 pts/0    00:00:00 java
   2494    2495 pts/0    00:00:00 java
   2494    2496 pts/0    00:00:00 VM Thread
   2494    2497 pts/0    00:00:00 Reference Handl
   2494    2498 pts/0    00:00:00 Finalizer
   2494    2499 pts/0    00:00:00 Signal Dispatch
   2494    2500 pts/0    00:00:00 C2 CompilerThre
   2494    2501 pts/0    00:00:00 C1 CompilerThre
   2494    2502 pts/0    00:00:00 Service Thread
   2494    2503 pts/0    00:00:00 VM Periodic Tas
   2494    2505 pts/0    00:00:00 Thread-0
```

java 里面的东西就是对系统调用的包装
连接的socket: 17 -> 得到文件描述符
```
ck@test104:~/SocketTest$ grep 'accept' *
ooxx.2495:accept(16, {sa_family=AF_INET6, sin6_port=htons(45520), inet_pton(AF_INET6, "::ffff:127.0.0.1", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, [28]) = 17
Binary file TestSocket.class matches
TestSocket.java:      Socket client = server.accept();
```

man accept
```
RETURN VALUE
       On  success,  these  system  calls  return  a nonnegative integer that is a file descriptor for the accepted
       socket.  On error, -1 is returned, errno is set appropriately, and addrlen is left unchanged.
```

![](/images/IO_multiplexing/thread.png)

Java 1.2 之前创建的线程为用户级线程，程序员们为JVM开发了自己的一个线程调度内核，而到操作系统层面就是用户空间内的线程实现；1.2及以后，JVM选择了更加稳健且方便使用的操作系统原生的线程模型，通过系统调用，将程序的线程交给了操作系统内核进行调度。也就是说，现在的Java中线程的本质，其实就是操作系统中的线程，Linux下是基于pthread库实现的轻量级进程，Windows下原生的系统32API提供系统调用从而实现多线程。

```
ck@test104:~/SocketTest$ grep 'fork' *
ck@test104:~/SocketTest$ grep 'clone' *
ooxx.2494:clone(child_stack=0x7fbcaed3efb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcaed3f9d0, tls=0x7fbcaed3f700, child_tidptr=0x7fbcaed3f9d0) = 2495
ooxx.2495:clone(child_stack=0x7fbcadc20fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcadc219d0, tls=0x7fbcadc21700, child_tidptr=0x7fbcadc219d0) = 2496
ooxx.2495:clone(child_stack=0x7fbcadb1ffb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcadb209d0, tls=0x7fbcadb20700, child_tidptr=0x7fbcadb209d0) = 2497
ooxx.2495:clone(child_stack=0x7fbcada1efb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcada1f9d0, tls=0x7fbcada1f700, child_tidptr=0x7fbcada1f9d0) = 2498
ooxx.2495:clone(child_stack=0x7fbcad637fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad6389d0, tls=0x7fbcad638700, child_tidptr=0x7fbcad6389d0) = 2499
ooxx.2495:clone(child_stack=0x7fbcad536fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad5379d0, tls=0x7fbcad537700, child_tidptr=0x7fbcad5379d0) = 2500
ooxx.2495:clone(child_stack=0x7fbcad435fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad4369d0, tls=0x7fbcad436700, child_tidptr=0x7fbcad4369d0) = 2501
ooxx.2495:clone(child_stack=0x7fbcad334fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad3359d0, tls=0x7fbcad335700, child_tidptr=0x7fbcad3359d0) = 2502
ooxx.2495:clone(child_stack=0x7fbcad233fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad2349d0, tls=0x7fbcad234700, child_tidptr=0x7fbcad2349d0) = 2503
ooxx.2495:clone(child_stack=0x7fbcad080fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad0819d0, tls=0x7fbcad081700, child_tidptr=0x7fbcad0819d0) = 2505
```

主方法调用poll
```
ck@test104:~/SocketTest$ tail -f ooxx.2495
futex(0x7fbca80b1078, FUTEX_WAKE_PRIVATE, 1) = 1
futex(0x7fbca80b1028, FUTEX_WAKE_PRIVATE, 1) = 1
mprotect(0x7fbca8194000, 4096, PROT_READ|PROT_WRITE) = 0
mprotect(0x7fbca8195000, 4096, PROT_READ|PROT_WRITE) = 0
mmap(NULL, 1052672, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x7fbcacf81000
clone(child_stack=0x7fbcad080fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad0819d0, tls=0x7fbcad081700, child_tidptr=0x7fbcad0819d0) = 2505
futex(0x7fbca800b278, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAIT_PRIVATE, 2, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAKE_PRIVATE, 1) = 0
poll([{fd=16, events=POLLIN|POLLERR}], 1, -1
```

nc localhost 8090 新建一个tcp连接, socket file descriptor = 18, clone 一个新线程2706 等待输入
```
ck@test104:~/SocketTest$ tail -f ooxx.2495
futex(0x7fbca80b1078, FUTEX_WAKE_PRIVATE, 1) = 1
futex(0x7fbca80b1028, FUTEX_WAKE_PRIVATE, 1) = 1
mprotect(0x7fbca8194000, 4096, PROT_READ|PROT_WRITE) = 0
mprotect(0x7fbca8195000, 4096, PROT_READ|PROT_WRITE) = 0
mmap(NULL, 1052672, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x7fbcacf81000
clone(child_stack=0x7fbcad080fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcad0819d0, tls=0x7fbcad081700, child_tidptr=0x7fbcad0819d0) = 2505
futex(0x7fbca800b278, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAIT_PRIVATE, 2, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAKE_PRIVATE, 1) = 0
poll([{fd=16, events=POLLIN|POLLERR}], 1, -1) = 1 ([{fd=16, revents=POLLIN}])
accept(16, {sa_family=AF_INET6, sin6_port=htons(45522), inet_pton(AF_INET6, "::ffff:127.0.0.1", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, [28]) = 18
fcntl(18, F_GETFL)                      = 0x2 (flags O_RDWR)
fcntl(18, F_SETFL, O_RDWR)              = 0
write(1, "step2: client\t45522", 19)    = 19
write(1, "\n", 1)                       = 1
mmap(NULL, 1052672, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x7fbcace80000
clone(child_stack=0x7fbcacf7ffb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fbcacf809d0, tls=0x7fbcacf80700, child_tidptr=0x7fbcacf809d0) = 2706
futex(0x7fbca800b27c, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAIT_PRIVATE, 2, NULL) = 0
futex(0x7fbca800b228, FUTEX_WAKE_PRIVATE, 1) = 0
poll([{fd=16, events=POLLIN|POLLERR}], 1, -1
```

也阻塞住了，等待输入
```
ck@test104:~/SocketTest$ tail -f ooxx.2706
rt_sigprocmask(SIG_UNBLOCK, [HUP INT ILL BUS FPE SEGV USR2 TERM], NULL, 8) = 0
rt_sigprocmask(SIG_BLOCK, [QUIT], NULL, 8) = 0
futex(0x7fbca800b27c, FUTEX_WAKE_PRIVATE, 1) = 1
futex(0x7fbca800b228, FUTEX_WAKE_PRIVATE, 1) = 1
sched_getaffinity(2706, 32, [0])        = 8
sched_getaffinity(2706, 32, [0])        = 8
mmap(0x7fbcace80000, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbcace80000
mprotect(0x7fbcace80000, 12288, PROT_NONE) = 0
prctl(PR_SET_NAME, "Thread-1")          = 0
recvfrom(18,
```

```
ck@test104:~/test$ ps -aL | grep 2494
   2494    2494 pts/0    00:00:00 java
   2494    2495 pts/0    00:00:00 java
   2494    2496 pts/0    00:00:00 VM Thread
   2494    2497 pts/0    00:00:00 Reference Handl
   2494    2498 pts/0    00:00:00 Finalizer
   2494    2499 pts/0    00:00:00 Signal Dispatch
   2494    2500 pts/0    00:00:00 C2 CompilerThre
   2494    2501 pts/0    00:00:00 C1 CompilerThre
   2494    2502 pts/0    00:00:00 Service Thread
   2494    2503 pts/0    00:00:02 VM Periodic Tas
   2494    2505 pts/0    00:00:00 Thread-0
   2494    2706 pts/0    00:00:00 Thread-1
```

发送数据
```
ck@test104:~/test$ nc localhost 8090
second client msg
```

```
ck@test104:~/SocketTest$ tail -f ooxx.2706
rt_sigprocmask(SIG_UNBLOCK, [HUP INT ILL BUS FPE SEGV USR2 TERM], NULL, 8) = 0
rt_sigprocmask(SIG_BLOCK, [QUIT], NULL, 8) = 0
futex(0x7fbca800b27c, FUTEX_WAKE_PRIVATE, 1) = 1
futex(0x7fbca800b228, FUTEX_WAKE_PRIVATE, 1) = 1
sched_getaffinity(2706, 32, [0])        = 8
sched_getaffinity(2706, 32, [0])        = 8
mmap(0x7fbcace80000, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbcace80000
mprotect(0x7fbcace80000, 12288, PROT_NONE) = 0
prctl(PR_SET_NAME, "Thread-1")          = 0
recvfrom(18, "second client msg\n", 8192, 0, NULL, NULL) = 18
ioctl(18, FIONREAD, [0])                = 0
write(1, "second client msg", 17)       = 17
write(1, "\n", 1)                       = 1
recvfrom(18,
```


每个线程对应一个client
BIO: blcok IO,阻塞IO
因为阻塞才抛出这么多线程


## select

NIO 一个线程解决N个客户端的问题 
Java NIO: new IO
操作系统NIO，non-blocking IO

需要修改内核，增加新的系统调用 select
select: 多路复用器

```
select(fds) // O(1)
kernel // 主动遍历 O(n)
recvfrom // O(m)
```

弊端：1 每次重复传递数据 2 每次调用要触发内核遍历

## epoll
![](/images/IO_multiplexing/epoll.png)