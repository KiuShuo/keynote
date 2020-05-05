# Swift工程编译速度

参考资料：  
[为什么Swift编译时间这么慢？](https://gxnotes.com/article/54102.html)  
[==Uber==-优化 Swift 编译速度](https://kemchenj.github.io/2017/04/30/2017-04-30/) `SWIFT_WHOLE_MODULE_OPTIMIZATION`  
[==王巍==-Swift 性能探索和优化分析](https://onevcat.com/2016/02/swift-performance/)  
[调整代码, 加速swift编译](http://www.jianshu.com/p/9825749efa8b)  
[如何有效提高swift的编译速度](http://www.jianshu.com/p/ea5c1ad0c26d)  
[评测 Swift 3.0 项目编译优化选项对编译速度的影响](https://imtx.me/archives/2106.html)  
[有效提升Swift编译速度
](http://hyyy.me/2016/12/01/SwfitCompileTimeSpeedingUp/)  


### 编译层面
结论：==开启`-wmo`会增加编译时间，但是会优化编译后的程序，所以按照Xcode的默认设置，debug状态下不开启`-wmo`，release状态下开启`-wmo`==。  


`-whole-module-optimization`(全模块优化，简称`-wmo`)，即在编译项目时，将同属于一个 module（可以理解为一个 Target、一个 Package）的所有源代码都串起来，进行整体的一个分析与优化。  

开启`-wmo`后编译阶段会多做什么？  

1. 检测没有被调用的方法和类型, 在预编译期去掉它们；  
2. 给没有被继承的类, 没有被继承的方法加上 final 标签, 给编译器提供更多信息, 以便这些方法被优化为静态调用或者是内联进去。  

优化带来的结果就是大幅优化最终目标文件，使性能提升 2~5 倍。所以release状态下最好要开启`-wmo`。  

为什么不建议在debug状态下开启`-wmo`？  

因为 `-wmo` 没有把增量编译优化好，可以发现，无论是改了一个没被多方引用的文件的私有函数，还是改了一个被其他文件引用的公开函数，增量编译都没能工作——编译器老老实实地把所有的文件再编译了一遍。开启`-wmo`会提升程序的效率, 但编译时需要加载更多的 context, 每合并一个文件, 就会遍历所有文件进行一次上面的检查，编译时间会随着文件的增多呈指数级增长。而在 `none` 下，这些更改带来的增量编译通常能更快完成编译。


### 代码层面

1. 指定类型与范型 √
2. 数字计算表达式 √
3. ?? 运算符与复杂的表达式一起使用  √
4. 过长的链式操作 
5. 尽量使用Swift类型 尽可能避免混合地使用 Swift 类型和 NSObject 子类，会对性能的提高有所帮助。X



