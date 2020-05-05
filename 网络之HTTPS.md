### HTTP

参考资料：  
《图解HTTP》    
[图解SSL/TLS协议](http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)   
[iOS 中 HTTPS 证书验证浅析](https://mp.weixin.qq.com/s/-fLLTtip509K6pNOTkflPQ)  

#### HTTP的缺点：  
1. 通信使用明文（不加密），内容可能会被窃听；  
2. 不验证通信双方的身份，因此有可能遭遇伪装；  
3. 无法证明报文的完整性，所以有可能已遭篡改。  


#### HTTP + 加密 + 认证（证书） + 完整性保护 = HTTPS

SSL: Secure Socket Layer 安全套接字层  
TLS: Transport Layer Security 传输层安全     

HTTPS是身披SSL的HTTP。在采用了SSL后，HTTP就有了HTTPS加密、证书和完整性保护这些功能。
HTTPS使用SSL/TLS协议，TLS是在SSL3.0的基础上开发的协议，所以有时会统一称该协议为SSL。  

SSL采用的是公开密钥加密的一种加密方式。 

公开密钥加密：  
公钥加密，私钥解密。  

共享密钥加密：   
加密和解密公用一个密钥的加密方式。  
  
HTTPS采用混合加密机制  
公开密钥加密相较于共享密钥加密更为复杂，所以处理速度较慢，若在通信时使用则会影响通信效率。所以在交换密钥阶段使用公开密钥加密方式，之后建立通信交换报文阶段则使用共享密钥加密方式。  

#### HTTPS通信过程  
步骤1: 客户端client发送Client Hello报文开始SSL通信，报文中包含客户端支持的SSL版本、加密组件列表（所使用的加密算法及密钥长度）。  

步骤2: 服务器可进行SSL通信时，会以Server Hello报文作为应答，和客户端一样，报文中会包含SSL版本和加密组件。服务器中的加密组件内容是从客户端的加密组件内容中过滤出来的。  
步骤3: 之后服务器发送Certificate报文，报文中包含公开密钥证书。  
步骤4: 最后通过Server Hello Done报文通知客户端，最初部分的SSL握手协商结束。  

步骤5: SSL第一次握手结束后，客户端以Client Key Exchange报文作为回应。报文中包含通信加密中使用的一种被称为Pre-master secret的随机密码串。该报文已用步骤3中的公开密钥进行了加密。  
步骤6: 接着客户端会发送Change Cipher Spec报文。该报文会提示服务器，在报文之后的通信会采用Pre-master secret进行密钥加密。  
步骤7: 客户端发送Finished报文。该报文包含通信至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器能否解密该报文作为判断标准。  

步骤8: 服务器同样发送Change Clipher Spec报文。  
步骤9: 服务器同样发送Finished报文。  

步骤10: 服务器和客户端的Finished报文交换完以后，SSL连接就算建立完成。当然，通信会收到SSL保护，从此处开始进行应用层协议通信即发送HTTP请求。  
步骤11: 应用层协议通信，即发送HTTP请求。  

步骤12: 最后由客户端断开连接。断开连接时，发送close_notify报文。这步之后在发送TCP PIN报文来关闭与TCP的通信。  

<div align="center">
<img src=https://raw.githubusercontent.com/KiuShuo/imageSource/master/HTTP/HTTPS%E9%80%9A%E4%BF%A1.png width=50%>
</div>

