## 对 5-12，16-17 给出描述

#### 5-BOUND Range Exceeded，边界异常

BOUND 指令有两个参数，一个待检查的寄存器（r16 或 r32），对应两个内存地址（上界和下界）。BOUND 指令用于检查有符号的数组索引是否在内存规定范围之内。如果不在，则发生 `BOUND Range Exceeded` 错误。

无差错码；CS:EIP（指令指针）指向产生异常的 BOUND 指令。

#### 6-Invalid Opcode Exception，无效操作码异常

操作码无效异常，操作码是汇编指令的编号。以下情况将导致该异常：

1. 试图执行无效或保留的操作码；
2. 指令的操作数和相应的操作码不匹配。如LES指令（内存中指定位置的双字操作数的低位字装入指令中指定的寄存器、高位字装入ES寄存器）的源操作数不是内存位置；
3. 运行当前处理器版本不支持的指令（如MMX、 SSE/SSE2/SSE3指令只能在有多媒体扩展的CPU上运行）；
4. 执行UD2指令；
5. LOCK 指令前缀配合规定的指令，可以在执行规定指令中锁定一个特定内存地址，当这个特定内存地址被锁定后，它就可以阻止其他的系统总线读取或修改这个内存地址。指令执行结束后取消锁定。当添加LOCK指令前缀但其下一条指令的目的操作数不是内存地址时产生异常；
6. 当保护模式未启用或当虚拟8086模式启用时，执行LLDT, SLDT, LTR, STR, LSL, LAR, VERR, VERW或ARPL指令；
7. 不在SMM模式时执行RSM。

无差错码；指令指针指向产生该异常的指令。

#### 7-Device Not Available Exception，设备不可用异常

该异常由以下三种情况产生：

1. CR0.EM = 1 时，执行一条WAIT/FWAIT之外的x87 FPU浮点指令；
2. 无论CR0.EM是否设定，当CR0.MP = 1 && CR0.TS = 1时执行WAIT/FWAIT指令；
3. CR0.TS = 1，CR0.EM = 0 时执行 x87 FPU, MMX, 或者 SSE/SSE2/SSE3 指令 (MOVNTI, PAUSE, PREFETCHh, SFENCE, LFENCE, MFENCE, 和 CLFLUSH除外) 

无差错码；指令指针指向产生该异常的指令。

#### 8-Double Fault Exception，双故障异常

该异常说明当第一个异常处理过程中处理器检测到第二个异常。正常情况下第二个异常会被顺次处理。为了描述不正常的情况，处理器将异常分为三种：良好的异常，助推的异常（指该异常出现将影响第二个异常的处理），以及缺页异常。以下情况将导致双故障异常产生：

1. 助推异常+助推异常；
2. 缺页异常+助推异常/缺页异常

若双故障异常处理过程中缺页异常或助推异常进入，则处理器进入停机状态。

- **良好的中断异常**

| 中断向量号 | 名称                        |
| ---------- | --------------------------- |
| 1          | debug                       |
| 2          | NMI Interrupt               |
| 3          | Breakpoint                  |
| 4          | Overflow                    |
| 5          | BOUND Range Exceeded        |
| 6          | Invalid Opcode              |
| 7          | Device Not Available        |
| 9          | Coprocessor Segment Overrun |
| 16         | Floating-Point Error        |
| 17         | Alignment Check             |
| 18         | Machine Check               |
| 19         | SIMD floating-point         |
| All        | INT n                       |
| All        | INTR                        |

- **助推的异常**

| 中断向量号 | 名称                |
| ---------- | ------------------- |
| 0          | Divide Error        |
| 10         | Invalid TSS         |
| 11         | Segment Not Present |
| 12         | Stack Fault         |
| 13         | General Protection  |

- **缺页异常**

| 中断向量号 | 名称                     |
| ---------- | ------------------------ |
| 14         | Page Fault               |
| 20         | Virtualization Exception |

#### 10-Invalid TSS Exception，无效TSS异常

该异常发生于任务切换过程，或者用到TSS中信息的指令执行时。可能发生于状态切换之前的任务，或者发生于切换后的任务。在新 TSS 的存在被完全确认之前，异常在原来的任务中产生；一旦新 TSS 被确认，任务切换完成。在这个节点后发生的无效 TSS 情况则在新 TSS 环境中处理。

错误码：包含产生该异常的段描述符中的段选择符索引将被压入栈。如果 EXT 位被设定，则说明该异常由正在运行程序之外的程序产生（如使用任务门的中断处理试图切换到无效的TSS）。

保存的指令指针：如果在任务切换之前发生，则保存触发任务切换的指令。如果在任务切换之后发生，则保存新任务的第一条指令。

- **TSS (Task State Segment)段选择符索引**
  - TSS 段的界限小于 67H(32-bit)/2CH(16-bit)
  - IRET 中，TI 位指向 LDT；TSS 段选择符超过描述符表的界限；TSS 描述符中 busy 位指向不活跃任务；尝试加载回退链接（之前任务的选择符，指向当前任务应该返回的任务）时超过限制；回退链接为空选择符；回退链接指向的描述符不是一个忙碌的TSS；
  - 新TSS出现问题，包括新 TSS 描述符超过了 GDT 界限；新 TSS 描述符不可写；由于调用或者异常事件切换，新 TSS 的回退链接不可写；试图锁住新的 TSS 时，新 TSS 为空、新 TSS 设置了 TI 位或新 TSS 描述符不是可用的 TSS 描述符；
  - 之前的 TSS 出现问题，包括试图保存旧 TSS 时遇到错误情况；旧 TSS 描述符由于遇到 jump 和 IRET 任务切换而不可写；
  - 在 LTR 指令（Load Task Register，将新任务的选择符加载到 Task Register 上）时 TSS 段选择符为空或TSS段选择符设置了 TI 位；TSS 段描述符/最大描述符超过了 GDT 段界限；  
- **LDT(Local Discriptor Table 局部描述符表) 段选择符索引**
  - LDT 未出现；
- **栈的段选择符索引**
  - 栈段选择符超过描述符表的界限；栈段选择符为空；栈段描述符是非数据的段；栈段不可写；栈段的 DPL $\neq$ CPL，或栈段的选择符 RPL $\neq$ CPL；
- **代码段选择符索引**
  - 代码段选择符超过描述符表的界限；代码段选择符为空；代码段描述符不是代码段类型；访问 NCCS 时 DPL $\neq$ CPL；访问 CCS 时 DPL 大于 CPL；（其中 NCCS 指 nonconforming code segment， CCS 指 conforming code segment；CCS 可以被比自己权限低的应用访问，而 NCCS 要求被同等权限的应用访问）
- **数据段选择符索引**
  - 数据段选择符超过描述符表的界限；数据段描述符不是可读代码或数据类型；数据段描述符是 nonconforming 代码类，且 CPL > DPL；

#### 11-Segment Not Present，段不存在
说明当前段或门描述符的 present flag 被清空。下列情况发生时会产生该异常：试图加载 CS，DS，ES，FS 或 GS 寄存器时；试图用 LLDT (Load Local Descriptor Table Register)加载 LDTR 时；在 TTS 被标记为不存在时执行 LTR 指令；试图使用被标记为“段不存在”的门描述符或TSS。

操作系统运用该异常实现段层面的虚拟内存。如果异常处理程序载入段并返回，被中断的任务继续执行。
错误码：包含产生该异常的段描述符的段选择符索引；如果 EXT 被设置，则说明异常由下列情况产生：外部事件（NMI 或 INTR）产生了中断，紧接着引用了一个不存在的段；良好的异常之后引用了不存在的段。如果错误码引用了中断描述符表的入口，则设置 IDT 位。这种情况当 IDT 入口指向不存在的门时发生。该事件可能由 INT 指令或硬件中断产生。

保存的指令指针：指向产生这条异常的指令。如果发生在加载新的 TSS 的段描述符时，指针指向新任务的第一条指令；如果异常在访问门描述符时产生，指针指向出发访问的指令（比如 CALL）。

程序状态的变化：如果是因为加载寄存器时寄存器未被加载，则恢复时需要把缺失的段加载到内存并设置段描述符中的 present flag。

#### 12-Stack Fault，栈错误

发生以下错误时产生该异常：

- 引用 SS (Stack Segment) 寄存器（用于存储寄存器的内容）时超出界限。使用栈相关的指令，如 pop push call 等时若栈空间不足会触发该异常。
- 当加载 SS 寄存器时检测到不存在的栈段，可能在任务切换，CALL 或 return 指令到不同的特权级，LSS，MOV，POP 指令。
- 64 bit 模式下探测到一个用非规范地址访问内存的操作。

错误码：如果异常由不存在的栈段或不同特权级间的调用时新栈的溢出，错误码包含造成该异常的段的段选择符。中断处理程序能够通过测试段选择符指向的段描述符中的 present flag 来确定异常产生的原因。

指令指针一般指向产生该异常的指令；但是在任务切换过程中，试图加载不存在的栈段时，指令指针指向新任务的第一条指令。

#### 16-Coprocessor Segment Overrun，协处理器段超限

此异常为 intel 保留，不被使用。最近的 IA-32 处理器不产生该异常。异常出现的原因是 x87 协处理器上执行指令时，在操作数上引用了无效页或无效段。

#### 17-Alignment Check Exception，对齐检查

当处理器检测到未对齐的内存操作时产生。对齐检查只在数据或栈的访问中执行。

## linux task struct

```C
struct task_struct {
    volatile long state; /* -1 unrunnable, 0 runnable, >0 stopped */
    struct thread_info *thread_info;
    atomic_t usage; /* 进程描述符使用计数，被置为2时，表示进程描述符正在被使用而且其相应的进程处于活动状态 */
    unsigned long flags; /* per process flags, defined below */
    unsigned long ptrace; /* ptrace系统调用，成员ptrace被设置为0时不需要被跟踪 */
    int lock_depth; /* 表示获取大内核锁的次数，未获得过锁则为 -1 */
    int prio, static_prio; /* 调度器的优先级保存在 prio，有些情况下需要暂时提高进程优先级。static_prio：进程的“静态优先级”，启动时分配，运行期间保持恒定 */
    struct list_head run_list;
    prio_array_t *array;
    unsigned long sleep_avg; /* 进程的平均睡眠时间 */
    long interactive_credit; /* 表示交互性的数值，若睡眠时间长则+1，运行时间长则-1 */
    unsigned long long timestamp;
    int activated; /* 进程被唤醒时使用的条件代码，即从什么状态被唤醒 */
    unsigned long policy; /* 表示进程调度策略 */
    cpumask_t cpus_allowed; /* 多处理器上使用，控制进程可以在哪些处理器上运行 */
    unsigned int time_slice, first_time_slice; /* time_slice 表示进程剩余的时间片；first_time_slice 表示是否将子进程的时间片还给父进程 */
    struct list_head tasks; /* 构建进程链表 */
    /*
    * ptrace_list/ptrace_children forms the list of my children
    * that were stolen by a ptracer.
    */
    struct list_head ptrace_children;
    struct list_head ptrace_list;
    struct mm_struct *mm, *active_mm; /* mm：进程所拥有的内存描述符；active_mm：进程运行时所使用的内存描述符 */
    /* task state */
    struct linux_binfmt *binfmt;
    int exit_code, exit_signal; /* exit_code 设置进程终止代号，正常终止或异常终止；exit_signal 进程退出时发给父进程的信号 */
    int pdeath_signal; /* The signal sent when the parent dies */
    /* ??? */
    unsigned long personality; /* 用于处理不同的 ABI */
    int did_exec:1; /* 记录进程代码是否被 execve() 函数所执行 */
    pid_t pid; /* 进程标识符 */
    pid_t tgid; /* threadgroup id */
    /*
    * pointers to (original) parent process, youngest child, younger
    sibling,
    * older sibling, respectively. (p->father can be replaced with
    * p->parent->pid)
    */
    struct task_struct *real_parent; /* real parent process (when
    being debugged) */
    struct task_struct *parent; /* parent process */
    /*
    * children/sibling forms the list of my children plus the
    * tasks I'm ptracing.
    */
    struct list_head children; /* list of my children */
    struct list_head sibling; /* linkage in my parent's children list */
    struct task_struct *group_leader; /* threadgroup leader */
    /* PID/PID hash table linkage. */
    struct pid_link pids[PIDTYPE_MAX];
    wait_queue_head_t wait_chldexit; /* for wait4() */
    struct completion *vfork_done; /* for vfork() */
    int __user *set_child_tid; /* CLONE_CHILD_SETTID */
    int __user *clear_child_tid; /* CLONE_CHILD_CLEARTID */
    unsigned long rt_priority; /* 表示实时进程的优先级 */
    unsigned long it_real_value, it_prof_value, it_virt_value;
    unsigned long it_real_incr, it_prof_incr, it_virt_incr;
    struct timer_list real_timer;
    unsigned long utime, stime, cutime, cstime;
    unsigned long nvcsw, nivcsw, cnvcsw, cnivcsw; /* context
    switch counts (nvcsw：自愿上下文切换计数；nivcsw：非自愿上下文切换计数)*/
    u64 start_time; /* 进程创建时间 */
    /* mm fault and swap info: this can arguably be seen as either
    mm-specific or thread-specific (缺页统计)*/
    unsigned long min_flt, maj_flt, cmin_flt, cmaj_flt;
    /* process credentials */
    uid_t uid,euid,suid,fsuid;
    gid_t gid,egid,sgid,fsgid;
    struct group_info *group_info;
    /* permitted 表示进程能够使用的  capabilities 的上限，
    内核检查某线程是否能够进行特权操作时检查 effective，
    执行 exec() 时能够被新的可执行文件继承的 capabilities 包含在 inheritable 中
    */
    kernel_cap_t cap_effective, cap_inheritable, cap_permitted;
    int keep_capabilities:1;
    struct user_struct *user;
    /* limits */
    struct rlimit rlim[RLIM_NLIMITS]; /* 表示进程能使用的资源限制 */
    unsigned short used_math;
    char comm[16];
    /* file system info */
    int link_count, total_link_count;
    /* ipc stuff */
    struct sysv_sem sysvsem;
    /* CPU-specific state of this task */
    struct thread_struct thread;
    /* filesystem information (表示进程与文件系统的联系，包括当前目录、根目录) */
    struct fs_struct *fs;
    /* open file information (进程当前打开的文件) */
    struct files_struct *files;
    /* namespace */
    struct namespace *namespace;
    /* signal handlers */
    struct signal_struct *signal; /* 指向进程的信号描述符 */
    struct sighand_struct *sighand; /* 指向进程的信号处理描述符 */
    sigset_t blocked, real_blocked; /* 表示被阻塞信号的掩码 */
    struct sigpending pending; /* 存放私有挂起信号的数据结构 */
    unsigned long sas_ss_sp; /* 信号处理程序备用堆栈的地址 */
    size_t sas_ss_size; /* 表示堆栈大小 */
    int (*notifier)(void *priv);
    void *notifier_data;
    sigset_t *notifier_mask;
    void *security;
    struct audit_context *audit_context;
    /* Thread group tracking */
    u32 parent_exec_id;
    u32 self_exec_id;
    /* Protection of (de-)allocation: mm, files, fs, tty */
    spinlock_t alloc_lock;
    /* Protection of proc_dentry: nesting proc_lock, dcache_lock,
    write_lock_irq(&tasklist_lock); */
    spinlock_t proc_lock;
    /* context-switch lock */
    spinlock_t switch_lock;
    /* journalling filesystem info */
    void *journal_info;
    /* VM state */
    struct reclaim_state *reclaim_state;
    struct dentry *proc_dentry;
    struct backing_dev_info *backing_dev_info;
    struct io_context *io_context;
    unsigned long ptrace_message;
    siginfo_t *last_siginfo; /* For ptrace use. */
    #ifdef CONFIG_NUMA
    struct mempolicy *mempolicy;
    short il_next; /* could be shared with used_math */
    #endif
};

```

## 同步互斥机制

#### PV 操作实现 receive 原语

```C
Recieve(source, message)
Begin
    根据 source 找到发送进程对应的消息缓冲区，如果未找到，出错返回；
    P(buf-full); //等待发送进程把东西放到缓冲区
		P(mutex);
			取缓冲区;
			将缓冲区中的值复制到 message;
            清空缓冲区
        V(mutex);
	V(buf-empty);
End

//buf-empty 初值为n
//buf-full 初值为0
//mutex 初值为1
```

#### TSL 指令对多处理器是否有效

TSL 指令对内存字 lock 进行操作，将 lock 读到 RX 寄存器中，再在内存地址上写下非零值。执行 TSL 的 CPU 将锁住内存总线，禁止其他 CPU 在本指令结束前访存，因此适用于多处理器。

#### XCHG 指令

```asm
enter_region:
	MOVE REGISTER, #1		| 在寄存器变量中放1
	XCHG REGISTER, LOCK 	| 交换寄存器与锁内容
    CMP REGISTER, #0		| 判断锁是否为0
    JNE enter_region 		| 不为0，则循环
    RET					    | 返回调用者，进入临界区

leave_region:
	MOVE LOCK, #0    		| 在锁中存入0
	RET 					| 返回调用者
```

### 