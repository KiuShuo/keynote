#### Timer

#### Timer的几种创建方法 

```
// 列举几个特殊的，其他的都差不多

// 创建一个Timer，但需要手动添加到RunLoop中才能生效。
public /*not inherited*/ init(timeInterval ti: TimeInterval, invocation: NSInvocation, repeats yesOrNo: Bool)

// 含scheduled的会默认将生成的Timer添加到当前的RunLoop中。
open class func scheduledTimer(timeInterval ti: TimeInterval, invocation: NSInvocation, repeats yesOrNo: Bool) -> Timer

// - parameter:  block  The execution body of the timer; the timer itself is passed as the parameter to this block when executed to aid in avoiding cyclical references
// iOS10之后的方法 avoiding cyclical references，可以避免循环引用
@available(iOS 10.0, *)
public /*not inherited*/ init(timeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Swift.Void)

// fireDate   The time at which the timer should first fire.  可以设置第一次启动的时间。
public init(fireAt date: Date, interval ti: TimeInterval, target t: Any, selector s: Selector, userInfo ui: Any?, repeats rep: Bool)
```

#### Timer与UIScroolView同时使用的时候的注意点

```
...
    func creatScrollView() {
        let scrollView = UIScrollView(frame: CGRect(x: 10, y: 100, width: UIScreen.width - 20, height: 100))
        scrollView.backgroundColor = UIColor.green
        let contentView = UIView(frame: CGRect(x: 0, y: 0, width: UIScreen.width - 20, height: 400))
        contentView.backgroundColor = UIColor.red
        scrollView.contentSize.height = 400
        scrollView.addSubview(contentView)
        view.addSubview(scrollView)
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        creatTimer()
    }
    
    func creatTimer() {
        if #available(iOS 10.0, *) {
            Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
                print("something")
            }
        }
    }
...

```
当Timer执行过程中滑动UIScrollView会发现Timer停止。这是因为此时的Timer运行在主线程的对应RunLoop的defaultMode上，而ScrollView在滑动时RunLoop会被切换到UITrackingRunLoopMode，不再接受其他事件。

解决方法： 
 
1、将Timer添加到RunLoop的commonModes中: 

```
let timer = Timer.init(fire: Date(), interval: 1.0, repeats: true, block: { [weak self] _ in
    // 事件处理
})
RunLoop.current.add(timer, forMode: .commonModes)
``` 

2、将Timer事件定义在另外一个线程中:

```
DispatchQueue.global().async {
    let timer = Timer.init(fire: Date(), interval: 1.0, repeats: true, block: { [weak self] _ in
        DispatchQueue.main.async {
            // 事件处理
        }
    })
    let runLoop = RunLoop.current
    runLoop.add(timer, forMode: .defaultRunLoopMode)
    runLoop.run()
}
```

#### 在UIViewController中使用Timer时的内存问题

参见内存管理中关于Timer的介绍。

#### NSTimer的实时性
```
...
    DispatchQueue.global().async {
        let timer = Timer.init(fire: Date(), interval: 1.0, repeats: true, block: { [weak self] _ in
            DispatchQueue.main.async {
                print(currentDateString())
            }
        })
    // perform(selector: with: afterDelay:) 的内部是一个Timer，所以在子线程中使用时也需要主动的执行RunLoop的run方法。
    self.perform(#selector(self.doSomeThing), with: self, afterDelay: 2.0)
        
    RunLoop.current.add(timer, forMode: .defaultRunLoopMode)
    RunLoop.current.run()
    }

...
func currentDateString() {
    let format = DateFormatter()
    format.locale = Locale(identifier: "zh_CN")
    format.dateFormat = "hh:mm:ss"
    let dateString = format.string(from: Date())
    print("dateString = \(dateString)")
}

func doSomeThing() {
    // 做一些耗时操作
    sleep(3)
}
...
// 打印结果为：  
05:05:16
05:05:17
05:05:18

05:05:21
05:05:22
```
如上，timer后的耗时操作导致缺少了05:05:19、05:05:20。

一个 Timer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。


