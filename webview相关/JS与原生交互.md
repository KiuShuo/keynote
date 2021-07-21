### 原生调用JS:

1、UIWebView中通过调用下面的方法来调用JS方法

```
func stringByEvaluatingJavaScript(from script: String) -> String?
```

2、iOS7之后可以使用JavaScriptCore实现原生与JS的交互，其中原生调用JS：  
```
jsContext.evaluateScript("js代码")
```

3、WKWebView调用JS方法

```
open func evaluateJavaScript(_ javaScriptString: String, completionHandler: ((Any?, Error?) -> Swift.Void)? = nil)
```

### JS调用原生：

1、可以使用拦截重定向的方式

```
func chongdingxiangjiaohu(urlString: String) {
        let components = urlString.components(separatedBy: "://")
        if components.count > 1 {
            let funcName = components[1].components(separatedBy: "?").first
            let params = queryComponents(urlStr: urlString)
            print("funcName = \(funcName ?? ""), params = \(params)")
        }
    }


// UIViewWebDelegate
func webView(_ webView: UIWebView, shouldStartLoadWith request: URLRequest, navigationType: UIWebViewNavigationType) -> Bool {
     if let urlStr = request.url?.absoluteString {
        if urlStr.hasPrefix("pawjscheme") {
            chongdingxiangjiaohu(urlString: urlStr)
        }
      }
     return true
}

// WKWebviewNavigationDelegate
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Swift.Void) {
     if let url = navigationAction.request.url, url.scheme == "pawjscheme" {
          chongdingxiangjiaohu(urlString: url.absoluteString)
          decisionHandler(.cancel)
          return
     }
     decisionHandler(.allow)
 }

// H5端的代码
document.location="pawjscheme://gotoGoodDetails?prdId="+p1+"&creditBalance="+p2;
```

2、iOS7之后可以使用JavaScriptCore实现原生与JS的交互，其中JS调用原生：  
定义好要添加的方法，并使用`unsafeBitCast`对方法进行类型转换；  
通过JSContext将转换后的方法添加/注入到JS里面，然后就可以在JS中调用该方法。

3、WKWebView中JS调用原生方法：

```
class KSWKWebViewController {
      override func viewDidLoad() {
          super.viewDidLoad()
          ...
          // 在合适的时机向webView中添加方法
        wkWebView.configuration.userContentController.add(self, name: "addFuncName")
        ...
    }
}

// 当前视图控制器服从WKScriptMessageHandler协议，通过该代理获取到来自JS的调用回调，并根据message.name区分方法 根据message.body获取参数
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

// 注意在合适的时机移除掉添加的方法
wkWebView.configuration.userContentController.removeScriptMessageHandler(forName: "addFuncName")

// JS中的调用上面add的原生方法
window.webkit.messageHandlers.addFuncName.postMessage({参数})
```



