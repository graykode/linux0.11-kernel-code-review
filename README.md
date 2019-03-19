linux0.11-kernel-code-review
==========

The old Linux kernel source ver 0.11 review with line by line for OS lecture. Original code is [here](https://github.com/yuanxinyu/Linux-0.11). More detail information about kernel code review, I recommend [this book](https://www.amazon.com/Art-Linux-Kernel-Design/dp/1466518030). I quoted the table of contents of the book to explain.



## Build on Linux

* a linux distribution: **debian, ubuntu and mint with GUI** are recommended, Ubuntu 16.04  GUI version with VM VirtualBox

1. `sudo apt-get update && sudo apt-get upgrade`
2. `sudo apt-get install build-essential`
3. `sudo apt-get install qemu`
4. `make`
5. `make start`



# Role of Memory Address

## 1. To the Main Function

- Real Model
  - 0x00000 ~ 0x003FF : Interrupt Vector Table
  - 0x00400 ~ 0x004FF : BIOS DATA
  - 0x0E05B ~ 0x0FFFE : Interrupt Service Program
  - bootsect.s : 0x07C000 → 0x90000
  - setup.s(2~5 sectors) : 0x90200
  - kernel(240 sectors) : 0x10000
  - device system data : 0x90000 ~ 0x901FF
- Protect Model
  - kernel : 0x10000 → 0x00000(head.s)
  - GDT : 0x90200 →  0x05CB8(2KB)
  - Page directory Table : 0x00000
  - Page Table 1 : 0x01000
  - Page Table 2 : 0x02000
  - Page Table 3 : 0x03000
  - Page Table 4 : 0x04000
  - Floopy disk buffer : 0x05000
  - IDT : 0x054B8



## 2. Device Initialization(16MB memory) & Process 0

- Setting Root Device and Hard-Disk Setting

- Setting Physical memory layout, buffer memory, ram disk and main memory : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c)

  - Memory layout was divided as main memory, buffer memory, ram disk.

  - end of buffer : 0x3FFFFF
  - end of physical memory : 0xFFFFFF

- Initialization and setting ram disk : [blk.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/blk_drv/blk.h), [ll_rw_blk.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/blk_drv/ll_rw_blk.c), [ramdisk.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/blk_drv/ramdisk.c)

  - ram disk : 0x3FFFFF ~ 0x5FFFFF

- Initialization  mem_map : [memory.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/mm/memory.c)

- Binding Interrupt Service Routine : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c), [traps.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/traps.c), [system.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/include/asm/system.h)

- Initialization Request Struct of Block Device 

  - peripheral devices can be divided with `Block Device` and `Character Device`
  - Request Struct(linked list) : link with peripheral devices and buffer zone

- make HCI Interface and Interrupt Service Routine binding for peripheral devices : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c), [tty_io.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/chr_drv/tty_io.c), [serial.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/chr_drv/serial.c)


#### Initialization Process 0 : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c), [sched.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/sched.c), [system.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/include/asm/system.h), [sched.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/include/linux/sched.h)

  ```c
  union task_union {
  	struct task_struct task;
  	char stack[PAGE_SIZE];
  };
  
  static union task_union init_task = {INIT_TASK,};
  struct task_struct * task[NR_TASKS] = {&(init_task.task), };
  ```

  - GDT
  
|  8   | Process2 TSS2 |
| :--: | :-----------: |
|  7   | Process1 LDT1 |
|  6   | Process1 TSS1 |
|  5   | Process0 LDT0 |
|  4   | Process0 TSS0 |
|  3   |     NULL      |
|  2   |  KERNERL DS   |
|  1   |   KERNEL CS   |
|  0   |    NULLGDT    |
- Timer Interrupt(10ms) Setting for Scheduling

- System Call Setting : System call binds `INT 0x80` of Interrupt Descriptor and `system_call()`

  ```c
  set_system_gate(0x80,&system_call);
  ```

- Initialization Buffer manager Structure : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c)

  - Buffer is gap in Hard Disk and Memory.
  - buffer heads linked with buffer blocks.
  - OS managers `buffer_head` and `hash_table[NR_HASH]`

- Initialization Hard Disk : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c)

- Initialization Floopy Disk : [main.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/init/main.c)

- **Activate Interrupt!** : [system.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/include/asm/system.h)

- Process Privilege level was changed from 0 to 3 and make process

  - All processes have to user level Privilege.
  - `iret` : make Privilege level 0 to 3. (restore)
  - `INT 0x80` : make Privilege level 3 to 0. (saved temporarily)

```assembly
#define move_to_user_mode() \
__asm__ ("movl %%esp,%%eax\n\t" \
	"pushl $0x17\n\t" \
	"pushl %%eax\n\t" \
	"pushfl\n\t" \
	"pushl $0x0f\n\t" \
	"pushl $1f\n\t" \
	"iret\n" \
	"1:\tmovl $0x17,%%eax\n\t" \
	"movw %%ax,%%ds\n\t" \
	"movw %%ax,%%es\n\t" \
	"movw %%ax,%%fs\n\t" \
	"movw %%ax,%%gs" \
	:::"ax")
```



## Process 1

- prepare process 1 : [unistd.h](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/include/unistd.h)

```assembly
#define _syscall0(type,name) \
  type name(void) \
{ \
long __res; \
__asm__ volatile ("int $0x80" \
	: "=a" (__res) \
	: "0" (__NR_##name)); \
if (__res >= 0) \
	return (type) __res; \
errno = -__res; \
return -1; \
}
```

- Privilege level 3 → IDT(INT 0x80) → Privilege level 0 →sys_call_table → call_find_empty_process → copy_process(or find_empty_process) → copy_mem(or get_free_page) → copy_page_tables(or get_limit or get_base or set_base) → get_free_page.
  - Process idle state and apply `pid` in Process 1 : [fork.c](https://github.com/graykode/linux0.11-kernel-code-review/blob/master/kernel/fork.c)
  - call `copy_process()` function
- todo



## Code Review Author

- Name : Tae Hwan Jung(@graykode)
- Email : nlkey2022@gmail.com
