##### 本文主要讲解了一下几个问题
1. @property的作用
2. OC中的strong、weak、assgin、copy
3. Swift 中的strong weak unowned
4. weak与assgin、weak与unowned的区别
5. weak的内部实现原理
6. swift中为什么不明确说明属性关键词/默认是什么

-----

参考资料：  
[《招聘一个靠谱的 iOS》—参考答案（上）](https://github.com/KiuShuo/iOSInterviewQuestions)
[WEAK 和 UNOWNED](http://swifter.tips/retain-cycle)
[iOS 开发之 Copy/MutableCopy](https://juejin.im/entry/57b15244a633bd00570955be)


### @property的本质是什么？@property后面有哪些关键词？
@property用来声明属性，他相当于 实例变量 + setter、getter方法。

@property后面的关键词有： 
1. 用来声明原子性atomic与非原子性nonatomic；
2. 用来设置读取特性的readwrite、readonly；
3. 用来表示内存特性的strong、weak、assign、copy；
4. 以及用来重新定义setter、getter方法名的如：@property(nonatomic, strong, getter=p_initBy, setter=setP_initBy:) NSString *initBy;

### Objective-C中的strong, weak, assign, copy
1. strong 表示指向并拥有该对象。其修饰的对象引用计数会增加 1。该对象只要引用计数不为 0 则不会被销毁。当然强行将其设为 nil 可以销毁它。
2. weak 表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁。
3. assign 主要用于修饰基本数据类型，如 NSInteger 和 CGFloat ，这些数值主要存在于栈上。
4. weak 一般用来修饰对象，assign 一般用来修饰基本数据类型。原因是 assign 修饰的对象被释放后，指针的地址依然存在，造成野指针，在堆上容易造成崩溃。而栈上的内存系统会自动处理，不会造成野指针。
5. copy 与 strong 类似。不同之处是 strong 的复制是多个指针指向同一个地址，而 copy 的复制每次会在内存中拷贝一份对象，指针指向不同地址。copy 一般用在修饰有可变对应类型的不可变对象上，如 NSString, NSArray, NSDictionary。

### Swift中的strong、weak、unowned
Swift的内存管理机制和Objective-C的ARC(Automatic Reference Counting)是一样的。  

1. strong 代表强引用，当一个对象被声明为strong时，表示父层级对该对象有一个强引用指向，引用计数会加1；     
2. weak 代表着弱引用。当对象被声明为 weak 时，父层级对此对象没有指向，该对象的引用计数不会增加1。它在对象释放后弱引用也随即消失。继续访问该对象，程序会得到 nil，不会崩溃；    
3. unowned 与弱引用本质上一样。唯一不同的是，对象在释放后，依然有一个无效的引用指向对象，它不是 Optional 也不指向 nil。如果继续访问该对象，程序就会崩溃。  

weak和unowned的区别：   
weak和unowned是为了解决strong带来的循环引用问题。  
当访问的对象可能已经被释放了，用weak;   
当访问的对象不可能被释放，用uowned;   
但实际上为了安全起见，都是用weak。 

### 什么时候使用weak？weak与assign、unowned的区别？
当会出现循环引用的时候，往往通过设置一端为weak来避免，比方说UITableView的delegate、datasource属性都是用weak修饰；已经进行过强引用没必要再进行强引用一次的时候，比方说通过xib/storyboard添加的控件关联到代码中的@IBOutlet属性。  

weak用来修饰对象类型，是弱引用，assign用来修饰基本数据类型；
weak表示一种非拥有关系(nonowning relationship)，为weak属性设置新值时，设置方法既不保留新值，也不释放旧值；但weak修饰的对象在被释放后，指针地址会被置为nil；如果在对象类型中使用assign修饰，当对象释放后assign类型的指针不会变为nil，就会导致野指针，再是访问就会导致崩溃。

```
@property (nonatomic, weak) id weakPoint;
@property (nonatomic, assign) id assignPoint;

- (void)setWeakPoint:(id)weakPoint {
    _weakPoint = weakPoint;
    // objc_setAssociatedObject(self, "weakPoint", weakPoint, OBJC_ASSOCIATION_ASSIGN);
    // [weakPoint cyl_runAtDealloc:^{
    //     _weakPoint = nil;
    // }];
}

- (void)setAssignPoint:(id)assignPoint {
    _assignPoint = assignPoint;
}

```

### 弱引用weak和无主引用unowned都可以解决循环强引用问题，二者有什么区别？

弱引用和无主引用允许循环引用中的一个实例引用另外一个实例，但不保持强引用。这样实例能够相互引用而不产生循环强引用。

因为弱引用不会保持所引用的实例，即使引用存在，实例也有可能被销毁。因此，ARC会在引用的实例被销毁后自动将其置为nil。并且因为弱引用可以允许它们的值在运行时被赋值为nil，所以它们会被定义为可选类型变量，而不是常量。 
**当 ARC 设置弱引用为nil时，属性观察不会被触发。**

无主引用通常都被期望拥有值。不过 ARC 无法在实例被销毁后将无主引用设为nil，因为非可选类型的变量不允许被赋值为nil。  
**使用无主引用，你必须确保引用始终指向一个未销毁的实例。如果你试图在实例被销毁后，访问该实例的无主引用，会触发运行时错误。**

例如：租客Person相对于公寓Apartment来说，由于公寓在某段时间内可能没有租客，所以租客的生命周期更短；持卡人Customer相对于信用卡CreditCard来说，由于信用卡必须被持卡人持有，所以持卡人的生命周期更长。

Person和Apartment的例子展示了两个属性的值都允许为nil，并会潜在的产生循环强引用。这种场景最适合用弱引用来解决。

Customer和CreditCard的例子展示了一个属性的值允许为nil，而另一个属性的值不允许为nil，这也可能会产生循环强引用。这种场景最适合通过无主引用来解决。

存在着第三种场景，在这种场景中，两个属性都必须有值，并且初始化完成后永远不会为nil。在这种场景中，需要一个类使用无主属性，而另外一个类使用隐式解析可选属性。

在闭包和捕获的实例总是互相引用并且总是同时销毁时，将闭包内的捕获定义为无主引用。

相反的，在被捕获的引用可能会变为nil时，将闭包内的捕获定义为弱引用。弱引用总是可选类型，并且当引用的实例被销毁后，弱引用的值会自动置为nil。这使我们可以在闭包体内检查它们是否存在。

**如果被捕获的引用绝对不会变为nil，应该用无主引用，而不是弱引用。**

#### runtime是如何实现weak变量自动设置为nil的？
runtime对注册的类会进行布局，对于weak变量会放入一个hash表中。用weak指向对象的内存地址作为key，当对象的引用计数为0的时候会进行dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 变量，从而设置为 nil。

##### @protocol 和 category 中如何使用 @property

### Swift中为什么不明确写出属性的内存属性关键词？Swift为什么不写nonatomic关键词？
Swift中大部分情况下不写属性关键词，是因为使用了默认的关键词。引用类型默认是用到 了strong，值类型默认是assign，weak仍需要开发者手动设置；Swift中所有的property都是nonatomic；读取属性通过var、let以及通过权限控制词private、fileprivate、internal、public、open加(set)来控制可写范围。


