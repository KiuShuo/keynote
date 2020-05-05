##### 本文主要讲解了一下几个问题

面向对象编程与面向协议编程

-----

## OOP与POP

OOP Object-Oriented-Programming 面向对象编程，POP Protocol-Oriented-Programming 面向协议编程。  

Objective-C是面向对象编程的语言，Swift即是面向对象编程，也是面向协议编程的语言。

#### 面向对象的优点
1. 封装和权限控制：相关的属性和方法被放入一个类中。Swift中通过private/fileprivate/internal/public/open进行权限控制，Objective-C中公共的方法和变量放在.h文件中，私有的变量和方法放在.m文件中，同时实现所有方法。 
2. 继承和多态：将公共的方法和变量定义在父类里面，在子类继承时在实现各自对应的功能，做到代码复用的高效运作。同时针对不同的情况调用不同的子类，大大增加代码的灵活性。
3. 扩展性：在Objective-C中可以通过Category和protocol给原有的类添加方法和属性，在Swift中通过Extention和protocol实现。这样做可以在不破坏原有代码封装的情况下实现新的功能。
4. 命名空间：在Swift中有命名空间，在不同的module里面，由于命名空间不同所以可以有相同名字的类。Objective-C没有命名空间，所有的代码和静态库最终都会被编译进同一个域和二进制，所以为了避免冲突或者重名，采用加前缀的方式进行解决。

#### 面向对象的缺点
1. 冗杂的父类：比方说一个UIViewController的子类和一个UITableViewController都必须加入一个方法如handleSomething()，OOP的解决方案是直接在UIViewController的Category添加这个方法，但随着方法的增多，UIViewController会变得越来越冗杂，并且Category中的方法还必须有默认实现，子类中如果不满足的话还需要重写。
2. 多继承：Swift和Objective-C都不支持多继承，因为这会造成棱形问题，即多个父类实现同一个方法，子类无法判断哪个父类的情况。Swift中有类似的protocol的解决方案。
3. 隐式共享：class是引用类型，在代码中某处修改某个实例变量的时候，另一处在调用此变量的时候就会受到影响。在Swift中使用值类型就可以避免这个问题。


#### 面向协议相较于面向对象的优点
1. 协议可以用于值类型：Swift中protocol可以用于值类型，而OOP只能用于引用类型。
2. 消除动态分发的风险：在Objective-C的父类或者Category中声明了一个方法没有实现，在类中调用时，编译期并不会报错，但到了运行时就会报错；而Swift中的服从protocol的类或者结构体，都必须实现协议的方法，否则编译时就会报错。
3. 减少依赖：相较于在方法中传入实例变量，我们可以传入protocol实现多态，这样可以减少对于对象及其实现的依赖。
4. 更加灵活：