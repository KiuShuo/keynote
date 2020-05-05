参考资料：   
[透彻理解 NSNotificationCenter 通知（附实现代码）](https://www.jianshu.com/p/e3a38b21420c)  
[Notification与多线程](http://southpeak.github.io/2015/03/14/nsnotification-and-multithreading/)  
[NSNotificationCenter深入研究](https://blog.csdn.net/u014600626/article/details/73603557)

##### name object userInfo参数

```
open class NSNotification : NSObject, NSCopying, NSCoding {

    // A tag identifying the notification. 通知标识符
    open var name: NSNotification.Name { get }
	// An object that the poster wishes to send to observers. 
	// 发送者想要发给接受者的对象
    open var object: Any? { get }
	// Storage for values or objects related to this notification. 
	// 存储通知关联的值或者对象
    open var userInfo: [AnyHashable : Any]? { get }

    ...
}

open class NotificationCenter : NSObject {

    
    open class var `default`: NotificationCenter { get }

    /* object: 
    The object whose notifications the observer wants to receive; that is, 
    only notifications sent by this sender are delivered to the observer.
	If you pass nil, the notification center doesn’t use a notification’s 
	sender to decide whether to deliver it to the observer.
	观察者想要接收的对象，如果object不为nil，则只会接收发送者中与object相同的对象；如果object为空，就不会做这个判断，只要name相同就可以接收。
	*/
    open func addObserver(_ observer: Any, selector aSelector: Selector, name aName: NSNotification.Name?, object anObject: Any?)
	
	...
}

NotificationCenter.default.post(name: Notification.Name(rawValue: "test"), object: obj, userInfo: nil)

NotificationCenter.default.post(name: Notification.Name(rawValue: "test1"), object: nil, userInfo: nil)

// add时object为空，只要name相同的通知都可以收到
NotificationCenter.default.addObserver(self, selector: #selector(toDo(notification:)), name: Notification.Name(rawValue: "test"), object: nil)
// add时object不为空，name相同 + object相同的通知才可以收到  -> 可以收到test通知
NotificationCenter.default.addObserver(self, selector: #selector(toDo(notification:)), name: Notification.Name(rawValue: "test"), object: obj)
// add时object不为空，name相同 + object相同的通知才可以收到  -> 收不到test/test1通知
NotificationCenter.default.addObserver(self, selector: #selector(toDo(notification:)), name: Notification.Name(rawValue: "test1"), object: obj1)

```

##### Q1. NotificationCenter 添加的通知是否需要手动移除？
 
 结论：iOS8开始添加到UIViewController的通知，不需要手动移除，iOS9之后添加到NSObject的通知也不需要手动移除。
 
 至少在iOS8之后（因为没有iOS7的测试机），在viewController里面添加一个通知，当viewController销毁的时候系统会自动的执行removeObserver方法。
即 iOS8之后在viewController里面添加的通知，在viewController释放的时候如果没有手动执行removeObserver方法是没有问题的。
从iOS9开发，NSObject也可以像UIViewController一样，在delloc的时候自动的移除通知
 
 原因：
 注册观察者时，通知中心不会对观察者做retain操作；
 iOS9之前，是做了不安全引用 unsafe_unretained；iOS9之后，是做了弱引用 weak。
 
 不安全引用（unsafe reference）和弱引用 (weak reference) 类似，它并不会让被引用的对象保持存活，但是和弱引用不同的是，当被引用的对象释放的时，不安全引用并不会自动被置为 nil，这就意味着它变成了野指针，而对野指针发送消息会导致程序崩溃。
 
 因此，iOS9之前，观察者对象在释放之前必须从通知中心移除引用，否则通知中心就会给野指针所引用的对象发送消息，导致程序崩溃。
 iOS9之后，不移除没有问题。
  
##### Q2. 通知是同步的吗？子线程中发送的通知 方法是在哪个线程执行的？
 
 答：通知是同步的，在哪个线程发出的通知，就在哪个线程执行方法。
 
 NSNotificationCenter消息的接收线程是基于发送消息的线程的，也就是同步的。因此，有时候，你发送的消息可能不在主线程，而大家都知道操作UI必须在主线程，不然会出现不响应的情况。所以，在你收到消息通知的时候，注意选择你要执行的线程。
 
##### 内部实现

NotifictionCenter中主要有三个步骤：添加通知、发送通知、移除通知   

定义一个KSNotificationCenter单例类；  
定义一个KSNotificationInfoModel类 里面存储观察者添加通知时设置的参数 即添加通知时的数据模型类；  
定义一个KSNotification类用来存储name object userInfo 即发送通知时的数据模型类;  

// 添加通知  
KSNotificationCenter单例类中设置一个可变字典observerDic，其具体类型为`NSMutableDictionary <NSString *, NSArray <KSNotificationInfoModel *>*>*`;   
KSNotificationCenter单例类中提供类似于系统中的几个添加通知的方法，最终核心是将添加通知时设置的参数封装进KSNotificationInfoModel类里面，然后以name为key 以相同name的KSNotificationInfoModel数组为value，添加进字典observerDic中；  

// 发送通知  
当调用KSNotificationCenter单例类的发送通知方法时，根据name找到`NSArray  <KSNotificationInfoModel *>`，然后再依据是否有object来找到数组中存储的具体的infoModel，然后让infoModel中存储的observer执行infoModel中的SEL；  

// 移除通知  
ARC中的dealloc方法不能继承也不能通过Runtime交换，所以为了实现观察者释放时自动移除相应的通知，可以在为observer添加通知的时候给其动态的关联一个监听器属性KSNotificationMonitor，当observer释放的时候 监听器属性的dealloc方法就会执行，此时就可以调用KSNotificationCenter单例类中的移除通知的方法。  

 
