# iOS RunLoop理解
---

## 含义

`RunLoop`顾名思义是一个循环，是`iOS`中的一个机制。在`iOS`系统中打开一个`APP`中，之所以能够一直运行，也是因为有这样一个**死循环**的主线程一直在循环执行，从而保证我们的应用不退出。  


## 作用

一个线程通常只执行一个任务，执行完后线程便会退出。当我们需要一个线程随时都可以处理任务时，那么就需要一个机制，也就是`RunLoop`来保证线程不退出。  

线程与`RunLoop`之间的一一对应的，其关系是保存在一个全局的`Dictionary`中。值得注意的是，线程刚创建时并没有`RunLoop`，它的创建发生在第一次获取时，它的销毁发生在线程结束时。主线程是由系统自动创建的并且它的`RunLoop`默认是开启的。  


对于`RunLoop`而言，为了避免资源占用，系统会在线程没有任务处理时，让它进入休眠状态。当有任务发生时，立马被唤醒。  

## RunLoop Mode

`RunLoop`也是一个对象，线程中的事件消息执行时，需指定一个`Mode`,执行完时`RunLoop`变待在一旁休息，等待下一个事件的唤醒。  

系统为我们提供了五种类型的`Mode`,分别为：

* **NSDefaultRunLoopMode**  //默认Model  常用Timer事件 网络事件  
* **UITrackingRunLoopMode** //UI事件触发，比如触摸，点击等。此模式下禁止使用耗时曹组，负责导致UI卡顿
* **NSRunLoopCommonModes**  //通用模式  
* **UIInitializationRunLoopMode** //APP启动时进入的第一个Mode,完成后不再使用  
* **GSEventReceiveRunLoopMode**  //接收系统事件的内部Mode,不被使用  

举例说明：主线程创建一个`NSTimer`和一个`UIScrollView`，默认情况下，当用滑动ScrollView时，`Timer`事件将停止。离开屏幕时，又开始执行。
> 原因就在于：`Timer`事件默认是在`RunLoop`的`NSDefaultRunLoopMode`中，而`UIScrollView`滑动是在`UITrackingRunLoopMode`中，因此二者在同一个`RunLoop`的不同模式下，同时只会被触发一个。  

`Mode`里面处理的事件分别为：`Source`、`Observer`、`Timer`。


### Source
分为`Source0`和`Source1`。

* **Source0** //开发者定义的事件，基于`Source1`  
* **Source1** //系统内核事件  

### Timer
`Timer`事件创建包含了普通的`NSTimer`，这个需要我们将其添加到一个启动了的`RunLoop`中，如果是子线程，那么`RunLoop`则需要我们来管理。如果我们通过`GCD`来创建的`Timer`事件，`GCD`已经封装好了，以及多线程的处理操作。  

### Observer
`Observer`为`RunLoop`的状态观察者，所有状态可在`API`中查看。通过添加观察者，可以在我们的代码中来控制`RunLoop`的事件处理。  
> 添加观察者示例代码

```Objective-C
- (void)addRunLoopObserver{
  //获取当前RunLoop
  CFRunLoopRef runloop = CFRunLoopRefGetCurrent();

  //定义上下文
  CFRunLoopObserverContext context = {
    0,
    (__bridge void *)self,  //桥接
    &CFRetain,
    &CFRelease，
    NULL
  };

  //定义一个观察者
  static CFRunLoopObserverRef defaultObserver;

  //创建观察者
  defaultObserver = CFRunLoopObserverCreate(
    NULL,
    kCFRunLoopBeforeWaiting,   //RunLoop唤醒前
    YES,                       //重复执行
    NSIntegerMax - 999,
    &CallBack,                 //回调,这里来处理我们的逻辑代码
    &context
  );

  //添加
  CFRunLoopAddObserver(runloop, defaultObserver, kCFRunLoopDefaultMode);

  //因为这部分代码为C,是脱离OC的ARC机制，因此需要手动来管理内存
  CFRelease(defaultObserver);
}
```
