### UIScrollView

参考资料：  
[UIScrollView的详细使用介绍和实现原理分析](https://juejin.im/entry/57d7ae538ac247006211770f)  
[UIScrollView 实践经验](https://mp.weixin.qq.com/s/x5hD0P-sCRvVlXZqlzpDxQ)  



几个常用的属性
  
```
// default CGPointZero 内容视图的原点相对于UIScrollView原点的偏移量，向左向上偏移为正值
open var contentOffset: CGPoint 

// default CGSizeZero 内容视图的大小
open var contentSize: CGSize 

// default UIEdgeInsetsZero. add additional scroll area around content
// 为内容视图周围额外增加的可滑动区域
open var contentInset: UIEdgeInsets 
```

<img src = https://raw.githubusercontent.com/KiuShuo/imageSource/master/%E7%9F%A5%E8%AF%86%E7%82%B9/UIScrollView_01.jpeg>

<img src = https://raw.githubusercontent.com/KiuShuo/imageSource/master/%E7%9F%A5%E8%AF%86%E7%82%B9/UIScrollView_02.jpeg>

<img src = https://raw.githubusercontent.com/KiuShuo/imageSource/master/%E7%9F%A5%E8%AF%86%E7%82%B9/UIScrollView_03.jpeg>
