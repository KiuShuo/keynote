##### 本文主要讲解了一下几个问题
1. block为什么要从栈区copy到堆区
2. NSString NSArray NSDictionary为什么要使用copy修饰
3. 深浅copy

-----

### 参考资料
[iOS进阶（一）block与property](https://www.jianshu.com/p/3c0dd2cbb509)

### 为什么要将block从栈区copy到堆区

[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW12)    
> You should specify copy as the property attribute, because a block needs to be copied to keep track of its captured state outside of the original scope. This isn’t something you need to worry about when using Automatic Reference Counting, as it will happen automatically, but it’s best practice for the property attribute to show the resultant behavior. For more information, see Blocks Programming Topics.

从官方文档中可以看出，block复制到堆区的主要目的是为了保持block的状态，延长其生命周期。因为block如果在栈上的话，其所属的变量作用域结束，该block就被释放掉，block中的__block变量也同时被释放掉。为了解决栈块在其作用域结束之后被释放掉的问题，我们就需要把block复制到堆中。  

详细分析参考：[iOS进阶（一）block与property](https://www.jianshu.com/p/3c0dd2cbb509)  

### block属性一定要加关键词使用copy吗？

实际上，block使用copy关键字是MRC遗留下来的代码习惯，{% em %}将block从栈区copy到堆区{% endem %}。在ARC中编译器会自动对block进行copy工作，无论是写了copy还是strong。只是写上copy可以让开发者更加明确的知道编译器会做这个copy操作。在Swift中一般都不会再写copy。

    
#### 通过NSString的创建方式进一步理解内存分区


### 为什么NSString、NSDictionary、NSArray要使用copy修饰符呢/深浅copy？

对于NSString、NSDictionary、NSArray等经常使用copy关键字，是因为它们有对应的可变类型：NSMutableString、NSMutableDictionary、NSMutableArray，它们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性时拷贝一份。

要搞清楚这个问题，我们先来弄明白深拷贝与浅拷贝的区别，以非集合类与集合类两种情况来进行说明下。  
先看非集合类的情况，代码如下：

```
NSString *name = @"xiaoMing";
NSString *newName = [name copy];
NSLog(@"name memory address: %p newName memory address: %p", name, newName);
```
运行之后，输出的信息如下：

```
name memory address: 0x10159f758 newName memory address: 0x10159f758
```
可以看出复制过后，内存地址是一样的，没有发生变化，这就是浅复制，只是把指针地址复制了一份。  
我们改下代码改成[name mutableCopy]，此时日志的输出信息如下：
```
name memory address: 0x101b72758 newName memory address: 0x608000263240
```
我们看到内存地址发生了变化，并且newName的内存地址的偏移量比name的内存地址要大许多，由此可见经过mutableCopy操作之后，复制到堆区了，这就是深复制了，深复制就是内容也就进行了拷贝。

上面的都是不可变对象，在看下可变对象的情况，代码如下：
```
NSMutableString *name = [[NSMutableString alloc] initWithString:@"xiaoMing"];
NSMutableString *newName = [name copy];
NSLog(@"name memory address: %p newName memory address: %p", name, newName);
```
运行之后日志输出信息如下：
```
name memory address: 0x600000076e40 newName memory address: 0x6000000295e0
```
从上面可以看出copy之后，内存地址不一样，且都存储在堆区了，这是深复制，内容也就进行拷贝。  
在把代码改成[name mutableCopy]，此时日志的输出信息如下：
```
name memory address: 0x600000077380 newName memory address: 0x6000000776c0
```
可以看出可变对象copy与mutableCopy的效果是一样的，都是深拷贝。

总结：对于非集合类对象的copy操作如下：
```
[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
```
采用同样的方法可以验证集合类对象的copy操作如下：
```
[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //单层深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
```