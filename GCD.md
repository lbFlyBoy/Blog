


#### 一 串行、并行、同步、异步

###### 同步

    1，阻塞当前线程，只到block内执行完成
    2，同步不会创建新的线程，只会在当前所属线程执行） 

###### 串行 (任务按顺序执行)

###### 异步 (开启新的线程)

###### 并行 (任务并发执行)

同步串行：

    dispatch_queue_t serialQueue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_SERIAL);
    NSLog(@"1111");
    dispatch_sync(serialQueue, ^{
        NSLog(@"currentThread %@----2222",[NSThread currentThread]);
    });
    NSLog(@"3333");
    结果：1111
         currentThread 主线程----2222
         3333

异步串行：

    dispatch_queue_t serialQueue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_SERIAL);
    NSLog(@"1111");
    dispatch_async(serialQueue, ^{
        NSLog(@"currentThread %@----2222",[NSThread currentThread]);
    });
    NSLog(@"3333");
    结果：1111
         3333
         currentThread 子线程----2222

同步并行
    
    dispatch_queue_t queue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_CONCURRENT);
        dispatch_sync(queue, ^{
            NSLog(@"currentThread %@----3333",[NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            sleep(5);
            NSLog(@"currentThread %@----4444",[NSThread currentThread]);
        });
        dispatch_sync(queue, ^{
            NSLog(@"currentThread %@ ----5555",[NSThread currentThread]);
        });
    NSLog(@"2222");
     结果：1111
          currentThread 主线程----3333
          currentThread 主线程----4444
          currentThread 主线程----5555
          2222        

异步并行   
    
    NSLog(@"1111");
    dispatch_queue_t queue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(queue, ^{
            NSLog(@"currentThread %@----3333",[NSThread currentThread]);
        });
        
        dispatch_async(queue, ^{
            sleep(5);
            NSLog(@"currentThread %@----4444",[NSThread currentThread]);
        });
        dispatch_async(queue, ^{
            NSLog(@"currentThread %@ ----5555",[NSThread currentThread]);
        });
    NSLog(@"2222");
    结果：1111
         2222
         currentThread 子线程1----3333
         currentThread 子线程2----5555
         currentThread 子线程3----4444

异步全局队列中添加同步并发

    NSLog(@"1111");
    dispatch_queue_t queue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        dispatch_sync(queue, ^{
            NSLog(@"currentThread %@----3333",[NSThread currentThread]);
        });
        dispatch_sync(queue, ^{
            sleep(5);
            NSLog(@"currentThread %@----4444",[NSThread currentThread]);
        });
        dispatch_sync(queue, ^{
            NSLog(@"currentThread %@ ----5555",[NSThread currentThread]);
        });
    });
    
    NSLog(@"2222");
     结果：1111
          2222
         currentThread 子线程1----3333
         currentThread 子线程1----4444
         currentThread 子线程1----5555


#### 二 dispatch_barrier_async（栅栏方法)

在执行完栅栏前面的操作之后，才执行栅栏操作，最后再执行栅栏后边的操作

    NSLog(@"1111");
    dispatch_queue_t queue = dispatch_queue_create("com.dreamlee", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        NSLog(@"任务1");
    });
    
    dispatch_async(queue, ^{
        NSLog(@"任务2");
    });
    
    dispatch_barrier_async(queue, ^{
        NSLog(@"任务barrier");
    });
    dispatch_async(queue, ^{
        NSLog(@"任务3");
    });
    dispatch_async(queue, ^{
        NSLog(@"任务4");
    });
    
    NSLog(@"2222");
    结果：
        1111
        2222
        任务1
        任务2
        任务barrier
        任务3
        任务4

    如果换成 dispatch_barrier_sync 则2222会在“任务barrier”执行完执行

#### 三 

3.1 dispatch_group（队列组)

    NSLog(@"1111");
    dispatch_group_t group =  dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"任务1");
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"任务2");
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"group---end");
    });
    
    NSLog(@"2222");
    结果：
        1111
        2222
        任务1
        任务2
        group---end
  
3.2 dispatch_group_wait（暂停当前线程（阻塞当前线程），等待指定的 group 中的任务执行完成后，才会往下继续执行。）
  
    NSLog(@"1111");  
    dispatch_group_t group =  dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"任务1");
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"任务2");
    });
    
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    NSLog(@"2222");
    
    结果：
        1111
        任务2
        任务1
        2222

3.3 dispatch_group_enter  dispatch_group_leave

     NSLog(@"1111");
    dispatch_group_t group =  dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"任务1");
        dispatch_group_leave(group);
    });
    
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        NSLog(@"任务2");
        dispatch_group_leave(group);
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"group---end");
    });
    NSLog(@"2222");
    结果：
        1111
        2222
        任务2
        任务1
        group---end

3.4 dispatch_semaphore（信号量）
    
    dispatch_semaphore_create：创建一个Semaphore并初始化信号的总量
    dispatch_semaphore_signal：发送一个信号，让信号总量加1
    dispatch_semaphore_wait：可以使总信号量减1，当信号总量为0时就会一直等待（阻塞所在线程），否则就可以正常执行。

