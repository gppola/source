---
title: Linux thread
date: 2020-06-15 12:47:50
tags: Linux
---

# task_struct / mm_struct

一个进程的虚拟地址空间主要由两个数据结构来描述。一个是最高层次的：mm_struct（定义在mm_types.h中），一个是较高层次的：vm_area_structs。最高层次的mm_struct结构描述了一个进程的整个虚拟地址空间。较高层次的结构vm_area_truct描述了虚拟地址空间的一个区间（简称虚拟区或线性区）。每个进程只有一个mm_struct结构，在每个进程的task_struct结构中，有一个指向该进程的结构。可以说，mm_struct结构是对整个用户空间（注意，是用户空间）的描述。

在linux系统里，进程和线程都是通过task_struct结构体来描述。Linux系统 对线程和进程并不特别区分。线程仅仅被视为一个与其他线程共享某些资源的进程。每个线程都拥有唯一自己的task_struct。



![](/images/linux_thread/pcb.png)

![](/images/linux_thread/mm.png)

![](/images/linux_thread/process_whiteboard.png)

# do_fork

![](/images/linux_thread/mm_copy.png)
https://elixir.bootlin.com/linux/latest/source/kernel/fork.c
https://cloud.tencent.com/developer/article/1690373
```
do_fork:
  copy_process:
 	dup_task_struct
	copy_mm:
	  if (clone_flags & CLONE_VM) { // 检查clone_flags中是否有CLONE_VM标志，若有，两个进程之间共享VM，即就是创建轻量级进程(线程)
		mmget(oldmm);
		mm = oldmm;
		goto good_mm;
	  }
	  dup_mm: // 新进程
		mm = allocate_mm(); // 分配一个新的内存描述符。把它的地址存放在新进程的mm中
		memcpy(mm, oldmm, sizeof(*mm)); // 并从当前进程复制mm的内容
		mm_init:
		  mm_alloc_pgd // 函数会调用pgd_alloc()会为该进程分配一页(4K)的页全局目录的线性地址并保存在 task_struct->mm_struct->pgd中
		/**
		 * dup_mmap不但复制了线性区和页表，也设置了mm的一些属性.
		 * 它也会改变父进程的私有，可写的页为只读的，以使写时复制机制生效。
		 */
		dup_mmap:
			for (mpnt = oldmm->mmap; mpnt; mpnt = mpnt->vm_next){
				tmp = vm_area_dup(mpnt);
				if (!(tmp->vm_flags & VM_WIPEONFORK))
					retval = copy_page_range(mm, oldmm, mpnt, tmp);
			}

struct vm_area_struct *vm_area_dup(struct vm_area_struct *orig)
{
	struct vm_area_struct *new = kmem_cache_alloc(vm_area_cachep, GFP_KERNEL);

	if (new) {
		ASSERT_EXCLUSIVE_WRITER(orig->vm_flags);
		ASSERT_EXCLUSIVE_WRITER(orig->vm_file);
		/*
		 * orig->shared.rb may be modified concurrently, but the clone
		 * will be reinitialized.
		 */
		*new = data_race(*orig);
		INIT_LIST_HEAD(&new->anon_vma_chain);
		new->vm_next = new->vm_prev = NULL;
	}
	return new;
}

copy_page_range:
	unsigned long addr = vma->vm_start;
	unsigned long end = vma->vm_end;
	dst_pgd = pgd_offset(dst_mm, addr);
	src_pgd = pgd_offset(src_mm, addr);
	do {
		next = pgd_addr_end(addr, end); // 页全局目录下一条
		if (pgd_none_or_clear_bad(src_pgd))
			continue;
		if (unlikely(copy_p4d_range(dst_mm, src_mm, dst_pgd, src_pgd,
					    vma, new, addr, next))) {
			ret = -ENOMEM;
			break;
		}
	} while (dst_pgd++, src_pgd++, addr = next, addr != end);
```

copy_process函数中首先调用dup_task_struct()函数为子进程创建task_struct结构体等信息，然后根据clone_flags集合中的标志值，设置共享或者复制父进程打开的文件、文件系统信息、信号处理函数、进程地址空间、命名空间等资源，其中copy_mm函数实现父进程地址空间的拷贝，也就是fork创建子进程时的写时复制机制的核心处了，接下来看看这个函数的实现。

copy_mm()中检查clone_flags中是否有CLONE_VM标志，
- 若有，两个进程之间共享VM，即就是创建轻量级进程(线程)，
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

- 为什么要多级页表

	- 因为操作系统是可以同时运行非常多的进程的，那这不就意味着页表会非常的庞大。在 32 位的环境下，虚拟地址空间共有 4GB，假设一个页的大小是 4KB（2^12），那么就需要大约 100 万 （2^20） 个页，每个「页表项」需要 4 个字节大小来存储，那么整个 4GB 空间的映射就需要有 4MB 的内存来存储页表。这 4MB 大小的页表，看起来也不是很大。但是要知道每个进程都是有自己的虚拟地址空间的，也就说都有自己的页表。那么，100 个进程的话，就需要 400MB 的内存来存储页表，这是非常大的内存了，更别说 64 位的环境了。如果使用了二级分页，一级页表就可以覆盖整个4GB 虚拟地址空间，但如果某个一级页表的页表项没有被用到，也就不需要创建这个页表项对应的二级页表了，即可以在需要时才创建二级页表。

	- 这对比单级页表的 4MB 是不是一个巨大的节约？那么为什么不分级的页表就做不到这样节约内存呢？**我们从页表的性质来看，保存在内存中的页表承担的职责是将虚拟地址翻译成物理地址。假如虚拟地址在页表中找不到对应的页表项，计算机系统就不能工作了。所以页表一定要覆盖全部虚拟地址空间，不分级的页表就需要有 100 多万个页表项来映射，而二级分页则只需要 1024 个页表项（此时一级页表覆盖到了全部虚拟地址空间，二级页表在需要时创建）**我们把二级分页再推广到多级页表，就会发现页表占用的内存空间更少了，这一切都要归功于对局部性原理的充分应用。对于 64 位的系统，两级分页肯定不够了，就变成了四级目录，分别是：
		- 全局页目录项 PGD（Page Global Directory）
		- 上层页目录项 PUD（Page Upper Directory）
		- 中间页目录项 PMD（Page Middle Directory）
		- 页表项 PTE（Page Table Entry）

- MMU 和 页表

Each process a pointer (mm_struct→pgd) to its own Page Global Directory (PGD) which is a physical page frame. This frame contains an array of type pgd_t which is an architecture specific type defined in <asm/page.h>. The page tables are loaded differently depending on the architecture. On the x86, the process page table is loaded by copying mm_struct→pgd into the cr3 register which has the side effect of flushing the TLB. In fact this is how the function __flush_tlb() is implemented in the architecture dependent code.
- 所有线程共享主线程的虚拟地址空间(current->mm指向同一个地址)，那线程栈是共享的吗？
不是，调用clone的时候需要自己提供子task的栈空间

threads share all segments except the stack. Threads have independent call stacks, however the memory in other thread stacks is still accessible and in theory you could hold a pointer to memory in some other thread's local stack frame

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
