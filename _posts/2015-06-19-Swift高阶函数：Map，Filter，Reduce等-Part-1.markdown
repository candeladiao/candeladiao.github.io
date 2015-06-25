---
layout: post
title: Swift高阶函数：Map，Filter，Reduce等-Part 1
date: 2015-06-19
comments: true
archive: false
---

本文已投稿到[CocoaChina](http://www.cocoachina.com/swift/20150619/12173.html)

##闭包介绍Swift一大特性便是使用简洁的[头等函数](https://en.wikipedia.org/wiki/First-class_function)/闭包语法代替了复杂的blocks。希望我们在Swift中不再需要像[fuckingblocksyntax](http://fuckingblocksyntax.com/)中所描述的语法。（译者注：头等函数-即可将函数当作参数传递给其它的函数，或作为其它函数的返回值，也可以将它们赋值给变量，或者将它们存储在数据结构中）。
闭包是自包含的blocks，它可以在代码中传递使用。
本文我们将重点介绍匿名定义的闭包（如：定义成内联且不具名）也称匿名闭包。我们能够将其作为参数传递给其它函数/方法或者将其作为返回值。闭包是[极其强大的语言特性](https://en.wikipedia.org/wiki/Church_encoding)，它能够让我们更快，更容易的编写程序，并且也更少出错。
Blocks/闭包（同一概念的不同命名）在Cocoa和Cocoa Touch中广泛使用，是iOS框架的核心。
让我们来看一些闭包例子以及它们为什么如此有用。我们需要两个方法，其中一个计算两个数的平方的平均值，另一个计算两个数的立方的平均值，传统的解决办法会是这样：
	func square(a:Float) -> Float { 	    return a * a 	} 	 	func cube(a:Float) -> Float { 	    return a * a * a 	} 	 	func averageSumOfSquares(a:Float,b:Float) -> Float { 	    return (square(a) + square(b)) / 2.0 	} 	 	func averageSumOfCubes(a:Float,b:Float) -> Float { 	    return (cube(a) + cube(b)) / 2.0 	} 
	我们注意到averageSumOfSquares和averageSumOfCubes的惟一不同只是分别调用平方函数或立方函数。如果我能够定义一个通用函数，这个函数以两个数和一个使用这两个数的函数作为参数，来计算平均值而不是重复调用的话将会更好，我们可以使用闭包作为函数参数。
	func averageOfFunction(a:Float,b:Float,f:(Float -> Float)) -> Float { 	    return (f(a) + f(b)) / 2 	} 	 	averageOfFunction(3, 4, square) 	averageOfFunction(3, 4, cube) 
	这里我们使用了命名闭包，我们也可以使用闭包表达式定义一个没有名称的内联闭包。
在Swift中有很多种定义闭包表达式的方法，这里从最啰嗦的开始讲，逐步简化到最简洁为止：
	averageOfFunction(3, 4, {(x: Float) -> Float in return x * x}) 
	(x: Float) -> Float 是闭包的类型（接收一个float参数，返回一个float值）， return x * x 为实现，关键字in为代码头，实现跟在in后面。下面我们简化一下。
首先可以忽略类型声明，因为类型能够从averageOfFunction声明中推断出来（编译器能够推断出作为averageOfFunction的参数传入的函数需要接收一个float值，然后返回另一个float值）。
	averageOfFunction(3, 4, {x in return x * x}) 
	接着我们可以忽略return语句。（译者注：Swift规定-单行表达式闭包可省略return）。
	averageOfFunction(3, 4, {x in x * x}) 
	最后可以忽略指定参数名，使用默认参数名$0（如果函数接收多个参数，使用$K作为第K-1个参数，如$0，$1，$2......）。
	averageOfFunction(3, 4, {$0 * $0}) 
	接下部分，我们都使用最后这种闭包表达式。也支持你使用命名闭包，如果实际编码中有明确要求。
把闭包作为参数传递而非重复代码，这个技术极大的提升了代码的表达性，避免了代码引用和拷贝粘贴所带来的错误。
##数组操作
Swift的数组支持三个高阶函数：map，filter和reduce。Objective-C的NSArray没有实现这些方法，但是有开源社区弥补了这点不足。
###Mapmap用于将每个数组元素通过某个方法进行转换。
	[ x1, x2, ... , xn].map(f) -> [f(x1), f(x2), ... , f(xn)] 
    我们用一个Int类型数组存储商品金额，想把每个金额后面添加一个字符“€”，把数组转成字符串数组。
	[10,20,45,32] -> ["10€","20€","45€","32€"] 
最笨的办法-创建一个空数组，遍历和转换原数组的每个元素，然后把转换结果添加到空数组里。
	var stringsArray : [String] = [] //Note that we have to specify the type of the array or else we'll get an type error 	 	for money in moneyArray { 	    stringsArray += "(money)$" 	} 
	转换原数组元素并由此得到一个新数组的操作很普遍，我们可以用map函数来做这个事。
在Swift中，map被声明成Array类的命名方法：
	func map<U>(transform: (T) -> U) -> U[] 
这个方法以一个命名函数transform作为参数，transform负责把元素类型T转成类型U并返回一个类型U的数组。
在上面的事例中，T为Int，U为String，作为转换函数传给map的是一个把Int转成String。
	stringsArray = moneyArray.map({"($0)€"}) 
	{"($0)€"}就是一个闭包，它把金额转成字符并在尾部添加一个字符€带命名参数。
	stringsArray = moneyArray.map({money in "\(money)€"}) 
	如果看不明白，那可能是你不熟悉字符串插值，下面截取出相关文档的小部分内容：
>字符串插值是一种构造新字符串的方法，它把常量，变量和表达式组合构造出一个新字符串，每一个要插入字符串的项都需要以反斜杠开头并包裹在一对圆括号里。
[Swift String Apple Reference](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html#//apple_ref/doc/uid/TP40014097-CH7-XID_368)
##Filter
filter用于选择数组元素中满足某种条件的元素。
使用前面金额的例子，筛选出金额大于30元素，结果应该是[45,32]再看一下传统实现：
	var filteredArray : [Int] = []  		for money in moneyArray { 	if (money > 30) { 	    filteredArray += [money] 		} 	} 
	我们惟一感兴趣的是money > 30，下面使用filter来定义同样的逻辑。Swift中filter在Array类中的定义为：
	filter(includeElement: (T) -> Bool) -> T[]
	以返回true或false的函数includeElement作为参数，对原数组元素调用includeElement时，只有返回true的元素会通过筛选。
对数组筛选，我们只需使用
	filteredArray = moneyArray.filter({$0 > 30}) 
	{$0 > 30}为筛选闭包，我们忽略了参数名而使用默认参数$0，返回类型隐式推断为Bool。
##Reduce
reduce把数组元素组合计算为一个值。
我们使用上面例子计算数组元素的和，结果应该为107(10 + 20 + 45 + 32)
传统实现：
	var sum = 0 	for money in moneyArray { 	    sum = sum + money 	} 
	也看一下数组元素的积：
	var product = 1 	for money in moneyArray { 	    product = product * money 	} 
	这两个惟一不同的是计算的初始值（sum为0，product为1）和计算操作符（sum为+，product为*）
Reduce便可用于快速完成这类操作，通过指定一个初始值和一个合并元素的方法。
Swift中reduct在Array类中的定义为
	reduce(initial: U, combine: (U, T) -> U) -> U
	接收两个参数，一个为类型U的初始值，另一个为把类型为U的元素和类型为T的元素合并成一个类型为U的值的函数。最终结果整个数组就变成了一个类型为U的值。
在事例中U和T是Int，初始值是0，combine函数计算两个Int之和。
和计算改写成：
	sum = moneyArray.reduce(0,{$0 + $1}) 
	Swift中操作符可用着函数，可简化成：
	sum = moneyArray.reduce(0,+) 
	Reduce可能是三个高阶函数中最难理解的一个。 需要注意的是combine函数的两参数类型不同，$0为计算结果类型，$1为数组元素类型。
另一点需要说明的是数据比较大的时候，高阶函数会比传统实现更快，因为它可以并行执行（如运行在多核上），除非真的需要更高定制版本的map，reduce和filter，否则可以总是使用它们以获得更快的执行速度。
可以肯定使用map，filter，reduct的代码质量会更好。但也需要在合适的场景中使用它们，不要指望用它们来解决所有问题。没有放之四海而皆准的真理。
下面这些闭包问题作为练习
   1.编写一个方法applyTwice(f:(Float -> Float),x:Float) -> Float，以函数f和float值x作为参数，然后用x参数调用f两次。如f(f(x))。
      2.编写一个方法applyKTimes(f:(Float -> Float),x:Float) -> Float，以函数f和float值x作为参数，然后用x参数调用f k次。
      3.使用applyKTimes实现x的k次方。
      4.给定一个User类数组，类包含属性name:String和age:Int，用map函数返回一个用户name组成的字符串数组。
      5.给定一个Dictionary数组，Dictionary包含key值name和key值age，用map函数返回name字符串数组。
      6.给定一个Number数组，用filter函数选出奇数。
      7.给定一个String数组，用filter函数选出能被转成Int的字符串。
      8.给定一个UIView数组，用filter函数选出UILabel的子类。
      9.用reduce函数把String数组中元素连接一个字符串，以换行符分隔。
      10.使用reduce函数找出整形数组中的最大元素。
      11.为什么不能用reduce以这种方式实现平均函数：{$0 + $1 / Float(array.count)}。
      12.实现一个reduce功能时，什么属性可以使操作更简单。
      13.在Swift中实现[丘奇数](https://en.wikipedia.org/wiki/Church_encoding)（终级练习，也是最难的）。。
   第二部分中，我们将结合具体用法，创建高阶数组函数，探寻其他有趣事例
敬请期待  
[工程代码](http://www.weheartswift.com/wp-content/uploads/2014/06/Clojures.zip)
如果觉得本文对你有用，请分享给你的朋友。
原文 [http://www.weheartswift.com/higher-order-functions-map-filter-reduce-and-more/](http://www.weheartswift.com/higher-order-functions-map-filter-reduce-and-more/)