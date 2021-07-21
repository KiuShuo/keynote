参考资料：  
[NSHipster -- WKWebView](http://nshipster.cn/wkwebkit/)   
[胡椒的笔记本 -- iOS与JavaScript交互系列](https://moxo.io/blog/)

## WKWebView

参考资料中的已经介绍了WKWebView与UIWebView的对比和WKWebView的基本使用、代理方法、与JavaScript的交互。这里就不再重复写了。

下面简单总结一下WKWebView与JavaScript的交互：

WKWebView与JavaScript的交互与UIWebView不同，他不依赖于JavaScriptCore，交互的方式封装到了WKWebView的内部。  

1. WKWebView调JS：

 ```
 open func evaluateJavaScript(_ javaScriptString: String, completionHandler: ((Any?, Error?) -> Swift.Void)? = nil)
 ```

 方法直接位于WKWebView中，一般在WKWebView加载完成后调用。如PADemo中的例子： 

 ```
 func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        // 加载完页面后调用里面的JS代码
        webView.evaluateJavaScript("sumValue(1, 2)") { (response, error) in
            let result = response as? Int
            print(result ?? -1)
            print(error ?? "success")
        }
    }
 ```
 
2. JS调用WkWebView：
    这个要间接一点，需用通过WKWebView中的configuration属性中的userContentController属性。
    
    ```
    // WKWebView
    @NSCopying open var configuration: WKWebViewConfiguration { get }
    // WKWebViewConfiguration
    open var userContentController: WKUserContentController
    ```
    
    通过add函数添加方法
    
    ```    
    wkWebView.configuration.userContentController.add(self, name: "addFuncName")
   ```
   
   // JS中的调用上面add的原生方法
   
   ```
	window.webkit.messageHandlers.addFuncName.postMessage({参数})
	```
   
   服从WKScriptMessageHandler协议的scriptMessageHandler实例可以在代理方法中收到来自JS对add过函数的通知(调用)
   
   ```
   extension KSWKWebViewController: WKScriptMessageHandler {
	func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
		witch message.name {
        case "addFuncName":
            let body = message.body
            print(body)
        default: ()
        }
	}
}
   ```
      
   通过上面的方法，WKWebView会对WKScriptMessageHandler实例强引用，所以需要在合适的时机将其remove掉。
   
   ```
    /*! @abstract Removes a script message handler.
     @param name The name of the message handler to remove.
     */
    open func removeScriptMessageHandler(forName name: String)
    ```
    具体可参考PADemo中KSWKWebViewController中的代码例子，这里就不再粘贴。  
    
3. WKUIDelegate提供原生控件显示网页方法的回调。
