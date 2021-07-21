参考资料：  
[Swift和Javascript的神奇魔法](https://www.jianshu.com/p/dadd1bd83752)  
[JavaScriptCore初探](https://hjgitbook.gitbooks.io/ios/content/04-technical-research/04-javascriptcore-note.html)  
[NSHipster -- Java​Script​Core](http://nshipster.cn/javascriptcore/) 

Demo：  
* [PADemo](https://github.com/KiuShuo/PADemo)   
其中： 
KSUIWebViewController类中有在UIWebView下使用JavaScriptCore实现native和JavaScript交互的示例；    
KSWKWebViewController类提供了WKWebView与JavaScript的交互示例  
* [Swift-JavascriptDemo](https://github.com/KiuShuo/Swift-JavascriptDemo)  
BasicsViewController中提供了不依赖于WebView进行native和JS交互的代码示例

## JavaScript于原生App交互的几种方式

### iOS7之前  
拦截/重定向的方式实现交互，必须依赖于WebView才能实现。
通过[note-on-iOS-work-with-JavaScript](https://github.com/movii/note-on-iOS-work-with-JavaScript)中的一个简单的代码片段可以说明UIWebView中怎样进行拦截实现JS与UIWebView的交互：

```
extension ViewController: UIWebViewDelegate {
  func webView(_ webView: UIWebView,
             shouldStartLoadWith request: URLRequest,
             navigationType: UIWebViewNavigationType) -> Bool {
    
    // 1 拦截名为 hello 协议；拦截之后根据具体的参数做具体的操作，从而实现了JS对原生的调用
    if request.url?.scheme == "hello" {
     
     // 2 获取协议后的 path，也就是lien；
     let path = request.url?.absoluteString as String!

      // 3 将 path 的 string 作为 window.alert() 方法的参数，
      // 通过 webview.stringByEvaluatingJavaScript() 方法执行该 JavaScript 函数；
      let msg = String(format: "alert('intercept scheme: %@')", path!)
      //func stringByEvaluatingJavaScript(_:String) -> String 这是UIWebView中实现对JS调用的函
      webview.stringByEvaluatingJavaScript(from: msg)

      // 4 因为拦截到了对应的hello 协议，在 delegate method 最终返回 false，
      // 阻止 UIWebView 继续执行该请求（跳转了也是个 404）。
      return false
    }

    return true
 }
}
```
拦截重定向的方式进行交互看似简单，但只能在WebView中使用成了其主要的限制。

### iOS7之后 JavaScriptCore  

##### JavaScriptCore可以实现native不依赖于webView直接和JavaScript进行交互，也提供了UIWebView与JavaScript进行交互的方式。注意：WKWebView与JavaScript的交互不使用JavaScriptCore。  

通过几个重要类型了解JavaScriptCore

* JavaScript代码的执行环境 - JSContext JS上下文

 ```
 // 官方API对JSContext的描述
 // JS上下文是JavaScript的执行环境，所有的JavaScript都在这个上下文里面运行，所有的JavaScript值都与这个上下文相关联。  
 A JSContext is a JavaScript execution environment. All
 JavaScript execution takes place within a context, and all JavaScript values
 are tied to a context. 
 ```  

* 任何出自JSContext的值都被包裹成JSValue类型，再从JSValue类型转换成需要的类型；也可以说任何JSValue值都来源于JSContext，同时JSValue强引用JSContext，所以在使用时要特别注意内存问题。

 ```
 // 官方API对JSValue的描述
 A JSValue is a reference to a JavaScript value. Every JSValue
 originates from a JSContext and holds a strong reference to it.
 When a JSValue instance method creates a new JSValue, the new value
 originates from the same JSContext.
 ```  

* JSExport协议可以将OC/Swift类型的属性、实例方法、类方法、析构函数(初始化方法)转换成JavaScript。从而实现JS对OC/Swift的调用。

 ```
 // 官方API对JSExport的描述
 JSExport provides a declarative way to export Objective-C objects and
 classes -- including properties, instance methods, class methods, and
 initializers -- to JavaScript.
 ```

##### JavaScriptCore中原生调用JS的简单示例：

 ```
 let context = JSContext()
 context.evaluateScript("var num = 5 + 5")
 context.evaluateScript("var names = ['Grace', 'Ada', 'Margaret']")
 context.evaluateScript("var triple = function(value) { return value * 3 }")
 let tripleNum: JSValue = context.evaluateScript("triple(num)")
 ```

 ```
 // JS代码片段
 function sumValue (a, b) {
     return a + b;
 }
 // Swift代码片段
 let context = JSContext()
 let sumFunc = context.objectForKeyedSubscript(sumValue)
 let sumResult = sumFunc?.call(withArguments: [1, 2])
 print("1 + 2 = \(sumResult.toInt32())") 
 ```

##### JavaScriptCore-JS调用原生

 ```
let nativeLog: @convention(block) (String) -> Void = { str in
    print("str = \(str)")
}
// unsafeBitCast用来强制类型转换 使用的时候需要明确的知道要转换的类型
let nativeLogObject = unsafeBitCast(nativeLog, to: AnyObject.self)
// setObject(:, forKeyedSubscript:) 为JS添加/注入方法或属性
context.setObject(nativeLogObject, forKeyedSubscript: "nslog" as NSCopying &
NSObjectProtocol)
// 通过上面的代码，JS里面就有了一个nslog函数可以直接使用了。
 ```

注意：webview获取JSContext使用下面的代码片段属于私有API，最好不用以免被拒，所以要找到一个替换方案。

 ```
 if let aContext = webView.value(forKeyPath: "documentView.webView.mainFrame.javaScriptContext") as? JSContext {
    self.context = aContext
 }
 ```

## WKWebView

WkWebView中获取不到JSContext，不使用JavaScriptCore与JavaScript进行交互。在下一片介绍[WKWebView的文章](https://github.com/KiuShuo/--kiushuo/wiki/JS%E4%BA%A4%E4%BA%92%E4%B9%8BWKWebView)中会详细说明WKWebView与JavaScript的交互方式。  
