## CocoaPods理解

##### CocoaPods的作用  
1. 方便查找使用第三方库，也可以用来管理公司内部的二方库；
2. 能够自动配置编译选项；

##### pod install与pod update的区别
`pod install`可以保证podfile中所有第三方都能够按照podfile.lock中固定的版本进行安装；  
`pod update`会更新podfile中所使用的第三方，并且更新pofile.lock中规定的版本；  

在podfile中直接指定了第三方的版本号，在使用`pod update`的时候可以保证所安装的第三方就是指定版本的第三方，但是会更新第三方中依赖的其他库的版本，所以还是只有使用`pod install`才能保证所使用的第三方以及其依赖能够按照podfile.lock中锁定的版本号进行安装。

