## 操作系统思考题

**姓名：**李婧如  		**学号：**1700012993 

### 第八讲

#### 1. 调研 SSD

固态盘（Solid State Disk, SSD）由与内存介质相同或相似的存储介质来构建，去除了磁盘中低效的机械运动。由于没有移动的机械部件，固态盘具有很多硬盘不具有的优点：可靠性高；没有噪音；访问速度高，接近内存的访问速度；热耗低于普通硬盘，更省电；不需要旋转，启动时间短；没有机械运动部分，空间利用率高；由于没有磁头磁臂的移动，所有存储单元的访问速度均衡，无需任何调度。固态盘的缺点为：成本较高，是普通硬盘的 10-20 倍；由于对固态盘不同部分使用的频率不同，导致”非均衡磨损“问题，导致固态盘的生命周期缩短（用”磨损平衡“，即将修改后的数据放在新的位置，可以缓解这个问题）；固态盘读写速度差异大于普通硬盘。

#### 2. 系统调用rename

```C
int rename(const char* oldpath, const char* newpath) 
```

分别找到旧文件名和新文件名对应 inode 的父亲节点，记为 `old_parent_inode` 与 `new_parent_inode` ，然后将 `old_parent_inode` 中 `oldpath` 对应的 `refcount` 递减（相当于删除原来文件的目录项），然后在 `new_parent_inode` 中加入 `newpath` 的目录项，不会移动实际的文件。

#### 3. 系统调用copy

用已有系统调用实现：

```C
#define BUF_SIZE 4096
#define OUTPUT_MODE 0700
int main(int argc, char * argv[])
{
	int in_fd, out_fd, rd_cnt, wt_ctn;
    char buffer[BUF_SIZE];
    in_fd = open(argv[1], O_RDONLY);			//open source file
    out_fd = create(argv[2], OUTPUT_MODE); 		//create target file
    while(true) {
		rd_cnt = read(in_fd, buffer, BUF_SIZE); //read from buffer
        if(rd_cnt <= 0) break;					//end file, exit
        wt_cnt = write(out_fd, buffer, rd_cnt); //write to target
     	if(wt_cnt <= 0) exit(4);
    }
    close(in_fd); close(out_fd);
    if(rd_cnt == 0) exit(0);
    else exit(5);
}
```



### 第十一讲

#### 1. 消除死锁的方法

计算机系学生想到了如下的一个消除死锁方法。 当某一进程请求个资源时，规定时间限。如果进程由于得不到需要的资源而阻塞，计时器开始运行。当超过时间限，进程会被释放掉并且允许该进程重新执行。如果你是教师，会给这样的学生多少分？为什么？

我会给较低的分数。首先，进程需要资源被阻塞，超时、被释放后依然需要资源，因此它还是会被阻塞，这不如一直保持阻塞。另外，系统分配新释放的资源时可能按照进程等待时间的长短进行分配，分配给等待事件较长的进程。如果使用这种到时间释放的方式，则进程无法记住自己等待了多长时间。

#### 2. 狒狒过峡谷

以向东通过的狒狒为例：

```C
semaphore mutex = 1; //互斥访问的信号量，保护 eastCnt westCnt 的修改
semaphore east = 0;
semaphore west = 0;
int state = NULL;
int eastCnt = 0;
int westCnt = 0;
void crossEast() {
	down(&mutex);
    if(state==NULL || state==EAST) { //如果没有狒狒或者有同方向的狒狒
		state = EAST;
        up(&east);					 //则允许通过
    }
    eastCnt ++;
    up(&mutex);
    
    down(&east);
    cross();
    
    down(&mutex);
    eastCnt --;
    if(eastCnt==0) { //最后一只向东的狒狒，唤醒等待中的向西方向的狒狒
		state = NULL; 
        for(int i=0; i<westCnt; ++i) {
			up(&west);
        }
        if(westCnt>0) 
            state = WEST;
    }
    up(&mutex);
}
```



#### 3. 不死锁的概率

两个进程A和B，每一个进程都需要申请资源 1，2，3。假如这两个进程都以 1、2、3的次序申请，系统不会发生死锁。但如果 A 以3、2、1的次序申请 ，B以1、2、3的次序申请，则死锁可能发生。试计算当两个进程申请资源的次序不确定的情形下，系统保证不发生死锁的概率是多少？

每个进程申请三种记录的顺序有 3! = 6 种，两个进程间组合有 6*6 = 36 种。只要两个进程第一次申请的资源相同，则不会发生死锁。这是因为第一个进程得到所有资源并释放后，第二个进程才能获得第一个资源。两个进程第一次申请相同资源资源有 3\*2\*2 = 12 种，故不发生死锁的概率为 12/36 = 1/3。