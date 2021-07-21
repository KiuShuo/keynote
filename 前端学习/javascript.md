# JavaScript学习

JavaScript代码可以直接嵌在网页的任何地方, 通常会把JavaScript代码放在<head>中.  
网页的js代码要放在`<script>js代码</script>`标签中, 一个文件中可以写多个`<script>js代码</script>`, 浏览器按照顺序依次执行.  

js代码也可以单独的放在.js文件中, 然后通过`<script src="js file path"></script>`引入文件.  
将js代码单独放在.js文件中便于维护代码, 并且多个页面可以各自引用同一份.js文件, 当然一个页面中也可以引入多个.js文件.  

JavaScript每个语句以`;`结尾, 但是浏览器中负责执行JavaScript代码的引擎会自动在每个语句的结尾补上`;`, 所以理论上可以不写, 但为了避免在某些情况下引擎自动加的分号出错, 建议还是写上.  
JavaScript严格区分大小写, 如果弄错了大小写, 程序将报错或者运行不正常.  

### 数据类型和变量

##### Number
JavaScript中不区分整数和浮点数, 统一使用Number表示.  

```
NaN; // NaN表示Not a Number, 当无法计算结果时用NaN表示
Infinity; // Infinity表示无限大, 当数值超过了JavaScript的Number所能表示的最大值时,就表示为Infinity
```

##### 布尔值
JavaScript中有两种比较运算符`==`和`===`;   
`==`会自动转换数据类型再进行比较, 如  
```
false == 0; // true
```
`===`不会自动转换数据类型, 如果数据类型不一致, 直接返回false, 如果一致, 再比较, 如:  
```
false === 0; // false
```

另外, NaN这个特殊的Number与其他所有的值都不相等, 包括他自己:  
```
NaN === NaN; // false
```
唯一能判断NaN的方法是通过isNaN()函数:  
```
isNaN(NaN); // true
```

##### null和undefined
null表示一个空值, 类似于OC中的null和Swift中的nil;  
undefined表示值未定义.   
大多数情况下, 我们对使用null, undefined仅仅在判断函数参数是否传递的情况下有用.  

##### 对象
JavaScript中的对象是一组由键值组成的无需集合(类似于OC中的字典), 例如:  

```
var person = {
		name: 'Bob',
		age: 20,
		tags: ['js', 'web', 'mobile'],
		city: 'Beijing',
		hasCar: true,
		zipcode: null
	};
	
person.name; // Bob
person.zipcode; // null
```

##### 变量
在JavaScript中使用var开头来声明一个变量; JavaScript中的变量可以使用任意数据类型进行赋值, 同一个变量可以反复赋值, 并且可以是不同的类型, 如:

```
var a = 123; // a的值是整数类型
a = 'ABC'; // a的值现在是字符串
```

##### strict模式
不用var声明的变量会被视为全局变量, 当引入多个JavaScript文件的时候, 如果有多个同名变量都没有使用var进行声明, 则会产生混淆, 为了避免这一缺陷, 在JavaScript代码的第一行加上字符串`'use strict;'`来启用strict模式, 当下面的JavaScript代码中的变量没有使用var进行声明时控制台将会报错.  

##### 字符串
使用反引号`(也就是键盘上左上角esc按键下面的那个按键)代替单引号'来引用字符串:  
1. 可以在不使用换行符的情况下直接支持换行;  
2. 可以在字符串中起到替换变量的作用;
如:   

```
console.log(`第一行
第二行
第三行`)
// 等同于
console.log('第一行\n第二行\n第三行')

var num1 = '1';
var num2 = '2';
console.log(`${num1}${num2}345`) // 12345
console.log('${num1}${num2}345') // ${num1}${num2}345
console.log(num1 + num2 + '345') // 12345
```
在JavaScript中, 可以通过下标来访问字符串中某一个位置的字符, 但是不能通过访问下标来修改字符串, 如:  

```
var s = 'ahcd';
s[1] = 'b'; // 这样做虽然不会报错, 但也起不到修改的作用
console.log(s); // ahcd
```

##### 数组
JavaScript中的数组可以包含任意数据类型, 并通过索引来访问每个元素.  

```
var arr = [1, 'a', null, true]
arr.length; // 4
arr.lenght = 5; //直接给length赋一个新值会改变arr的大小, arr = [1, 'a', null, true, undefined]
arr.length = 2; // arr = [1, 'a']
arr[3]; // 通过下标访问数组中的元素, 当访问越界时并不会报错, 而是返回undefined
arr[2] = 3; // arr = [1, 'a', 3] 通过索引可以访问数组中指定下标的元素, 也可以设置指定下标的值来修改数组
arr[4] = 3; // arr = [1, 'a', 3, undefined, 3] 中间没有的补上undefined
arr.indexOf(3); // 数组中第一个元素为3的索引为2
arr.indexOf(100); // 找不到的返回-1
arr.slice(1, 3); // 从索引1开始到索引3结束, 但不包含索引3的子数组['a', 3] 注意和OC中的不同
arr.slice(1); // ['a', 3, undefined, 3] 表示的是从下标1开始到结束
arr.slice(); // 表示复制了一份
```


##### 函数
使用let于var声明变量的区别: 由于JavaScript中变量的作用域实际上是在函数内部, 为了解决块级作用域的需求, 引入了let进行声明. 以下例子是二者的区别:  

```
       function textFunction() {
            for (var j = 0; j < 10; j++) {
                //
            }
            for (let i = 0; i < 10; i++) {
                //
            }
            console.log('j = ' + j); // j = 10
            console.log('i = ' + i); // RefrenceError: i is not defined
        }
```

函数中的解构赋值类似于Swift中的元组:  

```
        function textFunction() {
            // JavaScript中的解构赋值 类似与Swift中的元组, 只是元组都是圆括号, 而解构赋值有时候是中括号, 有时候是大括号

            // 快速获取数组中的部分元素
            var [x, , z] = ['a', 'b', 'c'];
            console.log(x, z);
            // 获取对象属性的值
            var person = {
                name: 'zhangsan',
                age: 89,
                sex: 'male',
                address: {
                    city: 'beijing',
                    number: '001'
                }
            }
            // 注意变量的名称必须和属性名称一致, 如果想换名可使用oldName: newName的形式替换
            var {name, sex: gender, address: {city}} = person;
            console.log(`name: ${name}, gender: ${gender}, city: ${city}`);
            // 不引入第三方的情况下进行交换
            var f = 1, g = 2;
            [f, g] = [g, f];
            console.log(`f = ${f}, g = ${g}`);
            // 快速获取当前页面的域名和路径
            var {hostname: domain, pathname: path} = location;
            console.log(`hostName: ${domain}, pathName: ${path}`);
            // 如果一个函数接收一个对象作为参数, 那么, 可以使用解构赋值直接把对象的属性直接绑定到变量中, 
            // 类似于Swift中给参数设置默认值, 只是这里面是给对象变量的属性设置默认值
            function buildDate({year, month, day, hour = 0, minute = 0, second = 0}) {
                return new Date(year + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second)
            }
            var today = buildDate({year: 2018, month: 12, day: 18, hour: 10, second: 31});
            console.log(today);
        }
```


