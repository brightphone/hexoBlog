---
layout: post
title: "iOS锁"
date: 2020-05-20 12:00:00
comments: true
catagories: language
tags: [iOS]
---

当多个线程同时操作同一块资源或者说同一个对象的时候，可能会造成各种意想不到的情况(比如数据错乱、资源争夺、崩溃等)，而锁就是为了能够保证同一时刻只有一个线程在操作这个数据应运而生的，为了保证数据的原子性。
目前iOS中实现锁的方法也比较多

<!--more-->
![image](/res/images/article/ioslock/1.png)
![image](/res/images/article/ioslock/1.webp)

# OSSpinLock 自旋锁
OS_SPINLOCK_INIT： 默认值为 0,在 locked 状态时就会大于 0，unlocked状态下为 0
OSSpinLockLock(&oslock)：上锁，参数为 OSSpinLock 地址
OSSpinLockUnlock(&oslock)：解锁，参数为 OSSpinLock 地址
OSSpinLockTry(&oslock)：尝试加锁，可以加锁则立即加锁并返回 YES,反之返回 NO

如果线程进来发现锁被别人加过了,线程就会在这里等待,相等于线程就会阻塞这个位置,常见的线程阻塞有两种方案
一种是相等于写一个外循环,一直在等,一直执行代码,一直占用CPU资源,自旋锁就是这样的相当等于while(锁还没有被放开);一直执行.一旦锁放开,外循环退出.(自旋锁会出现优先级反转)
还有一种是让线程直接睡觉,直接睡眠,睡觉意味着不占用CPU资源.
自旋锁是计算机科学用于多线程同步的一种锁，线程反复检查锁变量是否可用。由于线程在这一过程中保持执行，因此是一种忙等待。一旦获取了自旋锁，线程会一直保持该锁，直至显式释放自旋锁。自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的。因此操作系统的实现在很多地方往往用自旋锁。
iOS 10.0开始被deprecated了，在 iOS 10/macOS 10.12 发布时，苹果提供了新的 os_unfair_lock 作为 OSSpinLock 的替代，并且将 OSSpinLock 标记为了 Deprecated。
被废弃的原因：[参考](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)
新版 iOS 中，系统维护了 5 个不同的线程优先级/QoS: background，utility，default，user-initiated，user-interactive。高优先级线程始终会在低优先级线程前执行，一个线程不会受到比它更低优先级线程的干扰。这种线程调度算法会产生潜在的优先级反转问题，从而破坏了 spin lock。具体来说，如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock。目前完全可以用o s_unfair_lock取代OSSpinLock，等待os_unfair_lock锁的线程会处于休眠状态，从用户态切换到内核态，而并非忙等。

```
#import <libkern/OSAtomic.h>
__block OSSpinLock oslock = OS_SPINLOCK_INIT;
//线程1
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"线程1 准备上锁");
    OSSpinLockLock(&oslock);
    sleep(4);
    NSLog(@"线程1");
    OSSpinLockUnlock(&oslock);
    NSLog(@"线程1 解锁成功");
    NSLog(@"--------------------------------------------------------");
});

//线程2
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"线程2 准备上锁");
    OSSpinLockLock(&oslock);
    NSLog(@"线程2");
    OSSpinLockUnlock(&oslock);
    NSLog(@"线程2 解锁成功");
});
```

# os_unfair_lock 互斥锁
os_unfair_lock用于取代不安全的OSSpinLock ，从iOS10开始才支持 从底层调用看，等待os_unfair_lock锁的线程会处于休眠状态，并非忙等.
os_unfair_lock并不是一种自旋锁，在加锁状态下，等待锁的线程会处于休眠状态，不占用CPU资源
```
#import <os/lock.h>
// 初始化
os_unfair_lock lock = OS_UNFAIR_LOCK_INIT;
//尝试加锁(如果不需要等待，就直接加锁，返回true。如果需要等待，就不加锁，返回false)
BOOL res = os_unfair_lock_trylock(&lock);
//加锁
os_unfair_lock_lock(&lock);
//解锁
os_unfair_lock_unlock(&lock);
```
```
- (void)unfairLock {
    
    __block os_unfair_lock lock = OS_UNFAIR_LOCK_INIT;
    //线程1
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"线程1 准备上锁");
        os_unfair_lock_lock(&lock);
        NSLog(@"线程1 start");
        sleep(4);
        NSLog(@"线程1 finish");
        os_unfair_lock_unlock(&lock);
        NSLog(@"线程1 解锁成功");
        NSLog(@"--------------------------------------------------------");
    });

    //线程2
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"线程2 准备上锁");
        os_unfair_lock_lock(&lock);
        NSLog(@"线程2 start");
        sleep(4);
        NSLog(@"线程2 finish");
        os_unfair_lock_unlock(&lock);
        NSLog(@"线程2 解锁成功");
    });
    NSLog(@"end");
}
```
记得加`__block`


## 自旋锁和互斥锁的比较
当预计线程等待锁的时间很短，或者加锁的代码（临界区）经常被调用，但竞争情况很少发生，再或者CPU资源不紧张，拥有多核处理器的时候使用自旋锁比较合适。
而当预计线程等待锁的时间较长，CPU是单核处理器，或者临界区有IO操作，或者临界区代码复杂或者循环量大，临界区竞争非常激烈的时候使用互斥锁比较合适
互斥锁会使得线程阻塞，阻塞的过程又分两个阶段，第一阶段是会先空转，可以理解成跑一个 while 循环，不断地去申请加锁，在空转一定时间之后，线程会进入 waiting 状态，此时线程就不占用CPU资源了，等锁可用的时候，这个线程会立即被唤醒

# pthread_mutex
pthread_mutex是属于pthread api中的，mutex属于互斥锁。在加锁状态下，等待锁的线程会处于休眠状态，不会占用CPU的资源。

mutex初始化的时候需要传入一个锁的属性（int pthread_mutex_init(pthread_mutex_t * __restrict,const pthread_mutexattr_t * _Nullable __restrict);），如果传NULL就是默认状态PTHREAD_MUTEX_DEFAULT

初始化mutex之后再需要加锁的时候调用pthread_mutex_lock(),解锁的时候调用pthread_mutex_unlock();
另外pthread_mutex中有一个销毁锁的方法int pthread_mutex_destroy(pthread_mutex_t *);在不需要锁的时候通常需要调用一下，将mutex锁的地址作为参数传入


```
- (void)groupLock {
    static pthread_mutex_t plock;
    pthread_mutex_init(&plock, NULL);//传NULL就是默认状态PTHREAD_MUTEX_DEFAULT
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    for (int i = 1; i < 6; i ++) {
        dispatch_async(globalQueue, ^{
            NSLog(@"Thread %d start",i);
            pthread_mutex_lock(&plock);
            NSLog(@"Thread %d start excute",i);
            int sleepTime =  arc4random() % 5;
            NSLog(@"Thread %d sleep %d s",i,sleepTime);
            sleep(sleepTime);
            NSLog(@"Thread %d  excute finish",i);
            pthread_mutex_unlock(&plock);
            NSLog(@"Thread %d finish",i);
        });
    }
    NSLog(@" waiting... ");
}
```
加锁后只能有一个线程访问该对象,后面的线程需要排队,并且lock和unlock是对应出现的,同一线程多次lock是不允许的,如果对同一个线程多次lock会导致被死锁。

## pthread_mutex(recursive)  pthread递归锁

在开发中时常遇到递归调用的情况，如果在一个函数中进行了加锁和解锁操作，然后在解锁之前递归。那么递归的时候线程会发现已经加锁了，会一直在等待锁被释放。这样递归就没法继续往下进行，锁也永远不会被释放，就造成了死锁的现象
为了解决这个问题，pthread_mutex的属性中提供了将pthread_mutex变为递归锁的属性。’
递归锁就是同一条线程可以对一把锁进行重复加锁，而不同线程却不可以。这样每一次递归都会加一次锁，所以互不冲突，当递归结束之后会从后往前以此解锁。不同线程的时候，递归锁会判断这条线程正在等待的锁与加锁的不是一条线程，所以不会进行加锁，而是在等待锁被释放。
创建递归锁的时候需要初始化一个pthread_mutexattr_t属性：

```
pthread_mutexattr_t attr;
pthread_mutexattr_init(&attr);
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
pthread_mutex_init(&_pthreadMutex, &attr);
pthread_mutexattr_destroy(&attr);
```

```
- (void)groupLock {
    static pthread_mutex_t plock;
    pthread_mutex_init(&plock, NULL);
    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
    pthread_mutex_init(&plock, &attr);
    pthread_mutexattr_destroy(&attr);
    
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
//    dispatch_async(globalQueue, ^{
//        static void (^RecusiveBlock)(int);
//        RecusiveBlock = ^(int value) {
//            pthread_mutex_lock(&plock);
//            if (value > 0) {
//                NSLog(@"value = (%d)",value);
//                RecusiveBlock(value - 1);
//            }
//            pthread_mutex_unlock(&plock);
//        };
//        RecusiveBlock(5);
//    });
    for (int i = 1; i < 6; i ++) {
        dispatch_async(globalQueue, ^{
            NSLog(@"Thread %d start",i);
            pthread_mutex_lock(&plock);
            NSLog(@"Thread %d start excute",i);
            int sleepTime =  arc4random() % 5;
            NSLog(@"Thread %d sleep %d s",i,sleepTime);
            sleep(sleepTime);
            pthread_mutex_lock(&plock);
            sleep(sleepTime);
            NSLog(@"Thread %d  excute finish",i);
            pthread_mutex_unlock(&plock);
            pthread_mutex_unlock(&plock);
            NSLog(@"Thread %d finish",i);
        });
    }
    NSLog(@" waiting... ");
}
```
递归锁允许同一个线程在未释放其拥有的锁时反复对该锁进行加锁操作.但是lock和unlock的数目一定要一直，不然同样会引起死锁。
pthread_mutex_t结束之后要进行销毁
```
- (void)dealloc{
    pthread_mutex_destroy(&_mutex);
}
```

## pthread_mutex 条件锁
当两条线程在操作同一资源，但是一条线程的执行，需要依赖另一条线程的执行结果的时候，由于默认多线程的访问时间和顺序是不固定的，所以不容易实现。pthread_mutex提供了执行条件的api，使用pthread_cond_init()初始化一个条件。
在需要等待的地方使用pthread_cond_wait();等待信号的到来，此时线程会进入休眠状态并且放开mutex锁，等待信号到来的时候会被唤醒并且对mutex加锁。信号发送使用pthread_cond_signal()来告诉等待的线程，自己的线程处理完了，依赖的可以开始执行了，等待的线程就会往下继续执行。也可以使用pthread_cond_broadcast()进行广播，告诉所有等待的该条件的线程。条件也是需要销毁的，使用pthread_cond_destroy()销毁条件。
```
{
    NSMutableArray *array = [[NSMutableArray alloc] init];
    static pthread_mutex_t plock;
    pthread_mutex_init(&plock, NULL);
    
    static pthread_cond_t cond;
    pthread_cond_init(&cond, NULL);
    //线程1
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"线程1 准备上锁");
        pthread_mutex_lock(&plock);
        NSLog(@"线程1 start");
        for (int i = 0; i < 10; i ++) {
            [array addObject:@(i).stringValue];
        }
        NSLog(@"线程1 finish");
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&plock);
        NSLog(@"线程1 解锁成功");
    });

    //线程2
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"线程2 准备上锁");
        pthread_mutex_lock(&plock);
        pthread_cond_wait(&cond, &plock);
        NSLog(@"线程2 start");
        NSString *last = [array lastObject];
        NSLog(@"last string is:%@",last);
        sleep(2);
        NSLog(@"线程2 finish");
        pthread_mutex_unlock(&plock);
        NSLog(@"线程2 解锁成功");
    });
    NSLog(@" waiting... ");
}
- (void)dealloc{
    pthread_cond_destroy(&_cond);
    pthread_mutex_destroy(&_mutex);
}
```
条件也是需要销毁的，使用pthread_cond_destroy()销毁条件。
# NSLock
NSLock和NSRecursiveLock是对pthread_mutex普通锁和递归锁的OC封装。更加面向对象,使用也比较简单。它使用了NSLocking协议来生命加锁和解锁的方法。由于上面已经对pthread_mutex进行了简单的介绍，NSLock和NSRecursiveLock的api都是OC的也比较简单。这里不再赘述，只是说明有这样一种实现线程同步的方法。
```Objective-C
- (void)OCLock {
    NSMutableArray *array = [[NSMutableArray alloc] init];
    NSLock *lock = [[NSLock alloc] init];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //lockBeforeDate会在指定的时间之前加锁，所以已经使用过[_lock lock]了.下面相当于在当前时间之前上锁了。
        [lock lockBeforeDate:[NSDate date]];
        NSLog(@"1需要线程同步的操作1 开始");
        sleep(10);
        NSLog(@"1需要线程同步的操作1 结束");
        [lock unlock];
    });
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        if ([lock tryLock]) {//尝试获取锁，如果获取不到返回NO，不会阻塞该线程
            NSLog(@"2锁可用的操作");
            [lock unlock];
        }else{
            NSLog(@"2锁不可用的操作");
        }
        NSDate *date = [[NSDate alloc] initWithTimeIntervalSinceNow:3];
        if ([lock lockBeforeDate:date]) {
            //尝试在未来的3s内获取锁，并阻塞该线程，如果3s内获取不到恢复线程, 返回NO,不会阻塞该线程
            NSLog(@"2没有超时，获得锁");
            [lock unlock];
        }else{
            NSLog(@"2超时，没有获得锁");
        }
    });    
    NSLog(@" waiting... ");
}
```
## tryLock
tryLock 并不会阻塞线程。[lock tryLock] 能加锁返回 YES，不能加锁返回 NO，然后都会执行后续代码
## lockBeforeDate
lockBeforeDate: 方法会在所指定 Date 之前尝试加锁，会阻塞线程，如果在指定时间之前都不能加锁，则返回 NO，指定时间之前能加锁，则返回 YES;
#  NSRecursiveLock递归锁
NSRecursiveLock递归锁可以被同一线程多次请求，但不会引起死锁。这主要是用在循环或者递归操作场景中
```
- (void)useNSRecursiveLock{
    //如果使用_lock会招致死锁，因为被同一个线程多次调用。每次进入这个block时，都会去加一次锁，而从第二次开始，由于锁已经被使用了且没有解锁，所以它需要等待锁被解除，这样就导致了死锁，线程被阻塞住了。
//    _lock = [[NSLock alloc] init];
    NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //创建一个静态方法，block方法
        static void (^RecursiveMethod)(int);
        RecursiveMethod = ^(int value) {
            [lock lock];
            if (value > 0) {
                NSLog(@"value = %d", value);
                sleep(1);
                RecursiveMethod(value - 1);
            }
            [lock unlock];
        };
        RecursiveMethod(5);//方法内部判断来执行5次
    });
}
```
# NSConditionLock 条件锁
当我们在使用多线程的时候，只有一把会lock和unlock的锁就不能满足我们的需要了。因为普通的锁只关心锁与不锁，但是并不在乎什么时候才能开锁，而在处理资源共享场景的时候，多数情况下只有满足一定条件下才能打开这把锁。（Condition：美 [kən'dɪʃən] 条件）
NSConditionLock实现步骤：
NSConditionLock实现了NSLocking协议，一个线程会等待另一个线程unlock或者unlockWithCondition:之后再走lock或者lockWhenCondition:之后的代码。
锁定和解锁的调用可以随意组合，也就是说 lock、lockWhenCondition:与unlock、unlockWithCondition: 是可以按照自己的需求随意组合的。

划重点：
1、只有 condition 参数与初始化时候的 condition 相等，lock 才能正确进行加锁操作。
2、unlockWithCondition: 并不是当 condition 符合条件时才解锁，而是解锁之后，修改 condition 的值
```
 在线程 1 解锁成功之后，线程 2 并没有加锁成功，而是继续等了 1 秒之后线程 3 加锁成功，这是因为线程 2 的加锁条件不满足，初始化时候的 condition 参数为 0，而线程 2
 加锁条件是 condition 为 1，所以线程 2 加锁失败。
 lockWhenCondition 与 lock 方法类似，加锁失败会阻塞线程，所以线程 2 会被阻塞着。
 tryLockWhenCondition: 方法就算条件不满足，也会返回 NO，不会阻塞当前线程。
 lockWhenCondition:beforeDate:方法会在约定的时间内一直等待 condition 变为 2，并阻塞当前线程，直到超时后返回 NO。
 ```
```
- (void)nsconditionlock {
    NSConditionLock * cjlock = [[NSConditionLock alloc] initWithCondition:0];
    
    //1、线程 1 解锁成功
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [cjlock lock];
        NSLog(@"线程1加锁成功");
        sleep(1);//线程休眠一秒
        [cjlock unlock];
        NSLog(@"线程1解锁成功");
    });
    
    //2、初始化时候的 condition 参数为0，所以此处加锁失败，返回NO，此处线程阻塞。全部现成执行完毕后执行此处锁
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);//线程休眠一秒
        [cjlock lockWhenCondition:1];
        NSLog(@"线程2加锁成功");
        [cjlock unlock];
        NSLog(@"线程2解锁成功");
    });
    
    //3、tryLockWhenCondition尝试加锁  初始化时候的 condition 参数为0，所以此处加锁成功。方法就算条件不满足，也会返回 NO，不会阻塞当前线程。
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        
        if ([cjlock tryLockWhenCondition:0]) {
            NSLog(@"线程3加锁成功");
            sleep(2);
            /*
             A：成功案例
             这里会先解锁当前的锁，之后修改condition的值为100.在下一个condition为100的线程中会加解锁成功，如果下个锁中的condition等待的值不是100，那么就会导致加锁失败。
             */
            [cjlock unlockWithCondition:100];
            NSLog(@"线程3解锁成功");
            
            /*
             B：失败案例
             [cjlock unlockWithCondition:4];
             NSLog(@"线程3仍然会解锁成功，之后修改condition的值为4");
             */
            
        } else {
            NSLog(@"线程3尝试加锁失败");
        }
    });
    
    //4、lockWhenCondition:beforeDate:方法会在约定的时间内一直等待 condition 变为 2，并阻塞当前线程，直到超时后返回 NO。
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        if ([cjlock lockWhenCondition:100 beforeDate:[NSDate dateWithTimeIntervalSinceNow:10]]) {
            NSLog(@"线程100加锁成功");
            [cjlock unlockWithCondition:1];
            NSLog(@"线程100解锁成功");
        } else {
            NSLog(@"线程100尝试加锁失败");
        }
    });
}
```
NSConditionLock可以用来实现异步线程同步有序的执行某一段代码.
# NSCondition
NSCondition是对mutex和cond的封装，由于NSCondition也遵循了NSLocking协议，所以他也可以加锁和加锁。使用效果和pthread的cond一样，在等待的时候调用wait，发送信号调用singal
NSCondition 是一种特殊类型的锁，通过它可以实现不同线程的调度。A线程被某一个条件所阻塞，直到B线程满足该条件，从而发送信号给A线程使得A线程继续执行，例如：你可以开启一个线程下载图片，一个线程处理图片。这样的话，需要处理图片的线程由于没有图片会阻塞，当下载线程下载完成之后，则满足了需要处理图片的线程的需求，这样可以给定一个信号，让处理图片的线程恢复运行。
重点：
1、NSCondition 的对象实际上作为一个锁和一个线程检查器，锁上之后，其他线程也能继续上锁，之后根据条件决定是否继续运行线程，如果线程进入waiting状态，当其他线程中的该锁执行signal（信号）或者broadcast（广播）时，线程被唤醒，继续运行该线程之后的方法。。
2、NSCondition 可以手动控制现成的挂起和唤醒，可以利用这个特性设置依赖。
特别提醒：
signal只是唤醒单个线程，broadcast唤醒所有的线程。
broadcast ：广播 { 英 ['brɔːdkɑːst] 美 ['brɔdkæst]}
```
- (void)nsCondition {
    NSCondition * cjcondition = [NSCondition new];
    /*
     在加上锁之后，调用条件对象的 wait 或 waitUntilDate: 方法来阻塞线程，直到条件对象发出唤醒信号或者超时之后，再进行之后的操作。
     */
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [cjcondition lock];
        NSLog(@"线程1线程加锁----NSTreat：%@",[NSThread currentThread]);
        [cjcondition wait];
        NSLog(@"线程1线程唤醒");
        [cjcondition unlock];
        NSLog(@"线程1线程解锁");
    });
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [cjcondition lock];
        NSLog(@"线程2线程加锁----NSTreat：%@",[NSThread currentThread]);
        if ([cjcondition waitUntilDate:[NSDate dateWithTimeIntervalSinceNow:5]]) {
            NSLog(@"线程2线程唤醒");
            [cjcondition unlock];
            NSLog(@"线程2线程解锁");
        }
    });

    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        sleep(3);
        /*
1、休眠时间如果超过了线程中条件锁等待的时间，那么所有的线程都不会被唤醒。不管是哪一个线程中设置的时间，都不能超时，否则就会返回NO，全部不执行！切记切记！
2、一次只能唤醒一个线程，要调用多次才可以唤醒多个线程，如下调用两次，将休眠的两个线程解锁。
3、唤醒的顺序为线程添加的顺序。
*/
        
        [cjcondition signal];
        [cjcondition signal];

        //一次性全部唤醒
        //[cjcondition broadcast];
    });
}
```

# dispatch_semaphore
dispatch_semaphore_create(1): 传入>= 0 ,如果= 0 ,阻塞线程并等待timeout,时间到后会执行其后的语句
dispatch_semaphore_wait(signal,overTime): 可以理解为lock ,  使得signal -1,overTime表示等待时间可以为DISPATCH_TIME_FOREVER，一直等待
dispatch_semaphore_signal(signal)  可以理解为unlock, 使得signal +1
```
- (void)groupLock {
    dispatch_semaphore_t signal = dispatch_semaphore_create(1);
    dispatch_time_t overtime = dispatch_time(DISPATCH_TIME_NOW, 3.0f * NSEC_PER_SEC);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"Thread 1 start");
        dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
        NSLog(@"Thread 1");
        sleep(5);
        dispatch_semaphore_signal(signal);
        NSLog(@"Thread 1 finish");
    });
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"Thread 2 start");
        dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
        NSLog(@"Thread 2");
        sleep(3);
        dispatch_semaphore_signal(signal);
        NSLog(@"Thread 2 finish");
    });
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"Thread 3 start");
        dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
        NSLog(@"Thread 3");
        sleep(4);
        dispatch_semaphore_signal(signal);
        NSLog(@"Thread 3 finish");
    });
    NSLog(@" waiting... ");
}
```
I.函数异步执行，无序执行
II.waiting... 最先执行,dispatch_semaphore_t 信号量不会阻塞当前进程;

```
- (void)groupLock {
    for (int i = 0; i<3; ++i) {
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        dispatch_queue_t globeQ = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_async(globeQ, ^{
            [NSThread sleepForTimeInterval:3];
            NSLog(@"%d--over",i+1);
            dispatch_semaphore_signal(semaphore);
        });
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"%d--through",i+1);
    }
    NSLog(@" waiting... ");
}
- (void)groupLock {
    for (int i = 0; i<3; ++i) {
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        // dispatch_queue_t globeQ = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        dispatch_queue_t concurrentQ = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(concurrentQ, ^{
            dispatch_async(dispatch_get_global_queue(0, 0), ^{
                [NSThread sleepForTimeInterval:3];
                NSLog(@"%d--over",i+1);
                dispatch_semaphore_signal(semaphore);
            });
        });
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"%d--through",i+1);
    }
    NSLog(@" waiting... ");
}
```
上面执行是同步操作，当前线程等待信号量才能继续走for循环，所以切勿在主线程中使用dispatch_semaphore
可以得出：
I.异步操作,同步 并且 顺序执行;
II.waiting... 最后执行,dispatch_semaphore_t 信号量会阻塞当前进程;

## semaphore + dispatch_group
```
- (void)groupLock {
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_semaphore_t signal = dispatch_semaphore_create(1);
    for (int i = 1; i < 6; i ++) {
        dispatch_group_async(group, globalQueue, ^{
            NSLog(@"Thread %d start",i);
            dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
            NSLog(@"Thread %d start excute",i);
            int sleepTime =  arc4random() % 5;
            NSLog(@"Thread %d sleep %d s",i,sleepTime);
            sleep(sleepTime);
            NSLog(@"Thread %d  excute finish",i);
            dispatch_semaphore_signal(signal);
            NSLog(@"Thread %d finish",i);
        });
    }
    dispatch_group_notify(group, globalQueue, ^{
        NSLog(@"All finished");
    });
    NSLog(@" waiting... ");
}
```
从上面可以得出:
I:异步操作,异步并且无序执行;
II:dispatch_group 执行全部完成之后,会执行 dispatch_group_notify里面的block;
III:waiting...立即执行,dispatch_group_t不会阻塞当前线程;

当dispatch_group_async的block里面执行的是异步任务，如果还是使用上面的方法你会发现异步任务还没跑完就已经进入到了dispatch_group_notify里面的block
```
- (void)groupLock {
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_queue_t concurrentQ = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_semaphore_t signal = dispatch_semaphore_create(1);
    for (int i = 1; i < 6; i ++) {
        dispatch_group_async(group, concurrentQ, ^{
            dispatch_async(globalQueue, ^{
                NSLog(@"Thread %d start",i);
                dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
                NSLog(@"Thread %d start excute",i);
                int sleepTime =  arc4random() % 5;
                NSLog(@"Thread %d sleep %d s",i,sleepTime);
                sleep(sleepTime);
                NSLog(@"Thread %d  excute finish",i);
                dispatch_semaphore_signal(signal);
                NSLog(@"Thread %d finish",i);
            });

        });
    }
    dispatch_group_notify(group, globalQueue, ^{
        NSLog(@"All finished");
    });
    NSLog(@" waiting... ");
}
```
dispatch_group_t执行的是异步操作,这时用dispatch_group_enter和dispatch_group_leave就可以解决这个问题
```
{
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_queue_t concurrentQ = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_semaphore_t signal = dispatch_semaphore_create(1);
    for (int i = 1; i < 6; i ++) {
        dispatch_group_enter(group);
        dispatch_async(globalQueue, ^{
            NSLog(@"Thread %d start",i);
            dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
            NSLog(@"Thread %d start excute",i);
            int sleepTime =  arc4random() % 5;
            NSLog(@"sleep %d s",i);
            sleep(sleepTime);
            NSLog(@"Thread %d  excute finish",i);
            dispatch_semaphore_signal(signal);
            NSLog(@"Thread %d finish",i);
            dispatch_group_leave(group);
        });
    }
    dispatch_group_notify(group, globalQueue, ^{
        NSLog(@"All finished");
    });
    NSLog(@" waiting... ");
}
```
这里使用semaphore只是确保一次只执行一个线程.
一定要注意不要再主线程中使用信号量dispatch_semaphore_t,除非你有阻塞主线程的需求.一定要将操作放在异步线程.

# @synchronized
## synchronized的原理
synchronized关键字编译后，会在同步代码块的前后加上montor_enter和montor_exit两个指令。

## synchronized的使用方式
```
@synchronized(OC对象，如果用self，但是要注意可能导致死锁){
    //加锁的代码
}
```
synchronized是使用的递归mutex来做同步， synchronized里面传入的obj对象可以理解为互斥信号量
当我们通过@synchronized对一段代码加锁，信号量传入obj1的时候，其他线程如果使用的同样的信号量Obj1，那么就需要等待上一个线程执行完之后再执行，所以如果是对不同代码加锁，请使用不同的信号量
## synchronized的原理
@synchronized(nil)不起任何作用
@synchronized结构在工作时为传入的对象分配了一个递归锁recursive_mutex_t.递归锁在被同一线程重复获取时不会产生死锁.你可以在这找到一个它工作原理的精巧案例.和NSRecursiveLock类似.
[objc-sync 源码](https://opensource.apple.com/source/objc4/objc4-646/runtime/objc-sync.mm)

 struct SyncData,包含了object 就是我们传入的对象和一个关联的recursive_mutex_t, 它就是那个跟object关联在一起的锁.每个SyncData也包含一个指向另一个SyncData对象的指针,叫nextData.所以可以把每个SyncData结构体看做是链表中的一个元素.每个SyncData还包含一个threadCount,这个syncData对象中的锁会被一些线程使用或等待,threadCount就是此时这些线程的数量.syncData结构体会被缓存,threadCount= 0 代表这个syncData实例可以被复用.

  Struct SyncList的定义 把SyncData当做是链表中的节点.每个SyncList结构体都有个指向SyncData节点链表头部的指针,也有一个用于防止多个线程对此列表做并发修改的锁.
  sDataLists的声明是一个SyncList结构体数组,大小16.通过哈希算法将传入对象映射到数组上的一个下标.
当调用ovjc_sync_enter(obj)时,用obj内存地址的哈希值查找合适的SyncData,将其上锁.当调用objc_sync_exit(obj),查找合适的SyncData将其解锁
总结:
    1.当调用synchronzied的每个对象,runtime都会为其分配一个递归锁并存储在哈希表中
    2.如果在synchronzied内部对象被释放或为nil,会执行类似objc_sync_nil的空方法
    3.不要想synchronzied block 传入nil , 如果为nil ,则将会从代码中线程安全

Swift 中它已经 (或者是暂时) 不存在了。其实 @synchronized 在幕后做的事情是调用了 objc_sync 中的 objc_sync_enter 和 objc_sync_exit 方法，并且加入了一些异常判断。因此，在 Swift 中，如果我们忽略掉那些异常的话，我们想要 lock 一个变量的话，可以这样写：

```
func synchronized(lock: AnyObject, closure: () -> ()) {
    objc_sync_enter(lock)
    closure()
    objc_sync_exit(lock)
}
然后在Xcode中选择菜单Product->Perform Action->Assemble “xxx.m”，就得到了如下的汇编代码
```
@synchronized(self)会大大降低代码效率，因为所有的同步块（ @synchronized(self) ）都会彼此抢夺同一个锁。要是有多个属性这么写，每个属性的同步块（ @synchronized(self) ）都要等其他所有的同步块执行完毕之后才能执行，这并不是我们想要的结果，我们只想要每个属性各自独立的同步。

还有，不得不说，按上面这么做，虽然可以在一定程度上提供“线程安全”，但却无法保证访问该对象时是绝对线程安全的。事实上，上面的写法，就是atomic，也就是原子性属性xcode自动生成的代码，这种方法，在访问属性时，必定可以从中得到有效值，然而如果在一个线程上多次调用getter方法，每次得到的结果却未必相同，在两次读操作之间，其他线程可能会写入新的属性值。

直接将self传入@synchronized当中，这是种很粗糙的使用方式，容易导致死锁的出现
比如：
//class A@synchronized (self) { [_sharedLock lock]; NSLog(@"code in class A"); [_sharedLock unlock];}
//class B[_sharedLock lock];@synchronized (objectA) { NSLog(@"code in class B");}[_sharedLock unlock];

原因是因为self很可能会被外部对象访问，被用作key来生成一锁，类似上述代码中的@synchronized (objectA)。两个公共锁交替使用的场景就容易出现死锁。
所以正确的做法是传入一个类内部维护的NSObject对象，而且这个对象是对外不可见的。

# 串行队列
使用GCD的串行队列实现线程同步，原理是因为串行队列必须一个接着一个执行，只有在执行完上一个任务的情况下，下一个任务才会继续执行。
使用dispatch_queue_create("queue", DISPATCH_QUEUE_SERIAL);创建一条串行队列，将多线程任务都放到这条串行队列当中执行

# atomic
在OC中定义属性通常会指定属性的原子性也就是使用nonatomic关键字定义非原子性的属性，而其默认为atomic原子性
atomic用于保证属性setter、getter的原子性操作，相当于在getter和setter内部加了线程同步的锁，但是它并不能保证使用属性的过程是线程安全的。

假设有一个 atomic 的属性 "name"，如果线程 A 调[self setName:@"A"]，线程 B 调[self setName:@"B"]，线程 C 调[self name]，那么所有这些不同线程上的操作都将依次顺序执行——也就是说，如果一个线程正在执行 getter/setter，其他线程就得等待。因此，属性 name 是读/写安全的。

但是，如果有另一个线程 D 同时在调[name release]，那可能就会crash，因为 release 不受 getter/setter 操作的限制。也就是说，这个属性只能说是读/写安全的，但并不是线程安全的，因为别的线程还能进行读写之外的其他操作。线程安全需要开发者自己来保证。

如果 name 属性是 nonatomic 的，那么上面例子里的所有线程 A、B、C、D 都可以同时执行，可能导致无法预料的结果。如果是 atomic 的，那么 A、B、C 会串行，而 D 还是并行的

```
//@property(nonatomic, retain) UITextField *userName;
//系统生成的代码如下：

- (UITextField *) userName {
    return userName;
}

- (void) setUserName:(UITextField *)userName_ {
    [userName_ retain];
    [userName release];
    userName = userName_;
}
```
而 atomic 版本的要复杂一些 
```
//@property(retain) UITextField *userName;
//系统生成的代码如下：

- (UITextField *) userName {
    UITextField *retval = nil;
    @synchronized(self) {
        retval = [[userName retain] autorelease];
    }
    return retval;
}

- (void) setUserName:(UITextField *)userName_ {
    @synchronized(self) {
      [userName release];
      userName = [userName_ retain];
    }
}
```

# pthread_rwlock_t
当我们多项城操作一个文件的时候，如果同时进行读写的话，会造成读的内容不完全等问题。所以我们经常会在多线程读写文件的时候，实现多读单写的方案。即在同一时间可以有多条线程在读取文件内容，但是只能有一条线程执行写文件的操作。
使用pthread_rwlock_t的时候，需要调用pthread_rwlock_init()进行初始化。然后在读的时候调用pthread_rwlock_rdlock()对读操作进行加锁。在写的时候调用pthread_rwlock_wrlock()对读进行加锁。使用pthread_rwlock_unlock()进行解锁。在用不到锁的时候使用pthread_rwlock_destroy()对锁进行销毁。

# dispatch_barrier_async & dispatch_barrier_sync

共同点：

1、等待在它前面插入队列的任务先执行完

2、等待他们自己的任务执行完再执行后面的任务

不同点：

1、dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们

2、dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。


```
- (void)barrierTest {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("my.concurrent.queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(concurrentQueue, ^(){
        sleep(2);
        NSLog(@"dispatch-1");
    });
    NSLog(@"1");
    dispatch_async(concurrentQueue, ^(){
        sleep(1);
        NSLog(@"dispatch-2");
    });
    NSLog(@"2");
    dispatch_barrier_sync(concurrentQueue, ^(){
        sleep(5);
        NSLog(@"dispatch-barrier");
    });
//    dispatch_barrier_async(concurrentQueue, ^(){
//        sleep(5);
//        NSLog(@"dispatch-barrier");
//    });
    NSLog(@"barrier");
    dispatch_async(concurrentQueue, ^(){
        sleep(2);
        NSLog(@"dispatch-3");
    });
    NSLog(@"2");
    dispatch_async(concurrentQueue, ^(){
        sleep(1);
        NSLog(@"dispatch-4");
    });
    NSLog(@"4");
}
```


