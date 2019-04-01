


#### 一 串行、并行、同步、异步

######同步
    1，阻塞当前线程，只到block内执行完成
    2，同步不会创建新的线程，只会在当前所属线程执行） 

######串行 (任务按顺序执行)

######异步 (开启新的线程)

######并行 (任务并发执行)

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

