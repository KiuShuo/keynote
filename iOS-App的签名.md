参考资料：  
[iOS App 签名的原理](http://blog.cnbang.net/tech/3386/)  
[ipa重签名最直接的教程](https://www.jianshu.com/p/52deb349d5d1)  
[企业证书重签名发布APP（支持APNS）](https://www.jianshu.com/p/54b39c2d6d52)  
[细谈证书与Provisioning Profile](https://www.jianshu.com/p/abd3c55db48e)  

iOS开发和发布过程都需要用到开发证书、发布证书、开发用配置文件（Provisioning Profile）、发布用描述文件，以及推送证书、AppId等。这些证书、文件的作用是苹果为了控制App在iOS设备上的安装。  

开发者想要就一个项目安装到iOS设备上，需要以下几个步骤：  

1. 在开发者电脑上通过钥匙串生成证书请求文件(CertificateSiginingRequest.certSiginingRequest)；    
2. 在开发者账号上创建开发证书(需要使用到上一个步骤中生成的证书请求文件)；  
3. 在开发者账号上生成AppID(需要关联项目的Bundle id)；  
4. 在开发者账号上添加Device，将需要安装开发项目的设备UDID添加进来；  
5. 在开发者账号上生成开发配置文件（需要关联前几个步骤中生成的开发证书、AppID和设备列表）;  
6. 将证书安装到开发者的电脑上（安装好后会显示在钥匙串中），将配置文件安装到开发者的电脑上（安装好后会显示在路径~/Library/MobileDevice/Provisioning Profiles下，当然也可以使用如：iPhone 配置实用工具 的工具软件来查看当前设备上已经安装的描述文件）；   
7. 在Xcode中选择相应的证书和描述文件，打包到开发设备；  

通过上面的步骤就可以顺利的将开发中的项目安装到调试设备上，苹果之所以要搞的这么复杂，是为了控制iOS设备上安装的App，下面讲解一下怎么控制：  

1. 开发者电脑上生成证书请求文件的过程相当于生成了一对公私钥（简称为公钥L、私钥L），其中公钥就包含在证书请求文件中，或者可以理解证书请求文件就是一个公钥，私钥已经保存在设备上；  
2. 在开发者中心创建开发证书的过程实际上是苹果使用自己的私钥A对开发者上传过来的公钥L进行签名的过程（苹果使用私钥A对开发者提供的公钥L进行加密），最后生成的开发证书中包含了公钥L和签名；  
3. 生成描述文件的过程实际上是将开发证书、AppID、设备列表放进一个文件中并对这些信息进行签名，最后获取到的描述文件为：开发证书 + AppID + UDID列表 + 数字签名；  
4. 其中，开发者将开发证书安装到本地后就会在钥匙串中查看到，并且在钥匙串中还可以看到的是对应的私钥；开发者可以通过在钥匙串中导出p12的形式将这一对公私钥导出分发同一项目的其他开发者；  
5. 开发者在项目中选择完证书和描述文件进行项目编译和安装的过程，会使用本地的私钥L对项目进行签名，获取到包含签名信息的App文件（App + 使用私钥L获取的App签名）；然后将App文件和第3步中的描述文件打包进一个ipa类型的文件中，然后将ipa文件传到iOS设备上；   
6. iOS设备拿到ipa包以后会解压文件获取到App文件和描述文件（可以通过查看ipa包内容的方式发现里面有一个embedded.mobileprovision文件），系统会使用预装在设备中的公钥A验证描述文件的签名，保证描述文件没有被篡改，然后再验证证书中公钥L的签名，保证公钥L没有被篡改；然后再用公钥L验证App，保证App没有被篡改；通过后即可判断AppID中的Bundle id与App中的Bundle id是否一致；判断设备UDID列表中是否包含当前设备的UDID；全部验证通过之后就可以保证App是合法的、没有经过篡改的，就允许安装到iOS设备上了。  


![](https://raw.githubusercontent.com/KiuShuo/imageSource/master/%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%20app%E7%AD%BE%E5%90%8D%E5%8E%9F%E7%90%86.png)  

#### 安装的开发版本的App，如果开发证书revoke之后，之前安装的应用就无法启动了，这是为什么？  
iOS设备会将获取到的证书存储到本地的某一个地方，然后定期到苹果服务器上查询当前证书是否仍然有效，如果证书已经无效（比方说开发证书被revoke）了，那就会做一个标记，下次打开应用的时候就会直接闪退，也就是应用将打不开了。  

#### 已经发布到AppStore的App，如果发布证书无效后，App依旧可以下载和打开，这是为什么？  
如果去下载一个 AppStore 的安装包，会发现它里面是没有 embedded.mobileprovision 文件的，也就是它安装和启动的流程是不依赖这个文件，验证流程也就跟上述开发环境的App不一样了。  

据猜测，因为上传到 AppStore 的包苹果会重新对内容加密，原来的本地私钥签名就没有用了，需要重新签名，从 AppStore 下载的包苹果也并不打算控制它的有效期，不需要内置一个 embedded.mobileprovision 去做校验，直接在苹果用后台的私钥重新签名，iOS 安装时用本地公钥验证 App 签名就可以了。  

那为什么发布 AppStore 的包还是要跟开发版一样搞各种证书和 Provisioning Profile？猜测因为苹果想做统一管理，Provisioning Profile 里包含一些权限控制，AppID 的检验等，苹果不想在上传 AppStore 包时重新用另一种协议做一遍这些验证，就不如统一把这部分放在 Provisioning Profile 里，上传 AppStore 时只要用同样的流程验证这个 Provisioning Profile 是否合法就可以了。

所以 App 上传到 AppStore 后，就跟你的 证书 / Provisioning Profile 都没有关系了，无论他们是否过期或被废除，都不会影响 AppStore 上的安装包。

#### 推送证书的作用/推送证书revoke后，从App Store下载安装的App就收不到推送消息了，这是为什么？    
我们推送需要通过苹果的推送服务器，苹果的推送服务器中安装有推送证书中包含的是我们生成推送证书公钥，开发者导出的推送证书p12文件中含有私钥，当App Server向APNs Server进行信息推送时，会使用p12证书里面的私钥对信息加密，然后APNs Server会使用公钥对内容验签。APNs Server会定期去查询App的推送证书是否有效，如果查询到无效了就不会再使用该证书进行验签，若果此时还没有创建新的推送证书，推送功能就会失效。所以当推送证书过期或者revoke后，要及时生成新的证书，并替换App Server推送证书，苹果APNs Server会自动更新使用新证书。  

#### ipa包里面的代码签名文件和描述文件
解压ipa包发现里面有一个代码签名文件`_CodeSignature/CodeResources`，里面是各种资源文件、.nib文件、.storyboard文件、.bundle文件、.framework文件、`embedded.mobileprovision`文件等只要是能在`_CodeSignature`同级文件夹下看到的所有文件的hash值和签名(除了可执行文件)。  

解压包里面还有一个`embedded.mobileprovision`文件，该文件实际上就是打包时选择的Provisioning Profile文件，只是文件名被系统统一改成了`embedded.mobileprovision`(可以通过选中点击空格查看)。该文件是经过苹果私钥A签名的，签名结果存储在`_CodeSignature/CodeResources`文件中。   

#### 蒲公英、fir等第三方机构是怎么使用自己的企业证书给ipa包重签名的？    

#### RSA加密算法  
公钥加密私钥解密 / 私钥签名公钥验签 使用的就是RSA非对称加密算法。
