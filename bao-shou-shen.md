#### 参考资料  

[《iOS安装包瘦身指南》](http://www.zoomfeng.com/blog/ipa-size-thin.html)  
[iOS微信安装包瘦身](http://www.cocoachina.com/ios/20151211/14562.html)  
[iOS App Thinning](http://alicialy.github.io/2017/04/07/iOS-App-Thinning/)  

#### 检测工具
* [Xcode重复代码检测](https://jerrychu.github.io/2018/08/05/Xcode-cpd/)  
* [搜索项目不用的类](https://github.com/HSFGitHub/XcodeProjectArrangementTool)  
* [检查每个类占用空间大小工具](https://github.com/huanxsd/LinkMap)  

去掉对老机型的兼容:
在Xcode的Build Setting -> Valid Architectures中，去掉armv7、armv7s不再兼容iPhone5s之前的机型，能够大幅减少ipa包的大小。

#### 使用动态库framework的方式会加快编译速度吗/会增加安装包的大小吗
