---
title: Virtual Memory
date: 2020-11-28 09:24:51
tags: Linux
---

![](/images/virtual_mem/intro.png)
## Handling Page Fault
- page miss causes page fault (an exception)
- page fault handler selects a victim to be evicted(here vp4)

![](/images/virtual_mem/page_fault1.png)

![](/images/virtual_mem/page_fault2.png)

## Tool for caching
- uses main memory efficiently
	use DRAM as a cache for parts of a virtual address space

- DRAM is about **10x** slower than SRAM
- Disk is about **10,000x** slower than DRAM
![](/images/virtual_mem/cache.png)

## Tool for Memory Management

Each process gets the same uniform linear address space, simplify memory allocation
- Each virtual page can be mapped to any physical page
- A virtual page can be stored in different physical pages at different time

Shared libraries, lib.c is the same code for every process running on the system 
![](/images/virtual_mem/manage.png)

## Simplify Linking and loading

- Linking
	- Each program has similar virtual address space
	- Code, data, and heap always start at the same address
- Loading
	- **execve** allocates virtual pages for **.text** and **.data** sections & create PTEs marked as invalid
	- The **.text** and **.data** sections are copied, page by page, on demand by the virtual memory system 
![](/images/virtual_mem/load.png)

## isolate address spaces
	- one process can't interfere with another's memory
	- user program cannot access privileged kernel information and code

# Q&A

- When you load a page from disk into memory, does it also get cached in cached memory hierarchy?
Yes, if you load an entire page, that page will be **broken up into 64 bytes blocks and load it into the cache**. 
So everything you fetch from the memory goes through the cache hierarchy.

- How is the virtual address space implemented on disk?
Most pages there's an option when you allocate a new virtual memory page, you can allocate it so that it's all zeros, you can say I want to allcoate a page of all zeros. In that case, that page doesn't need to get stored on disk, it's just as though it was created on disk and then loaded into memory. **So thoese pages that are all zeros don't exist on disk.** When pages are modified, pages can be mapped to particular files for example when we load and elf binary, the pages that correspond to the code are actually mapped to the bytes in the binary that contains the code. So when you miss on that page it brings in those code pages. **So pages can be mapped to user level files on disk or not(they can be anonymous and not mapped).** So if they're mapped to user level files and you write to a page, then it will get written back to the page that i's mapped to. **If it's not mapped to any page, it's stored in this area called the swap area or the swap file.**


