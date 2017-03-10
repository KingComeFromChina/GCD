
##[GCD](http://www.jianshu.com/p/4ac227e763fa)
Grand Central Dispatch (GCD) 是 Apple 开发的一个多核编程的解决方法。该方法在 Mac OS X 10.6 雪豹中首次推出，并随后被引入到了 iOS4.0 中。GCD 是一个替代诸如 <code>NSThread</code>, <code>NSOperationQueue</code>, <code>NSInvocationOperation</code> 等技术的很高效和强大的技术。
GCD 和 block 的配合使用，可以方便地进行多线程编程。
###[Clone](https://github.com/KingComeFromChina/GCD.git)
`git clone https://github.com/KingComeFromChina/GCD.git`
###任务和队列
1.任务分为同步任务和异步任务，队列分为串行和并行
2.同步任务不会开线程，异步任务会开线程
```

- (void)sync_queue:(dispatch_queue_t)queue{

//同步任务
dispatch_sync(queue, ^{
NSLog(@"同步1 - %@",[NSThread currentThread]);
});

dispatch_sync(queue, ^{
NSLog(@"同步2 - %@",[NSThread currentThread]);
});

dispatch_sync(queue, ^{
NSLog(@"同步3 - %@",[NSThread currentThread]);
});

dispatch_sync(queue, ^{
NSLog(@"同步4 - %@",[NSThread currentThread]);
});
}

```

```
- (void)async_queue:(dispatch_queue_t)queue{

//异步任务
dispatch_async(queue, ^{
NSLog(@"异步1 - %@",[NSThread currentThread]);
});

dispatch_async(queue, ^{
NSLog(@"异步2 - %@",[NSThread currentThread]);
});

dispatch_async(queue, ^{
NSLog(@"异步3 - %@",[NSThread currentThread]);
});

dispatch_async(queue, ^{
NSLog(@"异步4 - %@",[NSThread currentThread]);
});
}

```
3.串行队列一个一个的执行，并行队列一起执行

####同步任务
`dispatch_sync(<#dispatch_queue_t  _Nonnull queue#>, <#^(void)block#>)`
####异步任务
`dispatch_async(<#dispatch_queue_t  _Nonnull queue#>, <#^(void)block#>)`
####主队列
`dispatch_queue_t mainQueue = dispatch_get_main_queue();`
####全局并发队列
`dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);`
####自定义串行队列
`dispatch_queue_t serialQueue = dispatch_queue_create("wanglei", NULL);`
####自定义并行队列
`dispatch_queue_t concurrentQueue = dispatch_queue_create("king", DISPATCH_QUEUE_CONCURRENT);`
#####同步主线程会卡死，造成死循环
`[self sync_queue:mainQueue];`
#####异步主线程不会开线程，顺序执行
`[self async_queue:mainQueue];`
#####同步全局并发不会开线程，顺序执行
`[self sync_queue:globalQueue];`
#####异步全局并发会开线程，乱序执行
`[self async_queue:globalQueue];`
#####同步串行不会开线程，顺序执行
`[self sync_queue:serialQueue];`
#####异步串行会开线程，顺序执行
`[self async_queue:serialQueue];`
#####同步并行不会开线程，顺序执行
`[self sync_queue:concurrentQueue];`
#####异步并行会开线程，乱序执行
`[self async_queue:concurrentQueue];`
###线程间通信
```
#pragma mark - 线程间通讯
- (void)threadCommunication{

//主队列
dispatch_queue_t mainQueue = dispatch_get_main_queue();
//全局并发队列
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_async(globalQueue, ^{

NSURL *url = [NSURL URLWithString:@""];
NSData *data = [[NSData alloc]initWithContentsOfURL:url];
NSLog(@"%@",[NSThread currentThread]);

dispatch_async(mainQueue, ^{

data;
//在这里刷新UI
NSLog(@"mainQueue -- %@",[NSThread currentThread]);
});
});
}

```
###GCD创建单例
```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
NSLog(@"只执行一次");
});
}

```
###GCD创建分组任务
```
- (void)dispatch_group{

// 创建一个分组
dispatch_group_t group = dispatch_group_create();

// 全局队列
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 第一个参数: 任务所在的分组
// 第二个参数: 任务所在的队列

dispatch_group_async(group, globalQueue, ^{
NSLog(@"分组任务1");
});

dispatch_group_async(group, globalQueue, ^{
NSLog(@"分组任务2");
});

// 当上面两个任务都完成以后，会执行这个方法，我们在这里处理我们的需求
dispatch_group_notify(group, globalQueue, ^{
NSLog(@"上面分组任务完成后，才会执行");
});
}

```
###GCD延迟操作
```
- (void)dosomethingByTime{

// 延迟加载函数
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(10 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
NSLog(@"延迟10s加载");
});

// 自定义并行队列
dispatch_queue_t concurrentQueue = dispatch_queue_create("wanglei", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(concurrentQueue, ^{
NSLog(@"dispatch_async - 1-%@",[NSThread currentThread]);
});

dispatch_async(concurrentQueue, ^{
NSLog(@"dispatch_async - 2-%@",[NSThread currentThread]);
});

// dispatch_barrier_async 使用于并行环境下
// 使用dispatch_barrier_async添加的任务会在之前的block全部运行完毕之后，才会继续执行。保证对非线程安全的对象进行正确的操作
// 运行完dispatch_barrier_async的block才会执行后面的任务
// dispatch_barrier_async所在的线程跟前一个任务是同一条线程
dispatch_barrier_async(concurrentQueue, ^{
NSLog(@"dispatch_barrier_async-%@",[NSThread currentThread]);
});

dispatch_barrier_async(concurrentQueue, ^{
NSLog(@"dispatch_barrier_async- 3 -%@",[NSThread currentThread]);
});

dispatch_barrier_async(concurrentQueue, ^{
NSLog(@"dispatch_barrier_async- 4 -%@",[NSThread currentThread]);
});

}

```
###GCD下载图片及合成
```
- (void)drawRectImage{

// 创建全局并发队列
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 异步下载
dispatch_async(globalQueue, ^{

// 下载第一张图片
NSURL *url1 = [NSURL URLWithString:@"http://pic6.huitu.com/res/20130116/84481_20130116142820494200_1.jpg"];
NSData *data1 = [NSData dataWithContentsOfURL:url1];
UIImage *image1 = [UIImage imageWithData:data1];

// 下载第二张图片
NSURL *url2 = [NSURL URLWithString:@"http://g.hiphotos.baidu.com/image/pic/item/c2cec3fdfc03924578c6cfe18394a4c27c1e25e8.jpg"];
NSData *data2 = [NSData dataWithContentsOfURL:url2];
UIImage *image2 = [UIImage imageWithData:data2];

// 合并图片
// 开启一个位图上下文
UIGraphicsBeginImageContextWithOptions(image1.size, NO, 0.0);

// 绘制第一张图片
CGFloat image1Width = image1.size.width;
CGFloat image1Height = image1.size.height;
[image1 drawInRect:CGRectMake(0, 0, image1Width, image1Height)];

// 绘制第二张图片
CGFloat image2Width = image2.size.width * 0.5;
CGFloat image2Height = image2.size.height * 0.5;
CGFloat image2Y = image1Height - image2Height;
[image2 drawInRect:CGRectMake(0, image2Y, image2Width, image2Height)];

// 得到上下文中的图片
UIImage *fullImage = UIGraphicsGetImageFromCurrentImageContext();

// 结束上下文
UIGraphicsEndImageContext();

// 回到主线程显示图片

dispatch_queue_t mainQueue = dispatch_get_main_queue();
dispatch_async(mainQueue, ^{
self.imageView.image = fullImage;
});
});
}

```
###GCD实现的验证码倒计时
```
- (void)btnClick{



[_btn setTitle:@"重发(60s)" forState:UIControlStateNormal];
__block int timeout=59; //倒计时时间
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,queue);
dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0); //每秒执行
dispatch_source_set_event_handler(_timer, ^{
if(timeout<=0){ //倒计时结束，关闭

dispatch_source_cancel(_timer);
dispatch_async(dispatch_get_main_queue(), ^{
//设置界面的按钮显示 根据自己需求设置
self.btn.userInteractionEnabled = YES;
[self.btn setTitle:@"获取验证码" forState:UIControlStateNormal];

});
}else{
int seconds = timeout % 60;
NSString *strTime = [NSString stringWithFormat:@"%.2d", seconds];
dispatch_async(dispatch_get_main_queue(), ^{
//设置界面的按钮显示 根据自己需求设置
[UIView beginAnimations:nil context:nil];
[UIView setAnimationDuration:1];
[self.btn setTitle:[NSString stringWithFormat:@"重发(%@秒)",strTime] forState:UIControlStateNormal];

[UIView commitAnimations];
self.btn.userInteractionEnabled = NO;

});
timeout--;
}
});

dispatch_resume(_timer);




}

```
![效果图](http://upload-images.jianshu.io/upload_images/3873966-76f9eb56f6adc6c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![效果图](http://upload-images.jianshu.io/upload_images/3873966-ae572a11922a691e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
