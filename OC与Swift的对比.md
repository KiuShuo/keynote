##### 本文主要讲解了一下几个问题：
1. Swift与Objective-C的区别? 类（class）和结构体（struct）有什么区别？
Swift 是面向对象还是函数式的编程语言？在 Swift 中，怎样理解是 copy-on-write？
2. Swift协议中动态派发、扩展中静态派发
3. Optional可选类型 泛型? 在 Swift 中，什么是可选型（optional）？在 Swift 中，什么是泛型（Generics）？
4. 访问控制权限? 请说明并比较以下关键词：Open, Public, Internal, File-private, Private?  
5. 属性观察willSet didSet? 什么是属性观察（Property Observer）？
6. 在 Swift 中，怎样理解是 copy-on-write？

-----

## Objective-C与Swift的对比  

首先可以简单的说两个语言都是为iOS开发而定制的语言（当然不限于只能用于iOS的开发）；  

### 对比之--数据结构
##### 结构体和值类型的对比   

相比于Objective-C，Swift更加注重值类型数据结构，String Array Dictionary被设计成了值类型，这么做是因为：

* 值类型相较于引用类型，最大的优点在于内存高效。值类型在栈(stack)上操作，引用类型在堆(heap)上操作。栈上的操作仅仅是单个指针的上下移动，而堆上的操作则涉及到合并、位移、重新链接等。也就是说Swift这样的设计大幅减小了堆上内存分配和回收的次数，同时Copy-On-Write又将传递和复制的开销降到了最低。  
* String Array Dictionary被设计成值类型，也是为了线程安全考虑。通过Swift的let设计，使得数据达到了真正意义上的“不变”，他也从根本上解决了多线程中内存访问和操作顺序的问题。  
* Swift中的值类型可以服从协议，Objective-C不行；  
* 引用类型也有一些值类型没有的特性：引用类型可以继承、可以在deinit中释放资源、可以被多次引用、可以在运行时检查和解释一个实例的类型；

##### 值类型每次赋值操作都是一次复制吗？/ Copy-On-Write    
答案是否定的，值类型采用的是Copy-On-Write（写时复制）的机制，只有当值需要改变的时候才会进行复制，将值类型传递和复制的开销降到了最低，因此内存使用更加高效。  

Copy-On-Write:  

```
// array1是一个值类型的数组
var array1 = [1, 2, 3]
// 此时的array2和array1内存地址相同，指向同一块内存区域，内存中并没有多生成一个数组
var array2 = array1
// 当array2发生改变的时候，array2内存地址发生改变，在内存中就变成了一个新的数组
array2.append(4)

```

### 对比之--编程思路

如何实现Objective-C和Swift的混编:  
Swift调用OC文件，需要将OC文件导入到桥接文件ProjectName-Bridging-Header.h中，该文件一般会在OC项目中第一次创建Swift类型文件的时候提示自动生成，也可以自己生成.h一个文件后在buid settings中Objective-C Bridging Header进行路径设置。   
OC调用Swift代码，可以导入Swift生成的头文件ProgectName-Swift.h。  

##### Swift即是面向对象的也是函数式编程 也是面向协议的语言
Objective-C是面向对象的语言，实际上Swift也是面向对象的语言，他也支持面向对象的三个原则：封装、继承和多态。
同时Swift是函数式编程，因为他map、flatmap、filter、reduce这些去中间状态、数学函数式方法，更加强调计算结果而不是中间状态。 

##### 多态
定义：某一类事物的多种表现形式。父类指针指向子类对象。  
实现的原理：runtime 实现动态识别、动态绑定、动态加载。  
优点：简化了编程接口，允许类与类之间重用一些习惯性的函数名子；使得代码可以分散在不同的对象里面，不需要在一个方法/函数里面考虑到所有的可能。  

##### Swift和Objective-C中的Protocol有什么异同？  
[关联类型](https://swift.gg/2016/08/01/swift-associated-types/)  
[Swift中协议的关联类型](https://www.jianshu.com/p/ec2f5029a56b)  
[初探 Swift Sequences 和 Generators](https://swift.gg/2016/03/10/experimenting-with-swift-2-sequencetype-generatortype/)  
相同点：Swift和Objective-C中的协议都可以用作代理。  
不同点：Swift的Protocol可以用于值类型，如结构体和枚举；Swift的Protocol可以对接口进行抽象，例如对Protocol进行扩展、设置泛型、设置关联类型（associatedType）。  

可以通过协议扩展中增加默认实现的方式来控制协议中的方法是optional的还是required;  

注意：Swift中的值类型和引用类型都可以服从协议，所以设置代理时不能直接设置weak属性，可以通过设置协议只能为引用类型服从的方式来加以限制。

### 对比之--语言特性

##### 谈谈对Swift和Objective动态性的理解  
Swift被认为是静态语言，他的动态特性都是通过桥接OC来实现；  
OC的动态性通过runtime机制来实现。  
runtime可以实现：  
* 动态获取某个类的所有成员变量、成员方法；  
* 动态的为对象添加成员变量和成员方法；  
* 动态交换两个方法和实现（特别是交换系统自带的方法）。   

---

## 常见问题

##### 协议中动态派发、扩展中静态派发

```
protocol Chef {
    func makeFood()
}

extension Chef {
    func makeFood() {
        print("Make Food")
    }
    func test() {
        print("123")
    }
}

struct SeafoodChef: Chef {
    func makeFood() {
        print("Cook Seafood")
    }
    func test() {
        print("456")
    }
}

let chefOne: Chef = SeafoodChef()
let chefTwo: SeafoodChef = SeafoodChef()
chefOne.makeFood()  // Cook Seafood
chefTwo.makeFood()    // Cook Seafood
chefOne.test()	// 123
chefTwo.test()	// 456
```
协议中动态派发，协议中如果声明方法，那么方法调用时会根据对象的实际类型进行调用；  
扩展中静态派发，如果协议中没有声明方法，会根据对象的声明类型进行方法调用。   

##### Optional可选类型
Objective-C中没有明确的可选类型的概念，但引用类型默认是可以为nil的，值类型不行；Swift中明确提出了这个概念，并且将其扩展到了值类型上。   

##### 泛型
泛型主要是为了增加代码的灵活性，可以使得对应的代码满足任意类型的变量或方法。  

```
// 只能用来交换两个int类型的值
func swap(_ a: inout Int, _ b: inout Int) {
	(a, b) = (b, a)
}
// 不仅可以交换int类型的值，也可以用来交换float类型的值  
func swap<T>(_ a: inout T, _ b: inout T) {
	(a, b) = (b, a)
}

```

##### 访问控制权限
从高到低排序Open, Public, Internal, File-private, Private    
private：其修饰的对象（属性/方法）只能在当前的作用域下可以使用；  
fileprivate: 其修饰的对象只能在当前文件下使用（比方说同一文件中类的extention中）；  
internal: 默认权限，可以在同一module(模块)下进行重写和访问；  
public: 仅次于open，表示其修饰的对象可以在其他module(模块)下访问，但不能重写；  
open: 修饰的对象可以在其他模块中进行访问和重写。  

高权限的变量里面可以包含低权限的属性；低权限的变量里面不能包含高权限的属性。  

Objective-C没有命名空间，所有的代码和静态库最终都会被编译进同一个域和二进制，所以为了避免冲突或者重名，采用加前缀的方式进行解决；Swift同一个bundle下不能使用相同的名字进行命名，但可以通过新建framework的形式新建一个module。


##### 属性观察willSet didSet

    ```
    var title: String {
        willSet {
            print("旧\(tilte)将变成新的\(newValue)")
        }
        didSet {
            print("旧\(oldTitle)已变成新的\(title)")
        }
    }
    ```
初始化方法对属性的设定，以及在willSet、didSet中对属性的设定都不会引起属性观察的调用。  
