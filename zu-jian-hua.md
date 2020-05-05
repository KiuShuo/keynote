### 组件化

参考资料：  
* [组件化架构漫谈](https://www.jianshu.com/p/67a6004f6930)  
* [MGJRouter](https://github.com/meili/MGJRouter)  
* [iOS 组件化 —— 路由设计思路分析](https://juejin.im/post/58b2aad6b123db0052cc9edd)
* [iOS 组件化方案探索](http://blog.cnbang.net/tech/3080/)  
* [iOS App 组件化开发实践](https://www.infoq.cn/article/ios-app-component-development-practice)  
* [iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)  

进行组件化开发后，每个组件可以作为一个单独的App独立运行，每个组件甚至可以采用不同的架构，例如分别使用MVC、MVVM、MVPCS等架构。


### 层次架构 + 组件化架构  
组件化架构在物理上是不分层的，但是在组件化的基础上，应该根据项目和业务涉及的层次架构，区分组件所处的层次和职责。  

业务层 -> 核心层 -> 基础层

在分层架构中，需要注意的是只能上层对下层依赖，下层不能对上层依赖，下层中不能包含上层业务逻辑。在项目中的公共资源和代码，应该将其下沉到下层中。  

在设计组件的时候应该遵循“高内聚，低耦合”的设计规范，组件的调用应该简单直接，减少调用方的其他处理。对于核心层和基础层的划分，可以以是否涉及到业务、是否涉及同级组件间通信、是否经常改动为参照点，如果符合这些条件则划分为核心层，不符合则划分到基础层。  

在集成业务层和核心层组件后，组件间的通信都是通过URLRouter进行的，项目中不允许直接依赖组件源码。而基础层组件则在集成后直接依赖，例如资源文件和配置文件，都是在主工程和组件中直接使用的。第三方库则是通过核心层的业务封装，封装后由URLRouter进行通信，但核心层也是直接依赖第三方库源码的。

### 组件间通信  
使用蘑菇街的[MGJRouter](https://github.com/meili/MGJRouter)进行组件间通信。下面是MGJRouter的简单分析：

MGJRouter的本质就是一个字典存储器，通过注册函数存储分解url以及对应的block，打开的时候再根据url分解获取到block和分解参数，然后执行block。

通过MGJRouter可以将事件block通过一个url链存储起来，然后在合适的时机在通过url(可以拼接上参数)执行block。

核心代码分为存储`registerUrl`和取出`openUrl`两部分
举例说明：

``` Objective-C

// register部分
NSString *registerUrl = @"jk://router/toDetail";
// 通过执行注册方法
// + (void)registerURLPattern:(NSString *)URLPattern toHandler:(MGJRouterHandler)handler
// 在routes中添加了新元素 routes += [jk: [router: [toDetail: [_: toHandel]]]]
[MGJRouter registerURLPattern:registerUrl toHandel: ^(NSDictionary *routerParameters){
    // routerParameters的内容为
    /*
     [
     MGJRouterParameterUserInfo: userInfo,
     MGJRouterParameterCompletion: completion,
     MGJRouterParameterURL: @"jk://router/toDetail?p1=1&p2=2",
     p1: 1,
     p2: 2
     ]
     */
}];

NSString *openUrl = @"jk://router/toDetail?p1=1&p2=2";

/*
 通过执行open方法
 + (void)openURL:(NSString *)URL withUserInfo:(NSDictionary *)userInfo completion:(void (^)(id result))completion;
 根据openUrl找到对应的handel, 并且执行handel
 userInfo 是传递给register block的额外参数；
 completion 是传递给register block的回调函数
 */

[MGJRouter openURL:openUrl withUserInfo:userInfo completion:^(id result){
    // 执行额外的操作
}];


```
MGJRouter在使用的时候有一个问题就是硬编码，当存在大量的组件间通信时，就容易出现url和参数不清晰的问题，我们可以通过使用一个plist文件来存储组件所对应的url和所需要的参数。  

MGJRouter中也提供了一个方法用来方便处理注册和open使用的url不同时共用一个宏定义管理的方法(如下)： 

```
/**
 *  调用此方法来拼接 urlpattern 和 parameters
 *
 *  #define MGJ_ROUTE_BEAUTY @"beauty/:id"
 *  [MGJRouter generateURLWithPattern:MGJ_ROUTE_BEAUTY, @[@13]];
 *  
 *
 *  @param pattern    url pattern 比如 @"beauty/:id"
 *  @param parameters 一个数组，数量要跟 pattern 里的变量一致
 *
 *  @return 返回生成的URL String @"beauty/13"
 */
+ (NSString *)generateURLWithPattern:(NSString *)pattern parameters:(NSArray *)parameters
```



#### 库
静态库与动态库的区别：  
静态库会在链接时被完整的复制到可执行文件中，有多次使用就会有多次冗余拷贝。  
使用静态库，编译完成之后库文件实际上就没有什么作用了，目标程序没有外部依赖，可以直接运行；缺点就是会使目标程序增大。  

动态库在编译的时候并不会被拷贝到目标程序中，目标程序只会存储指向动态库的引用，等程序运行时动态库才会被真正加载进来。  
系统的动态库不需要加载到目标程序中，自建的动态库可以在工程内多个库共享，因此可以减少目标程序的体积，但是，由于把静态链接做的事情放到了运行时来做，程序启动会变慢。  


使用cocoapods管理的三方库、二方库和直接将动态库或者静态库引入到工程里面有什么不同？  
为什么cocoapods中的动态库或者静态库不需要将头文件加入到public中而是在project里面就可以在工程中访问到？  


参考[手动配置.framework形式开发包](http://lbsyun.baidu.com/index.php?title=iossdk/guide/buildproject)的过程可以发现，手动导入一个静态库的过程异常麻烦，比方说：  
第二步中引入所需的系统动态库，很容易引用错误或者少引入；
第四步配置环境，手动在`TARGETS -> Build Settings -> Other Link Flags`中添加`-Objc`，这是因为百度地图中存在分类，而工程找不到。  

