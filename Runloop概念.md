## RunLoop

###### 参考资料

[iOS刨根问底-深入理解RunLoop](http://www.cnblogs.com/kenshincui/p/6823841.html)  
[iOS夯实：RunLoop](https://github.com/100mango/zen/blob/master/%23iOS夯实：RunLoop/%23iOS夯实：RunLoop.md)  
[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)  
[Runloop笔记（一）](https://immanito.github.io/2017/01/06/Runloop笔记（一）/)  
[Runloop笔记（二）](https://immanito.github.io/2017/01/06/Runloop笔记（二）/)

#### 是什么

RunLoop 可以简单的理解为一个while循环。

```
func loop() {
    while event != quit {
        var event = nextEvent()
        process(event)
    }
}
```

在循环里等待事件，然后处理事件，而这个循环是基于线程的。通过RunLoop这样的机制，线程能够在没有事件处理的时候休息，有事情的时候运行，减轻CPU的压力。  
应用程序走执行操作的时候会不停的创建和配置线程，但这样做非常的消耗性能；所以就引入了RunLoop机制，用来实现线程长驻。

#### RunLoop与线程的关系

RunLoop是基于pthread进行管理的，pthread是基于C的跨平台多线程操作底层API。NSThread是基于pthread的封装，与RunLoop一一对应，但不是说有一个线程就一定有一个RunLoop，但有一个RunLoop一定会有一个与其对应的线程。

iOS中并未提供直接创建RunLoop的方法，需要通过下面的两个get属性获取：

```
open class var current: RunLoop { get }

@available(iOS 2.0, *)
open class var main: RunLoop { get }
```

其内部实现大概是这样的：

```
/// 全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问 loopsDic 时的锁
static CFSpinLock_t loopsLock;

/// 获取一个 pthread 对应的 RunLoop。
CFRunLoopRef _CFRunLoopGet(pthread_t thread) {
    OSSpinLockLock(&loopsLock);

    if (!loopsDic) {
        // 第一次进入时，初始化全局Dic，并先为主线程创建一个 RunLoop。
        loopsDic = CFDictionaryCreateMutable();
        CFRunLoopRef mainLoop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, pthread_main_thread_np(), mainLoop);
    }

    /// 直接从 Dictionary 里获取。
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread));

    if (!loop) {
        /// 取不到时，创建一个
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }

    OSSpinLockUnLock(&loopsLock);
    return loop;
}

CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
}

CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
}
```

从上面的代码可以看出，RunLoop与线程是一一对应的关系，保存在一个全局的字典里面。线程刚创建的时候没有RunLoop，如果不主动获取就一直不会有。RunLoop的创建发生在第一次获取时，RunLoop的销毁发生在线程结束时。只能在线程内部获取RunLoop，除了主线程对应的mainRunLoop。

#### RunLoopMode

RunLoop必须在一种特定的Mode下运行，每种RunLoop都可以包含多个Mode。  
iOS中提供的可用的Mode如下：

```
extension RunLoopMode {
    // 默认的RunLoopMode 进入iOS程序默认不做任何操作就处于该Mode下.
    // 此时滑动UIScrollView主线程切换RunLoop到trackingRunLoopMode，不再接收其他事件操作。
    public static let defaultRunLoopMode: RunLoopMode

    // commonModes不是一种特定的Mode，而是一组Mode组合，它默认包含了defaultRunLoopMode和trackingRunLoopMode.
    @available(iOS 2.0, *)
    public static let commonModes: RunLoopMode
}
```



