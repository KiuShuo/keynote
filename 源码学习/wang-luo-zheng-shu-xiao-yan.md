[AFNetWorking源码之AFSecurityPolicy](https://huang303513.github.io/2017/04/26/AFNetWorking%E6%BA%90%E7%A0%81%E4%B9%8BAFSecurityPolicy.html)  


获取证书以及二进制对应的base64编码：
* 获取公钥证书：
* 直接从官网拖出来到本地；
* 通过终端命令获取证书文件对应的base64编码：

```
cat fileName | base64
```

。。。怎么将base64在转回证书文件：先将base64转换为data 在存储到本地

下面主要介绍AFSecurityPolicy中的几种验证类型以及每种类型对应的验证方法：

### AFSecurityPolicy内部的核心原理

AFSecurityPolicy用来进行证书的安全行验证，定了了三种验证类型：

```
typedef NS_ENUM(NSUInteger, AFSSLPinningMode) {
    AFSSLPinningModeNone, 
    AFSSLPinningModePublicKey,
    AFSSLPinningModeCertificate,
};
```

下面过程中用到的变量代表的含义：
SecTrustRef serverTrust 这是一个待验证的信任对象，包含待验证的证书（服务器返回的证书）和支持的验证方法。

##### AFSSLPinningModeNone 默认验证

```
// 如果允许不信任的证书（就是未在系统信任列表中的的证书机构签发的证书）或者是信任的证书颁发机构签名的证书，即可验证通过
if (self.SSLPinningMode == AFSSLPinningModeNone) {
    return self.allowInvalidCertificates || AFServerTrustIsValid(serverTrust);
} 
```

##### AFSSLPinningModePublicKey 验证公钥

```
// 首先在设置证书列表的时候（就是存储在项目中的证书列表）会遍历证书获取到每一个证书中的公钥，然后将公钥存储到集合中。
- (void)setPinnedCertificates:(NSSet *)pinnedCertificates {
    _pinnedCertificates = pinnedCertificates;

    if (self.pinnedCertificates) {
        NSMutableSet *mutablePinnedPublicKeys = [NSMutableSet setWithCapacity:[self.pinnedCertificates count]];
        // 迭代每一个证书
        for (NSData *certificate in self.pinnedCertificates) {
            // 获取证书对应的公钥
            id publicKey = AFPublicKeyForCertificate(certificate);
            if (!publicKey) {
                continue;
            }
            [mutablePinnedPublicKeys addObject:publicKey];
        }
        // 赋值给对应的属性
        self.pinnedPublicKeys = [NSSet setWithSet:mutablePinnedPublicKeys];
    } else {
        self.pinnedPublicKeys = nil;
    }
}
...

// 获取服务器返回证书中包含的公钥集合，然后通过遍历循环与上文中存储的证书公钥进行对比，只要有一个匹配上就验证通过
       case AFSSLPinningModePublicKey: {
            NSUInteger trustedPublicKeyCount = 0;
            // 根据serverTrust对象和SecPolicyCreateBasicX509认证策略，获取对应的公钥集合
            NSArray *publicKeys = AFPublicKeyTrustChainForServerTrust(serverTrust);
            
            for (id trustChainPublicKey in publicKeys) {
                // 把获取的公钥和系统获取的默认公钥比较，如果相等，则通过认证
                for (id pinnedPublicKey in self.pinnedPublicKeys) {
                    if (AFSecKeyIsEqualToKey((__bridge SecKeyRef)trustChainPublicKey, (__bridge SecKeyRef)pinnedPublicKey)) {
                        trustedPublicKeyCount += 1;
                    }
                }
            }
            return trustedPublicKeyCount > 0;
        }

```

##### AFSSLPinningModeCertificate 验证证书

```
// 首先验证服务端返回的证书是否是系统信任的证书颁发机构签发的证书，如果是再判断服务端返回证书的证书链中是否有与本地证书集合中相同的，如果有则表示验证通过
        case AFSSLPinningModeCertificate: {//验证整个证书
            NSMutableArray *pinnedCertificates = [NSMutableArray array];
            // 根据指定证书获取，获取对应的证书对象
            for (NSData *certificateData in self.pinnedCertificates) {
                [pinnedCertificates addObject:(__bridge_transfer id)SecCertificateCreateWithData(NULL, (__bridge CFDataRef)certificateData)];
            }
            // 将pinnedCertificates设置成需要参与验证的Anchor Certificate（锚点证书，通过SecTrustSetAnchorCertificates设置了参与校验锚点证书之后，假如验证的数字证书是这个锚点证书的子节点，即验证的数字证书是由锚点证书对应CA或子CA签发的，或是该证书本身，则信任该证书），具体就是调用SecTrustEvaluate来验证。
            SecTrustSetAnchorCertificates(serverTrust, (__bridge CFArrayRef)pinnedCertificates);

            if (!AFServerTrustIsValid(serverTrust)) {
                return NO;
            }

            // obtain the chain after being validated, which *should* contain the pinned certificate in the last position (if it's the Root CA)
            // 获取serverTrust证书链。直到根证书。
            NSArray *serverCertificates = AFCertificateTrustChainForServerTrust(serverTrust);
            // 如果`pinnedCertificates`包含`serverTrust`对象对应的证书链的根证书。则返回true
            for (NSData *trustChainCertificate in [serverCertificates reverseObjectEnumerator]) {
                if ([self.pinnedCertificates containsObject:trustChainCertificate]) {
                    return YES;
                }
            }
            
            return NO;
        }
```


