---
layout: post
title: "关于NSOperation"
date: 2015-09-30 17:56:07 +0800
comments: true
categories: iOS

---


NSOperation 是 abstract class 抽象类，不能直接使用，我们使用的时候，一般要子类化，或者直接使用系统提供的两个子类：

* NSInvocationOperation
* NSBlockOperation

虽然GCD使用起来很方便，但使用不当也会暴露一些问题，比如对任务状态的不好控制等。这可能导致有些我们本来需要取消掉的任务，一直在运行，占用资源。这些情况下我们可以考虑通过 NSOperationQueue 加强对任务的控制。


---


#### NSInvocationOperation

NSInvocationOperation是抽象类NSOperation的实体子类，可以直接使用。如果不结合NSOperationQueue，单独通过调用 start方法使用，NSInvocationOperation是非并发的操作。例如：

```

    // 通过 start 直接调用，任务将在主线程中执行。
    NSInvocationOperation *invocationOp = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(doSomeThingWithInvocation:) object:nil];
    [invocationOp start];

	// 或者通过 addOperation 将任务加入队列，可在子线程中执行
	myOperationQueue = [[NSOperationQueue alloc] init];
	[myOperationQueue addOperation:invocationOp];

```
		
当然，在swift中,NSInvocation相关的类，都已经去掉了(不知道以后会不会加入)，也许NSInvocationOperation也是退出历史舞台的节奏。


---

#### NSBlockOperation

NSBlockOperation也是NSOperation的子类，只是跟NSInvocationOperation使用NSInvocation不同，NSBlockOperation使用block，使用起来会更方便。		

```

	myOperationQueue = [[NSOperationQueue alloc] init];	  	   
	NSBlockOperation *blockOp = [NSBlockOperation blockOperationWithBlock:^{
        //需要执行的任务
    }];
    
    [blockOp addExecutionBlock:^{
        //需要执行的任务
    }];
    
    // finished属性为YES后(完成或者被取消)将会执行
    __weak NSBlockOperation *weakOp = blockOp;
    [blockOp setCompletionBlock:^{
		
		if (!weakOp.isCancelled) {
		
			[[NSOperationQueue mainQueue] addOperationWithBlock:^{
	        	//更新主线程
    	    }];			
		}
    }];

	[myOperationQueue addOperation:blockOp];
```
	
上面代码，block相互之间都是并行异步的。也就是说可以同时执行多个block，不同block之间不会相互阻塞。不用等待依赖其他block完成。block内部是顺序执行的。 


通过添加依赖，来控制不同的NSBlockOperation之间是否完全并行执行。注意不要循环依赖导致死锁。	

```	

	NSBlockOperation *blockOp2 = [NSBlockOperation blockOperationWithBlock:^{
		//需要执行的任务
    }];
    
    //blockOp2 将等待至 blockOp 结束后，开始执行
    [blockOp2 addDependency:blockOp];
	[myOperationQueue addOperation:blockOp2];
	

```


---

#### 子类化自定义NSOperation

我们可以选择重写 `main` 方法，或者 `start` 方法，来实现。

* main：重写main方法，不需要手动管理太多状态，使用起来比较简单。

```

	- (void)main {
    	@autoreleasepool {

			//在这里，我们要频繁的检测 isCancelled 属性
			if(self.isCancelled){
				return;
			}
	    }
	}
 
```


* start：我们一般不会重写start，相对于重写main方法，处理上要更复杂些。要自己注意任务的一些状态比如isExecuting，isFinished，isConcurrent(isAsynchronous)，isReady。注意，当实现了start，就不会执行main。



* dependency：任务之间添加依赖。有时候某一个任务，需要等待其他任务的结束后才能开始。这时候，相对于使用GCD的dispatch_barrier_async更容易理解一些。

```

	//在op2的任务尚未执行时，添加依赖，op2将等待直到op完成
	[op2 addDependency:op];
	//移出op2的依赖
	[op2 removeDependency:op];

```

* Completion block：任务完成后的回调，要注意此时并没有回到主线程。

```

    __weak NSBlockOperation *weakOp = blockOp;
    [blockOp setCompletionBlock:^{
        
        if ( !weakOp.isCancelled ) {
            [[NSOperationQueue mainQueue] addOperationWithBlock:^{

            }];
        }
    }];
```

另外有几点需要注意：
		
* 已经完成的任务，不能被重新执行。	

* 已经添加到队列的任务，也不能被重复添加。	

* 当我们对一个任务发送`cancel`消息时，属性isCancelled会变为YES。	不过，对于自定义的NSOperation来说，要格外注意些。
	
	在发送cancel消息的时候，如果任务还未开始执行(isExecuting == NO)，就会被 finished 并 从队列中 remove 。
	
	如果任务正在执行中，这时候任务不会被强制结束。所以这时候，我们要在main方法中，手动检测(isCancelled == YES)并处理。		
* isFinished等于YES，不代表任务成功完成。也可能任务被取消。所以有时候，我们在Completion block回调中，需要判断任务是否被取消。

---


#### NSOperationQueue	
	
我们有两种不同的队列，主队列和自定义队列。主队列在主线程运行，自定义会开启子线程。

```

	NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];  
	NSOperationQueue *queue = [[NSOperationQueue alloc] init]; 
```
	
使用NSOperationQueue	的时候，系统默认会根据当前资源使用的情况，来为我们考虑最合适的线程数。当然我们也可以手动设置`maxConcurrentOperationCount`，限制使用的最大的线程数。更多的时候，其实我们不关心，交给系统处理就好。		

* name：一般我们使用队列的时候，都要给一个队列名字，方便我们维护debug。
	
* suspended：用来暂停和恢复队列中的所有任务。不能单独针对队列中的某一个任务。
	
```
	
	// Suspend
	[myOperationQueue setSuspended:YES];
	... 
	// Resume
	[myOperationQueue setSuspended:NO];	
```

* operationCount (readonly)：获取队列中还存在的正在执行的、等待执行的任务数量。注意，当任务一旦finished，就会从队列中remove。		
* cancelAllOperations：对队列所有的operations，发送`cancel`消息，将 isCancelled 属性设置为 YES 。如我们之前讨论过的，当operation 的 isExecuting 状态为 NO 时，operation 会被 finished 并从队列中 remove 。如果 isExecuting 状态为 YES ，那么我们自定义的operation，就要手动检测 isCancelled 属性。



