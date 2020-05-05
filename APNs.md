## APNs

**参考资料：**  
[国内90%以上的 iOS 开发者，对 APNs 的认识都是错的](http://www.cocoachina.com/ios/20160426/16013.html)  
这不是一篇讲怎么制作证书以及推送流程的文章。  
如果想要了解详细的实现流程，推荐[梁杰_numbbbbb](http://www.jianshu.com/users/f00634cb1580/latest_articles)的这篇[Swift 通知推送新手指南](http://www.jianshu.com/p/1c42080d7d63)文章。

**近两年变化历程**

2014年6月份 WWDC 搭载 iOS8 及以上系统的 iOS 设备，能够接收的最大 playload 大小提升到2KB。低于 iOS8 的设备以及 OS X 设备维持256字节。 
 
==2015年12月17日起，发布 “基于 HTTP/2 的全新 APNs 协议”，iOS 系统以及 OS X 系统，统一将最大 playload 大小提升到4KB。==

旧版本的 APNs 是基于二进制的，新版本的 APNs 是基于 HTTP 2.0 的。

****

**1.关于推送长度的限制**  

新 APNs 支持 iOS6 等全版本推送内容达4096字节，旧 APNs 是14年6月之前只支持256字节，在此之后支持 iOS8 以上2048字节。  

**2.关于推送是否成功的判断**  

**基于二进制的APNs旧协议下：APNs 成功失败全靠猜！**  
在旧的协议下，如果服务器响应成功的话，你将不会收到任何回应，但是如果服务器响应失败（例如，使用了一个非法的 Push token），服务器将返回了一个错误编码，并关闭这个socket。最重要的是，你必须重新发送使用这个无效 token 以后发送的所有推送。因此，你可能一直不能确定你的推送是否成功的被 APNs 服务器接收。  

**基于 HTTP/2 的全新协议下：APNs 成功失败都会收到Response。**  
APNs推送成功时，Response 将返回状态码200，远程通知是否发送成功再也不用靠猜了！  
APNs推送失败时，Response 将返回 JSON 格式的 Error 信息。  

其中最大的变化就是基于了 HTTP/2 协议，采用了长连接设计，并提供 “HTTP/2 PING ” 心跳包功能检测、维持当前 APNs 连接，解决了老 APNs 无法维持连接的问题。

而且新增的状态码特性，也解决了这个问题：无法获知消息是否成功地推送系统投递到了 APNs 上。==**理论上，可以保证消息是100%投递到了APNs的**==，因为你可以准确的知道哪条消息到达了APNs，哪些没到。重发特定失败消息成为可能。

____

## 两种推送证书的选择 

    // Dev
    Apple Push Notification service SSL (Sandbox)  
    // Production
    Apple Push Notification service SSL (Sandbox & Production)

后台在实现往苹果服务器推送消息的时候需要根据实际环境选择正确的证书，方能使对应设备上的App手到推送信息。对于两种证书的选择有如下结论：  

**结论：**   
两种证书名称后都跟了一个小括号，Sandbox对应开发，Production对应发布，Dev推送证书只能在Debug下的测试包中使用；<a>Production推送证书在Debug和AdHoc以及Release下的安装包中都能使用。</a>这样看来发布的推送证书比较强大。


配合证书的选择，还要选好对应的推送服务地址：  

|          		| 服务器地址| 证书类型  |
|:------------:|:-------------:|:-----:|
| 开发状态(Debug包)       | gateway.sandbox.push.apple.com 2195 |开发证书或者发布证书|
| 发布状态(AdHoc、Release包)       | gateway.push.apple.com 2195      |必须发布证书|

**验证方式：**  
Github 上面有位大神分享了他的推送工具[NWPusher](https://github.com/noodlewerk/NWPusher?spm=5176.100239.blogcont4803.8.59EeIh)，大大减少了开发人员的工作量。  
当然，类似的小工具在AppStore上有很多，只是很多小工具都需要直接选择生成的cer证书，而这款小工具可以直接使用到出的p12证书。

使用Pusher进行验证：

<div align="center">
<img src=https://raw.githubusercontent.com/KiuShuo/imageSource/master/APNs/APNs00.png width=50%>

<img src=https://raw.githubusercontent.com/KiuShuo/imageSource/master/APNs/APNs01.jpeg width=50%>
</div>

----

### DeviceToken  
deviceToken的组成：设备UDID和app的bundleID混编而成。  
UDID: 设备的唯一设备识别符  

某一用户对应的deviceToken发生变化的几种情况：  
1.当app的bundleID发生变化的时候；  
2.当用户在新设备上登陆的后（设备UDID发生改变，例如设备越狱）；  
3.当用户更新操作系统的时候；  
3.iOS9以及其之后的系统，当用户在同一设备上删除应用并重新安装后；  

重新卸载安装后deviceToken的变化：  
iOS8 不会改变。    
562264ee 3ba1843d fc05a8e1 418327ae bd0b21e9 aa78d65a b14901f7 ff334ad9
562264ee 3ba1843d fc05a8e1 418327ae bd0b21e9 aa78d65a b14901f7 ff334ad9
562264ee 3ba1843d fc05a8e1 418327ae bd0b21e9 aa78d65a b14901f7 ff334ad9

iOS9 发生改变后之前的deviceToken失效。  
f598bc0b 28dba8f2 e38b542c 200fd519 bad11e78 e43e5242 b834d086 01160936
a81e8074 0c14703d 1ec91038 01171d26 8847a6fc ccb91c2c 5a3a2d28 1242f3fa

iOS10 删除重装前后的device token   
06F85E3168D3C0651D0FAE009A013503D8703CF41BE66FB9E3E117CB96A943CA
EB1A4D57FEE9148B478E23059400A75FDFE82DAB3B6A665463D658468AB97F7B

----

### 常见问题

做完推送证书以后会影响到已经做完的开发发布描述文件，具体表现为：描述文件的状态变为黄色的Invalid，这是因为描述文件中有包含是否有推送功能，当做推送证书的时候需要选择对应的应用id，所以做完推送证书之后就表明对应id的应用有了推送功能，所以之前没有勾选推送功能的测试发布描述文件就会失效，没关系，只要重新生成一个新的描述文件就行了；

iOS APNS远程推送

为避免iOS少推或者重复推送消息，请考虑一下几种情况：

首先判断给哪个设备上的哪个应用推送消息的唯一标志：deviceToken（会发生改变），iOS设备只能被动的接受来自服务器的消息，不能拦截。

1.请确定推送消息是否与用户相关?

2.考虑下面的几种情况下消息的推送情况:

当用户仅安装应用而没有登陆的时候，是否发送推送消息？

当用户登陆的时候，考虑一下两种情况：

同一用户 -> 对应多台设备（设备唯一标志UUID不会发生改变，出刷机外等极少数情况下）-> 每台设备对应的devickToken会发生改变
给哪台设备上的应用推送消息？

同一设备 -> 对应多个用户
给哪个用户推送消息？



