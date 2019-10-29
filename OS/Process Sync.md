# 进程同步机制

## 1、Tracy和Peter与金鱼的故事，理解题意并分析各种解法（见课件P97-P101）

- Peter 和 Tracy 通过观察鱼是否被喂饱而决定是否喂食。而观察和喂鱼的动作是分开的，当二人交替执行时将导致冲突访问共享区域。此时鱼被撑死。

| Peter  | Tracy  |
| ------ | ------ |
| 观察鱼 |        |
|        | 观察鱼 |
|        | 喂鱼   |
| 喂鱼   |        |

### 解法一：
- 如果在观察有没有纸条和留下纸条这一步被另一个人打断，则没有起到通知对方的效果，两人都将进入共享区域并类似上面的情况，鱼被撑死。
- 纸条是最初级别的锁，但是一把坏锁，没有保证上锁之后别人进不来。
- 锁的特性：
   - 初始时打开；
   - 进门上锁；（分两步，**等待锁打开**，即检查是否有字条；**将锁锁上**，即留下字条）
   - 出门开锁；
   - 门被锁了的话想进去的人在外面等着（见解法三的繁忙等待，如果喂鱼时间很长，要一直等）；
- 实际上不用把整个喂鱼过程都作为锁，只需要**锁留字条的部分**（减少繁忙等待时间）。

| Peter        | Tracy        |
| ------------ | ------------ |
| 发现没有纸条 |              |
|              | 发现没有纸条 |
|              | 留纸条       |
| 留纸条       |              |
| 观察鱼       |              |
|              | 观察鱼       |
|              | 喂鱼         |
| 喂鱼         |              |

### 解法二：
- 在解法一的基础上，变更了检查纸条、放纸条这一顺序。能够有效防止鱼被撑死，因为留下纸条的动作一定在检查纸条之前发生。所以怎样被打断都不会同时进入临界区。但是如果在 Peter 留纸条后被打断，Tracy 也放纸条，那么两个人都进不了临界区，导致鱼被饿死。

| Peter                   | Tracy                   |
| ----------------------- | ----------------------- |
| 留下 Peter 纸条         |                         |
|                         | 留下 Tracy 纸条         |
| 发现有 Tracy 纸条，不喂 |                         |
|                         | 发现有 Peter 纸条，不喂 |

### 解法三：
- 在解法二的基础上让一个人等另一个人把纸条拿走，再进入临界区。如果发现对方没有喂，再喂鱼。这样鱼总能被喂到。这种方法不会导致鱼撑死或饿死。然而让一个人等着并不公平，且浪费时间资源。同时，若 Peter 的优先级更高，则在 Tracy 放完纸条之后，Peter 抢占式进入。发现有 Tracy 纸条后下 CPU，被阻塞。Tracy 是低优先级进程，总上不了 CPU 执行任务。则没有人喂鱼。
- 改方法程序上不对称，为了使其美观我们修改解法一，把纸条的锁改成好的锁。

## 生产者 消费者
### sleep & wakeup
- 有锁的话去睡觉，不需要等；锁打开后被对方叫醒。`sleep` 后进入休眠，下 CPU；`wakeup` 发送信号给指定的接收进程。
```C
if(count == N) sleep();   //Producer can't insert
insert_item(item);
count = count + 1;
if(count == 1) wakeup(consumer);    //Consumer can remove

if(count == 0) sleep();   //Consumer can't remove
remove_item(item);
count = count - 1;
if(count == N-1) wakeup(producer);   //Producer can insert
```
- 程序的问题：count 没有被保护（竞争的读取、赋值）；如果在判断 `count == 0` 时生产者放了东西，叫醒消费者时发了空信号。一直放到 N 个且 sleep 了，切回消费者之后第一个动作也是 sleep。导致两个进程无法推进（死锁）。
- 解决：在 count 周围加锁；如果信号发送而丢掉了，导致了消费者醒不来，那么想办法保留这个信号即可。（使用信号量）

### semaphore
- 能够保留一个信号的值，不会丢失；只能用 PV 进行操作（不能用 if）；P 操作等价于 down，V 操作等价于 up。需要为信号赋予初始值。
```C
semaphore full = 0;     //full boxes;
semaphore empty = N;    //empty boxes;
semaphore mutex = 0;    //for critical section;

//producer:
P(empty);               //wait if all boxes are full
P(mutex);               //protect critical section
insert_item(item);
V(mutex);
V(full);

P(full);                //wait if all boxes are empty
P(mutex);
remove_item(item);
V(mutex);
V(empty);
```
- 对于生产者来说，`P(empty)` 与 `P(mutex)` 不能互换；因为 mutex 之后发现缓冲区满之后睡眠；消费者试图拿走一个物品，但是 mutex 上了锁，所以没办法拿到。也没有办法唤醒生产者。

### monitor
- OS 把**互斥**这件事交给了编译器解决，通过加 monitor 标记让编译器把某一段代码变成原语。
- 通过条件变量，wait+signal 进行**同步**。条件变量即是一个队列，不满足某种条件的 process 等在某个条件变量上。等满足的时候通过 signal 唤醒之。而唤醒一个变量时，将导致唤醒者、被唤醒者同时活跃。此时将做出选择。
   - Hoare 管程，signal 之后总让**被唤醒者**上 CPU，此时唤醒者停止运行，进入**紧急等待队列**。缺点是 signal 对于唤醒者来说，进程切换了两次。
   - MESA 管程，signal 之后重新调度，通过竞争，让 OS 决定哪个先上 CPU。进入阻塞状态的进程等待 notify 重新回到就绪队列，并等待机会上 CPU。但不知道调度回来时是否仍符合条件，因此用 while 代替 if.这种情况下就没必要一个一个唤醒，可以一下子唤醒整个队列。
   - ```C
     if(coundition) wait();
     change into:
     while(condition) wait();       //after wait, will verify the condition again
     ```
- 实现细节：wait 或者 leave 的时候，先从紧急等待队列里头挑；如果没有等着的进程，再释放锁，让外面等着的进来。
