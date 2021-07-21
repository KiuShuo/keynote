### KVO (key value observer)

**参考资料：**  
* [透彻理解 KVO 观察者模式（附基于runtime实现代码）](https://www.jianshu.com/p/7ea7d551fc69)  
* [如何优雅地使用 KVO](https://draveness.me/kvocontroller)  
* [objc kvo简单探索](http://blog.sunnyxx.com/2014/03/09/objc_kvo_secret/)   
* [KVO进阶（二）](http://www.jianshu.com/p/a8809c1eaecc)  
* [根据IOS Foundation框架汇编反写的KVC,KVO实现](https://github.com/renjinkui2719/DIS_KVC_KVO)  
* [iOS开发 -- KVO的实现原理与具体应用](https://www.jianshu.com/p/e59bb8f59302)  
* [iOS KVO详解](https://imlifengfeng.github.io/article/498/)  

**demo**
见[PADemo](https://github.com/KiuShuo/PADemo)中的`NotificationCenterController.swift`

##### KVO的原理

简单的说，KVO之所能够监控到属性值的变化，是因为系统在背后做了如下几步操作：

1. 当一个`object`有观察者时，系统使用`runtime`动态创建这个`object`的类的子类；
2. 对于每个被观察的`property`，重写其`set`方法；
3. 在重写的set方法中调用`- willChangeValueForKey:`和`- didChangeValueForKey:`通知观察者；
4. 当一个`property`没有观察者时，删除重写的方法；
5. 当没有`observer`观察任何一个`property`时，删除动态创建的子类。



