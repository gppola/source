---
title: Linux thread
date: 2020-06-15 12:47:50
tags: Linux
---

# task_struct / mm_struct

一个进程的虚拟地址空间主要由两个数据结构来描述。一个是最高层次的：mm_struct（定义在mm_types.h中），一个是较高层次的：vm_area_structs。最高层次的mm_struct结构描述了一个进程的整个虚拟地址空间。较高层次的结构vm_area_truct描述了虚拟地址空间的一个区间（简称虚拟区或线性区）。每个进程只有一个mm_struct结构，在每个进程的task_struct结构中，有一个指向该进程的结构。可以说，mm_struct结构是对整个用户空间（注意，是用户空间）的描述。

![](/images/linux_thread/pcb.png)

![](/images/linux_thread/mm.png)


# do_fork

	do_fork -> copy_process -> copy_mm -> dup_mmap

copy_process函数中首先调用dup_task_struct()函数为子进程创建task_struct结构体等信息，然后根据clone_flags集合中的标志值，设置共享或者复制父进程打开的文件、文件系统信息、信号处理函数、进程地址空间、命名空间等资源，其中copy_mm函数实现父进程地址空间的拷贝，也就是fork创建子进程时的写时复制机制的核心处了，接下来看看这个函数的实现。

copy_mm()中检查clone_flags中是否有CLONE_VM标志，
- 若有，两个进程之间共享VM，即就是创建轻量级进程(线程)，

```	
/**
 * 当创建一个新的进程时，内核调用copy_mm函数，
 * 这个函数通过建立新进程的所有页表和内存描述符来创建进程的地址空间。
 * 通常，每个进程都有自己的地址空间，但是轻量级进程共享同一地址空间，即允许它们对同一组页进行寻址。
 */
static int copy_mm(unsigned long clone_flags, struct task_struct * tsk) 
{
	...
	/**
	 * 指定了CLONE_VM标志，表示创建线程。
	 */
	if (clone_flags & CLONE_VM) {
		/**
		 * 新线程共享父进程的地址空间，所以需要将mm_users加一。
		 */
		atomic_inc(&oldmm->mm_users);
		mm = oldmm;
		...
	}
	/**
	 * 没有CLONE_VM标志，就必须创建一个新的地址空间。
	 * 必须要有地址空间，即使此时并没有分配内存。
	 */
	retval = -ENOMEM;
	/**
	 * 分配一个新的内存描述符。把它的地址存放在新进程的mm中。
	 */
	mm = allocate_mm();
	...
	/* Copy the current MM stuff.. */
	/**
	 * 并从当前进程复制mm的内容。
	 */
	memcpy(mm, oldmm, sizeof(*mm));
	...
	/**
	 * dup_mmap不但复制了线性区和页表，也设置了mm的一些属性.
	 * 它也会改变父进程的私有，可写的页为只读的，以使写时复制机制生效。
	 */
	retval = dup_mmap(mm, oldmm);
	...
}
```

```
/**
 * 既复制父进程的线性区，也复制它的页表。
 */
static inline int dup_mmap(struct mm_struct * mm, struct mm_struct * oldmm) {
	...

	/**
	 * 复制父进程的每一个vm_area_struct线性区描述符，并把复制品插入到子进程的线性区链表和红黑树中。
	 */
	for (mpnt = current->mm->mmap ; mpnt ; mpnt = mpnt->vm_next) {
		...
		/**
		 * copy_page_range创建必要的页表来映射线性区所包含的一组页。并且初始化新页表的表项。
		 * 对私有、可写的页（无VM_SHARED标志，有VM_MAYWRITE标志），对父子进程都标记为只读的。
		 * 为写时复制进行处理。
		 */
		retval = copy_page_range(mm, current->mm, tmp);
		...
	}
	...
}
```

- 否则，就是fork创建进程，从而调用dup_mm()函数为子进程分配一个新的mm_struct结构体。

	- 使用dup_mmap()函数为子进程拷贝父进程地址空间，其中调用copy_page_range()函数进行页表的拷贝，由于linux中采用四级分页机制，分别是pgd、pud、pmd、pte，因而依次对其进行拷贝，最终在拷贝pte的函数copy_pte_range中调用copy_one_page函数实现真正的写时复制。

在该函数中判断页是否支持写时复制，若支持就给其添加写保护，在写操作发生时，发生写保护错误，从而为子进程新分配一块内存。

![](/images/linux_thread/4page.png)
```
static inline void
copy_one_pte(struct mm_struct *dst_mm,  struct mm_struct *src_mm,
		pte_t *dst_pte, pte_t *src_pte, unsigned long vm_flags,
		unsigned long addr)
{
	...
	/*
	 * If it's a COW mapping, write protect it both
	 * in the parent and the child
	 */
	if ((vm_flags & (VM_SHARED | VM_MAYWRITE)) == VM_MAYWRITE) {
		ptep_set_wrprotect(src_pte);
		pte = *src_pte;
	}

	/*
	 * If it's a shared mapping, mark it clean in
	 * the child
	 */
	if (vm_flags & VM_SHARED)
		pte = pte_mkclean(pte);
	...
}
```


# "one-to-one" model

LinuxThreads follows the so-called "one-to-one" model: each thread is actually a separate process in the kernel. The kernel scheduler takes care of scheduling the threads, just like it schedules regular processes. The threads are created with the Linux clone() system call, which is a generalization of fork() allowing the new process to share the memory space, file descriptors, and signal handlers of the parent.

内核线程的最大优势就是能够充分利用多处理器

# 进程绑定CPU

cpu 虚拟化，通过虚拟化，intel超线程技术，虚拟化出更多的cpu，vcpu与物理cpu不是一一对应的关系
两层：
* 针对硬件芯片的虚拟化，单核4个超线程，一个核可以虚拟4个cpu出来
* 再往上走一层，虚拟cpu在vmware上也可以进程虚拟，纯软件的虚拟化，这个与cpu没有太大的关系，虚拟出来的vmware的多个cpu实际上是多个进程，并不是物理的cpu，4个处理器就是4个进程

```
#include <stdio.h>
#include <sys/syscall.h>
#include <unistd.h>
#define _USED_GNU
#include <sched.h>

int cpu_bind(int num) {
    pid_t self_id = syscall(__NR_gettid);

    cpu_set_t mask;

    CPU_ZERO(&mask);
    CPU_SET(self_id % num, &mask);

    sched_setaffinity(0, sizeof(mask), &mask);
    while(1);
}
int main() {
    int num = sysconf(_SC_NPROCESSORS_CONF);
    printf("cpu cores: %d\n", num);
    pid_t pid = 0;
    for (int i = 0; i < num; i++) {
        pid = fork();
        // 父进程 返回大于0的值，为子进程的id
        // 子进程 返回0
        if (pid <= (pid_t)0) { // 是子进程，让父进程先行
            usleep(1);
            break;
        }
    }
    if (pid > 0) { // 主进程
        printf("running...\n");
        getchar();
    } else if (pid == 0) {
         cpu_bind(num);
    }
    return 0;
}
```
cpu_bind: ![cpu_bind](/images/linux_thread/cpu_bind.png)
cpu_unbind: ![cpu_unbind](/images/linux_thread/cpu_unbind.png)
绑定后的进程运行效率更高



#  Q&A

- 所有线程共享主线程的虚拟地址空间(current->mm指向同一个地址)，那线程栈是共享的吗？
不是，调用clone的时候需要自己提供子task的栈空间
 
```
/**
 * 负责处理clone,fork,vfork系统调用。
 * clone_flags-与clone的flag参数相同
 * stack_start-与clone的child_stack相同
 * regs-指向通用寄存器的值。是在从用户态切换到内核态时被保存到内核态堆栈中的。
 * stack_size-未使用,总是为0
 * parent_tidptr,child_tidptr-clone中对应参数ptid,ctid相同
 */
long do_fork(unsigned long clone_flags,
          unsigned long stack_start,
          struct pt_regs *regs,
          unsigned long stack_size,
          int __user *parent_tidptr,
          int __user *child_tidptr)
{
    struct task_struct *p;
    ...
}
```

- 进程 vs 线程

    - 进程是资源分配的基本单位
    - 线程是调度和分配的基本单位（并发实体尽可能去共享进程的资源，比如共享一块地址空间，一个页表和一块物理内存）

![](/images/linux_thread/thread.png)


- nginx 为什么会选择多进程绑定的方式?

	- 背景: nginx是在apache之后,apache效率低是因为对多核cpu的支持不够强,多核vs单核没有达到翻倍的效果,但是nginx做到了,在应对多核的情况下采用了“多进程”来做的,而且每一个进程都绑定了cpu,选择nginx因为支持多核

- 线程 vs 协程

    - 每个线程都可以对应一个调度实体，某个线程阻塞了，其他线程还是可以工作
    - 某个协程 调用read(recv)阻塞，等待， 其他协程也在等待
![](/images/linux_thread/st-thread.png)
