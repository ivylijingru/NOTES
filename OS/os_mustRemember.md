### 期中

- 一个支持 100 个 SYSCALL 的操作系统如何实现？中断向量表中有系统调用总入口地址，系统调用总入口中有 100 个 SYSCALL 的地址，调用某一个 SYSCALL 时，硬件保存现场，压入系统调用号，通过查中断向量表把控制权交给系统调用总入口程序。总入口程序保存现场，**将参数从寄存器存到内核栈中**；通过查系统调用表把控制权转让给相应系统调用处理例程。执行系统调用例程。恢复现场，返回用户程序。

- 已有 15 个SYSCALL 基础上添加 SYSCALL16，硬件和 OS 分别做哪些工作。软件在系统调用总入口处新增 16 这个系统调用号，并添加该系统调用的处理函数。

- 画出进程的虚拟空间布局。说明 PCB 中如何体现进程虚拟空间的？代码段，数据段，堆，栈（注意这里的增长方向，中间夹着共享库、内存映射文件）。PCB 中将地址空间组织成一个 AVL 树，每个区域有权限和保护位。

- 仓库中保存两种材料 A、B，每种最大个数为 M，两个消费者取用材料 A 和 B，两个生产者提供 A 和 B。规定库存的材料先进出 ；一种材料比另一种多出 N 个的时候暂停该材料的生产（N＜M）。通过管程 Monitor 实现这个同步互斥机制 ，写出完整代码。

- ```pascal
  monitor ProducerConsumer:
  	integer cnt1, cnt2;
  	condition full_A, full_B, empty_A, empty_B, more_AB, more_BA;
  	
  	procedure insert_A(item : integer)
  	begin
  		if cnt1 == M then wait(full);
  		if cnt1 - cnt2 >= N then wait(more_AB);
  		if cnt2 - cnt1 == N-1 then signal(more_BA);
  		insert_item(item); cnt1 ++;
  		if cnt1 == 1 then signal(empty);
  	end;
  	
  	procedure insert_B(item : integer)
  	begin
  		if cnt2 == M then wait(full);
  		if cnt2 - cnt1 >= N then wait(more_BA);
  		if cnt1 - cnt2 == N-1 then signal(more_AB);
  		insert_item(item); cnt2 ++;
  		if cnt1 == 1 then signal(empty);	
  	end;
      
      function remove_A : item 
      begin
      	if cnt1 == 0 then wait(empty);
      	remove_A = remove_item; cnt1 --;
      	if cnt1 == M-1 then signal(full);
      end;
      
      function remove_B : item
      begin
      	if cnt2 == 0 then wait(empty);
      	remove_B = remove_item; cnt2 --;
      	if cnt2 == M-1 then signal(full);
      end;
  ```

- 消费者问题，资源为K，要求消费者必须等某个消费者拿完 5 个之后才能开始拿。用 PV 操作完成这个问题。

- ```
  semaphore wait_ = 5
  
  void others() {
  
  }
  
  void this_one() {
  
  }
  ```

- 分别用信号量和管程实现，霍尔管程和 hansen 管程的区别

### 内存

- 内存映射和写时复制的相似之处
  - 内存映射的基本思想：通过一个**系统调用**将文件映射到虚拟地址空间的一部分，访问文件就像访问内存中的大数组，而不是对文件进行读写。多数实现中，映射共享的页面时不会实际读入页面内容，而是在访问页面时，才会被每次一页地读入，磁盘文件被当作后备存储。进程退出或显式解除文件映射时，被修改页面会写回文件。
  - 写时复制主要改页表的权限。两个进程将私有对象映射到虚拟内存的不同区域，页表条目标记为只读。如果有进程试图写私有区域内某页面，则创建一个页面的新副本，更新页表条目，恢复该页可写权限。
  - 减少写磁盘的次数。内存映射将页面读入尽可能往后拖，写时复制将复制尽可能往后拖。
  
- 内存映射文件是什么？与虚拟内存管理有什么联系？内存映射文件是将文件映射到虚拟内存的一部分。存到内存映射文件区域中。

- 详细写出 copy on write 的实现（包括设计思路、用到的数据结构）设计思路为：进程复制数据时，从来不会进行写操作的页面是不用复制的，只有实际修改的页面需要。两个进程共享数据时，进程有自己的页表，但指向同一个页面集合，页表标为只读，**区域结构标记为私有的写时复制**（COW：必须有一个地方标志 READONLY 不是真错，TASK_STRUCT 中标 PRIVATE SHARE）。其中一个进程更新了数据，则触发只读保护，引发操作系统陷阱。故障处理程序查看地址空间信息，判断触发复制该页。此时生成该页副本，修改页表中的物理页面，将两个进程页表都设成读写。（fork 中只有 exec 才会改父进程代码部分，其他情况下不会对父进程代码部分做改动）

- PTE 包含哪些内容？为何这样设计？禁止缓存，有效位，访问位，保护位，物理页号，**修改位**

- 反转页表的设计思路？如何将虚拟地址转化为物理地址？正向页表每个进程建立一张，以虚拟内存地址为索引。由于虚拟内存远大于物理内存，故考虑以物理内存为索引建立页表，每个系统一张表。地址转化：给定进程 PID 和虚拟地址，哈希到一个物理地址，再沿着冲突链查找是否有表项填写了相同的 PID 和虚拟地址。若找到，则该项对应的物理地址即为所求。

- 48 位虚拟内存设计，第一问设计多级页表，第二问设计反转页表。并描述访问过程。

- 缺页异常的处理过程。硬件查页表时发现要访问的页不在内存，产生缺页异常。执行缺页异常处理程序，获得磁盘地址，启动磁盘。如果内存中有空闲框，则分配一页，将该页调入内存，修改页表中页框号和有效位。若没有空闲框则置换一页。若该页在内存中被修改，则写回磁盘。

- 系统在解析某内存地址中的行为（软件、硬件），自选内存组织方式

- ```C
  if(虚拟页面不在内存、页面非法、或被保护) { //陷入操作系统部分是软件机制
  	硬件产生异常，陷入操作系统，执行 PAGE FAULT 程序 
  } else { //硬件机制
  	页框号 = 页表[虚页号]
  	物理地址 = 页框号 concatenate 页内偏移
  }
  ```

- windows 虚拟内存管理每个箭头是什么场景？体现了什么思路？

  | 箭头        | 场景                                             | 思路                                                         |
  | ----------- | ------------------------------------------------ | ------------------------------------------------------------ |
  | 工作 2 空闲 | 进程结束，释放占用的页面                         |                                                              |
  | 空闲 2 工作 | 发生缺页，进程请求页面                           |                                                              |
  | 工作 2 后备 | 工作集管理器移除未修改页面到后备页面链表         | 定时清除页面，维持空闲页面数量在一定值内                     |
  | 工作 2 修改 | 工作集管理器移除修改页面到修改页面链表           |                                                              |
  | 后备 2 工作 | 发生软缺页异常，从内存中恢复该页面，修改页表状态 | 页面缓冲的思想，选择清除某页面的时候不直接写回磁盘，而是在内存中维护缓冲区，再用到它时可直接取 |
  | 修改 2 工作 | 发生软缺页异常，从内存中恢复该页面，修改页表状态 |                                                              |

  

### 文件系统

- 进程通过 `fopen` 系统调用读取文件过程与 PCB FCB 的关系？根据文件名在文件目录中检索，将文件 FCB 读入内存，根据文件号查系统打开文件表，看文件是否已被打开；是，则共享计数+1，否则将 FCB 信息填入系统打开文件表空表项，共享计数置1；根据该进程 PCB 中记录的用户打开文件表位置，找到用户打开文件表，从用户打开文件表中取空表项，填写打开方式等，并指向系统打开文件表对应表项。返回文件描述符 `fd`，用于以后读写文件。

- 创建一个文件（核心是建立 FCB，和 PCB 关系不大），PCB 和 FCB 有什么行为？1）为文件分配块；2）在目录中为新文件**建立一个目录项**，根据参数填写有关内容

- UNIX FCB 分解技术，提升文件系统性能：FCB = 目录项 + INODE，目录项 = 文件名 + INODE 号，INODE：描述文件相关信息。能够减少查找文件的访盘次数。

- UNIX 文件系统块的名字及作用（INODE 区采用综合模式的多级索引结构，15 个索引项，每项 2 字节；前 12 项直接寻址即存放物理块号，13-15 分别存放一二三级索引表的块号，索引表中能存放的表项个数 = 块大小/2）

  | 引导记录                         | superblock                 | 空闲区管理       | INODE 区                                           | 根目录区 | 文件和目录区                                         |
  | -------------------------------- | -------------------------- | ---------------- | -------------------------------------------------- | -------- | ---------------------------------------------------- |
  | 从该分区引导操作系统所需要的信息 | 记录之后每一个区从哪里开始 | 存放空闲块的位置 | FCB 的主要内容；INODE 号对应的文件装在哪个文件块中 |          | 目录文件则装下级文件的名称+INODE号，普通文件则装内容 |

- UNIX 文件系统设计中，根目录区前有哪些部分？

- 如何确定根目录起始地址？如何设计空闲区管理？

- FAT 文件系统块的名字及作用

  | 引导区             | FAT1                             | FAT2        | 根目录    | 其他目录和文件 |
  | ------------------ | -------------------------------- | ----------- | --------- | -------------- |
  | 记录根目录起始簇号 | 根目录从第二簇开始，采用链接结构 | FAT1 的副本 | 预留 4 簇 |                |

- 画图：1）每个目录文件下有固定的点、点点两个目录项；2）FAT 链接结构最后一块用 -1表示；3）FAT 根目录固定在第 2 簇，留 4 簇，UNIX 不用留；4）UNIX 目录项是文件名 + INODE 号，INODE 中放某文件在数据区的哪一块；5）画图时每个块下标明白文件、文件夹的名字；6）INODE 放装不下的文件时用一级索引

- 删除文件时访问几次磁盘？
- 复制某个文件到另一目录下过程怎样？启动磁盘几次？根据路径找到另一个目录，在这个目录下新建文件，读原先文件到 BUFFER，再从 BUFFER 写到新建立的文件。写成功后将两个文件关闭。
- 磁盘缓冲技术（那张图）体现的思想？用户对磁盘的访问通过访问文件缓存实现，当出现某一特定块的 I/O 请求时，首先检测以确定该块是否在磁盘高速缓存中。如果在，则可直接进行读操作，否则，先要将数据块读到磁盘高速缓存中，再拷贝到所需的地方。由于局部性原理，一数据块被读入磁盘高速缓存时，有可能将来还会被访问。读取数据的时候预取（prefetch），LRU 原则清除缓存的内容，（WRITE BACK）用户写数据时只更改 CACHE 中的内容，不直接反映到磁盘上，而由 CACHE Manager 决定何时写回磁盘。
- 使用磁盘高速缓存的好处？
- RAID 将数据分成数据块，并行读出、写入多个磁盘块，提升文件的传输效率。同时使用镜像、校验获得容错能力。
- 磁盘调度：从最近的柱面号开始，相同柱面不同扇区则从小到大。
- 给 32 GB 的 U 盘自己设计一个文件系统，要求可以从 U 盘启动，每个磁盘块大小 1024 B，最大支持 2 GB 的文件大小 ，**空闲块管理只用到一个磁盘块**，支持随机访问 ，注意文件性能 。（定死了成组链接法，三级索引装得下 2GB/1KB = 2^21 个块）

### I/O

- 内存映射编制和 I/O 独立寻址的**区别**？

  |          | 独立编址                                                     | 内存映射                                                     |
  | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 内容     | 所有端口地址空间完全独立，与内存地址空间无关；使用专门 I/O 指令进行操作 | 所有端口地址空间和内存地址空间统一编址；I/O 操作等同于内存操作 |
  | 优点     | 编程时能区分对内存或 I/O 操作                                | 避免把控制寄存器的地址空间放入用户的虚拟地址空间，则能阻止用户执行 I/O 操作；不需要特殊保护机制 |
  | 指令     | I/O 端口操作的指令类型少，操作不灵活                         | 不需要专门的 I/O 指令，能操作内存的都可操作 I/O              |
  | 地址空间 | 外设不占用内存地址空间                                       | I/O 端口可占有较大地址空间                                   |
  | 编程     | 设备驱动程序只能用汇编写                                     | 可以用 C 语言                                                |
  | 其他     |                                                              | 需要在页表中引入禁止缓存；共用一个地址空间则必须检查所有的内存引用，特别是具有内存总线的情况 |

  

- 内存映射的问题？对**设备控制寄存器**不能高速缓存。第一次引用 port-4 导致它被高速缓存，随后的引用将只从高速缓存中取值且不会查询设备，设备就绪时将无法发现。硬件必须具备禁用高速缓存的能力。

- ```asm
  LOOP: 
  	TEST PORT-4
  	BEQ READY
  	BRANCH LOOP
  READY:
  ```

- 以打印机输出为例说明 I/O 软件的设计思想。逻辑 I/O 层把打印机当作逻辑资源来处理，不关心实际控制设备的细节。打印机驱动程序将打印请求和数据转换成适当的 I/O 指令序列、通道命令和控制器指令。如果打印机在使用，请求被排入队列以备稍后处理。随后打印机处理请求，请求完成后设备产生中断，中断处理程序唤醒打印机驱动程序进程，进行中断处理。

- 同步异步 I/O：同步 I/O，应用程序被阻塞直到 I/O 完成。异步 I/O，应用程序启动 I/O 操作，在 I/O 请求执行的同时继续处理。具体实现，切换到其他线程保证 CPU 利用率。

- 硬件驱动程序的目的和作用。作用是设置设备寄存器、检查设备运行状态。目的是隐藏底层硬件的功能，为上层提供简洁的接口。同时将上层请求转化为具体设备的操作。

- I/O 管理的主要任务是什么？1）完成用户请求；2）建立方便、统一、独立于设备的接口；3）利用各种技术提高性能；4）保护



NTFS

删除时加 INODE 标志位，不用把页面全都写成 0

页目录项的作用？ 