## 锁的实现

### 中断启用与禁止实现
- 进程切换两种方式：主动放弃（`yeild`系统调用）；强制切换（周期性时钟中断获得 CPU 控制权）。执行过程中不会 yeild，只能通过**禁止中断**来实现。但是这个权限是OS的，不能直接给用户使用。所以OS提供了锁。
- 问题是频繁禁止中断导致对事件处理不及时。
```C
lock() {
    disable interrupts;
    while (value != FREE) {   //wait for a context switch from the process that has the lock.
        enable interrupts;
        disable interrupts;
    }
    value = BUSY;
    enable interrupts;
}

unlock() {
    disable interrupts;
    value = FREE;
    enable interrupts;
}
```

### test & set 实现
```C
//将 1 写入 X, 返回 X 原来的值
test_and_set(X) {
    tmp = X;
    X = 1;
    return(tmp);
}

//initialize value as 0
lock() {
    //if value is not zero, loop; else set it to 1 && exit
    while (test_and_set(value) == 1) {}
}

unlock() {
    value = 0;
}
```

### 无忙等待（错误实现）
```C
lock() {
    disable interrupts;
    if (value == FREE) {
        value = BUSY;
    } else {
        // if enable here, switch to unlock and do empty "wakeup", value is FREE but no one can wakeup this thread
        // if wakeup thread also sleeped and go back to this thread, then deadlock
        add thread to a queue waiting for this lock;  //sleep on the lock
        // if enable here, "sleep" state before, but if enable and take up by another, change to "ready" state
        switch to next runable thread;
    }
    enable interrupts;     // if other thread didn't yield nor enable interrupt, no thread can run after it.
}

unlock() {
    disable interrupts;
    value = FREE;
    if (any thread is waiting for this lock) {      //wake up a process
        move waiting thread from waiting queue to ready queue;
        value = BUSY;
    }
    enable interrupts;
}
```
- 正确的实现：
- 切换的时候将中断禁止；
- 从切换返回时启用中断。
