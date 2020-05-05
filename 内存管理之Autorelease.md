##### 本文主要讲解了一下几个问题

autorelease的工作原理

-----

参考资料：  
[黑幕背后的Autorelease](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)

#### autorelease对象什么时候释放

在没有手动添加`@autoreleasepool{}`的情况下，`autorelease`对象会在当前runloop迭代结束的时候释放，而他能够释放的原因是系统在每个ruploop迭代中都加入了自动释池的Push和Pop操作。

#### autorelease的原理

autorelease是和线程一一对应的，一个线程里面只有一个autorelease，一个autoreleasePool开始的时候会执行`void *context = objc_autoreleasePoolPush();`代码。autorelease是由多个autoreleasePoolPage通过双向链表的形式组成的。autoreleasePoolPage用栈的形式存储了需要管理的对象。当一个runloop循环结束的时候，会调用`objc_autoreleasePoolPop(context)`方法通知autorelasePool释放autorelasePoolPage中的对象。

手动写的@autorelasepool { ... } 大括号中的对象会在大括号结束的时候释放。此时编译器会将代码转换为下面的形式：

```
void *context = objc_autoreleasePoolPush();
// {}中的代码
objc_autoreleasePoolPop(context);
```

即不需要等当前的runloop循环结束就立即执行了Pop操作，释放掉了{}中的对象。这就需要autoreleasePoolPage用来存储对象栈的时候对手动添加的autorelasepool开始新增对象时进行标记。

#### Autorelease的工作原理

常用的便利构造器方法，实际上就是将生成的对象添加到了自动释放池中。

```
@interface A : NSObject

+ (instancetype)object;

@end

@implementation A

+ (instancetype)object {
    A *a = [A alloc];
    [a autorelease]; // autorelease自动释放内存 会在一个RunLoop结束的时候释放

    /*
        调用autorelease的实例，会在超出其作用域时执行其release方法。
        autorelease的具体使用方法如下：
        // 1.生成并持有NSAutoreleasePool对象
        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
        id object0 = [[NSObject alloc] init];
        // 2.调用分配对象的autorelease方法
        [object0 autorelease];
        // 3.废弃NSAutoreleasePool对象 在废弃的同时，系统会调用object的release方法
        [pool drain];

        // 在Cocoa框架中，相当于程序主循环的NSRunLoop或者其他程序可运行的地方，对NSAutoreleasePool对象进行生成、持有和废弃处理。
        // 因此，绝大部分时候，开发者不需要自己写NSAutoreleasePool的相关代码。
        // autorelease实例方法的本质就是调用NSAutoreleasePool对象的addObjec类方法
        // autorelease的内部实现
        - (instancetype)autorelease {
           [NSAutoreleasePool addObject:self];
        }
        // NSAutoreleasePool的类方法 + (void)addObject:(NSObject *)obj 的内部实现：
        + (void)addObject:(NSObject *)obj {
           NSAutoreleasePool *currentAutoreleasePool = [获取当前的NSAutoreleasePool对象];
           [currentAutoreleasePool addObject: obj]; // 调用NSAutoreleasePool对象的实例方法
        }
        // NSAutoreleasePool的实例方法 - (void)addObject:(NSObject *)obj 的内部实现：
        - (void)addObject:(NSObject *)obj {
           [mArray addObject:obj]; // 用一个可变数组存储obj
        }
        // drain的内部实现：
        - (void)drain {
           [self release];
           [mArray release];
        }
        - (void)dealloc {
           for (obj in mArray) {
               [obj release];
           }
        }
     */

    /*
     不能调用NSAutoreleasePool对象的autorelease方法
     在Foundation框架中，无论调用哪个对象的autorelease方法，都是调用NSObject类的autorelease方法；
     但对于NSAutoreleasePool类，autorelease实例方法已经被该类重载，因此运行时会出错。
     */

    return a;
}

@end
```



