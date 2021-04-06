---
layout: default
title: Swift
nav_order: 17
parent: 语言
has_children: true
---

## 大纲

Swift Package

```
swift package init   
swift package init --type executable 
swift package init --type library            
swift package generate-xcodeproj

```

## 术语

- 值
- 变量
- 引用
- 常量

值(value)是不变的，永久的，它从不会被改变。字面量。结构体和枚举是值类型。当你把一个结构体变量赋值给另一个，那么这两个变量将会包含同样的值。

引用(reference)是一种特殊类型的值：它是一个"指向"另一个值的值。类是引用类型。引用类型具有同一性，也就是说，你可以使用===来检查两个变量是否确实引用了同一个对象。

在程序语言的论文中，==有时候被称为结构相等，而===则被称为指针相等或者引用相等。

一个引用变量也可以用let来声明，这样做会使引用变量为常量。换句话说，这会使变量不能被改变为引用其他东西，不过很重要的是，这并不意味着这个变量所引用的对象本身不能被改变。这一点造成的问题是，就算在一个声明变量的地方看到let，你也不能一下子就知道声明的东西是不是完全不可变的。想要做出正确的判断，你必须先知道这个变量持有的是值类型还是引用类型。

复制值类型时，它通常执行深复制，也就是说，它包含的所有值会被递归地复制。这种复制可能是在赋值新变量时就发生的，也可能会延迟到变量内容发生变更的时候再发生。执行深复制的类型被称作具有值语义。

这里我们会遇到另一件复杂的事情。如果我们的结构体中包含有引用类型，在将结构体赋值给一个新变量时所发生的复制行为中，这些引用类型的内容是不会被自动复制一份的，只有引用本身会被复制。这种复制的行为被称作浅复制。

Data结构体实际上是对引用类型NSData的一个封装。当Data结构体发生变化的时候对其中的NSData对象进行深复制，它使用一种名为"写时复制的技术来保证操作的高效。我们需要重点知道的是，这种写时复制的特性并不是直接具有的，它需要额外进行实现。

Swift中，像是数组这样的集合类型也都是对引用类型的封装，它们同样使用了写时复制的方式在提供值语义的同时保持高效。不过，如果集合元素的类型是引用类型(比如一个含有对象的数组)的话，对象本身将不会被复制，只有对它的引用会被复制。也就是说，Swift的数组只有当其中的元素满足值语义时，数组本身才具有值语义。

有些类是完全不可变的，也就是说，从被创建以后，它们就不提供任何方法改变它们的内部状态。这意味着即使它们是类，它们依然具有值语义（因为就算被到处使用也从不会改变）。但是需要注意的是，只有那些标记为final的类能够保证不被子类化，也不会被添加可变状态。

在Swift中，函数也是值。你可以将一个函数赋值给一个变量，也可以创建一个包含函数的数组，或者调用变量所持有的函数。如果一个函数接受别的函数作为参数(比如map函数接受一个转换函数，并将其应用到数组中的每个元素中)，或者一个函数的返回值是函数，那么这样的函数叫做高阶函数。

函数不需要被声明在最高层级---你可以在一个函数内部声明另一个函数，也可以在一个do作用域或者其他作用域中声明函数。如果一个函数被定义在外层作用域中，但是被传递出这个作用域（比如这个函数被作为其他函数的返回值时），它将能够捕获局部变量。这些局部变量将存在于函数中，不会随着局部作用域的结束而消亡，函数也将持有它们的状态。这种行为的变量被称为闭合变量，我们把这样的函数叫做闭包。

函数可以通过func关键字来定义，也可以通过{}这样的简短的闭包表达式来定义，有时简称为闭包，不过不要让这种叫法蒙蔽了你的双眼。实际上使用func关键字的函数，如果它包含了外部的变量，那么他也是一个闭包。

函数是引用类型。也就是说，将一个捕获了状态的函数赋值给另一个变量，并不会导致这些状态被复制。和对象引用类似，这些状态会被共享。换句话说，当两个闭包持有同样的局部变量时，它们是共享这个变量以及它的状态的。

定义在类或者协议中的函数是方法，它们有一个隐式的self参数。有时我们会把那些不是方法的函数叫做自由函数，这可以将它们与方法区分开来。

自由函数和那些在结构体上调用的方法是静态派发的。对于这些函数的调用，在编译的时候已经确定了。对于静态派发的调用，编译器可能会实施内联优化，也就是说，完全不去做函数调用，而是将函数调用替换为函数中需要执行的代码。优化器还能够帮助丢弃或者简化那些在编译时就能确定不会被实际执行的代码。

子类型和方法重写是实现多态特性的一种手段，也就是说，根据类型的不同，同样的方法会呈现出不同的行为。第二种方式是函数冲载。它是指为不同类型多次写一个函数的行为。实现多态的第三种方法是通过泛型，也就是一次性地编写能够接受任意类型的函数或者方法，不过这些方法的实现会各有不同。与方法重写不同的是，函数重载和泛型中的方法在编译期间就是可以确定的。

## Swfit风格指南

- 对于命名，在使用时能清晰表意是最重要的。[Swift API设计准则](https://swift.org/documentation/api-design-guidelines/)
- 简洁经常有助于代码清晰，但是简洁本身不应该独自成为我们编码的目标。
- 务必为函数添加文档注释---特别是泛型函数。
- 类型使用大写字母开头，函数、变量和枚举成员使用小写字母开头。
- 使用类型推断。省略掉显而易见的类型会有助于提高可读性。
- 如果存在歧义或者进行定义契约（比如func就需要显式地指定返回类型）时候不要使用类型推断。
- 优先选择结构体，只在确实需要使用到类特有的特性或者引用语义时才使用类。
- 除非你的设计就是希望某个类被继承使用，否则都应该将它们标记为final。如果你允许这个类被模块内部继承，但不允许外部的用户进行子类化，那么标记这个类为public,而不是open。
- 除非一个闭包后面立即跟随有左括号，否则应该使用尾随闭包的语法。
- 使用guard来提早退出方法。
- 避免对可选值进行强制解包和隐式强制解包。
- 不要写重复的代码。试着将它们提取到一个函数里，并且考虑将这个函数转化为协议扩展的可能性。
- 试着去使用map和reduce,但这不是强制的。当合适的时候，使用for循环也无可厚非。高阶函数的意义是让代码可读性更高。但是如果使用reduce场景难以理解的话，强行使用往往事与愿违，这种时候简单的for循环可能会更清晰。
- 试着去使用不可变值：除非你需要改变某个值，否则都应该使用let来声明变量。
- 除非你确实需要，否则不要使用self。不过在闭包表达式中，self是被强制使用的，这是一个清晰的信号，表明闭包将会捕获self。
- 尽可能地对现有的类型和协议进行扩展，而不是写一些自由函数。

## 数组

### 数组索引
- 迭代数组：for x in array
- 迭代除了第一个元素以外的数组其余部分： for x in array.dropFirst()
- 迭代除了最后5个元素以外的数组：for x in array.dropLast(5)
- 列举数组中的元素和对应的下标：for (num, element) in collection.enumerated()
- 寻找一个指定元素的位置：if let idx = array.index{someMatchingLogic($0)}
- 对数组中所有元素进行变形：array.map{ someTransformation($0) }
- 筛选出符合某个特定标准的元素：array.filter{ someCriteria($0) }

first和last属性返回一个可选值，当数组为空时，它们返回nil。first相当于isEmpty? nil : self[0]。类似地，如果数组为空时调用removeLast，那么将会导致崩溃，然后popLast在数组不为空时删除最后一个元素并返回它，在数组为空时，它将不执行任何操作，直接返回nil。

### 数组变形

#### Map

- 长度很短，意味着错误少，比原来更清晰
- 一个函数被作用在数组的每个元素上
- 返回一个新数组，包含所有被转换后的结果

实现：把for循环中的代码模版部分，使用一个泛型函数封装起来。
~~~
extension Array {
	func map<T>(_ transform:(Element) -> T) -> [T] {
		var result:[T] = []
		result.reserveCapacity(count)
		for x in self {
			result.append(transform(x))
		}
		return result
	}
}
~~~

Element是数组中包含的元素类型的占位符，T是元素转换之后的类型占位符。map函数本身并不关心Element和T究竟是什么，它们可以是任意类型。T的具体类型由函数调用者传入给map的transform方法的返回值类型决定。


#### 使用函数将行为参数化

map设法将模版代码分离出来，这些模版代码并不会随着每次调用发生变动，发生变动的那些功能代码，也就是如何转换每个元素的逻辑。map通过调用者所提供的变换函数作为参数来做到这一点。

- map和flatMap：对元素进行变换
- filter：只包含特定的元素
- allSatisfy：针对一个条件测试所有元素
- reduce：将元素聚合成一个值
- forEach：访问每一个元素
- sort(by:)，sorted(by:),lexicographicallPrecedes(_:by:),partition(by:)重排元素
- firsIndex(where:),lastIndex(where:),first(where:),last(where:),contains(where:) 元素是否存在
- min(by:),max(by:)：找到所有元素的最小、最大值
- elementsEqual(_:by:)、starts(with:by:) 将元素与另一个数组进行比较
- spilt(whereSeparator:)：把所有元素分成多个数组
- prefix(while:)： 从头取元素直到条件不成立。
- drop(while:) 当条件为真时，丢弃元素，一旦不为真，返回其余的元素(和prefix类似，不过返回相反的集合)
- removeAll(where:)：删除所有符合条件的元素


#### 可变和带有状态的闭包

闭包是指那些可以捕获和修改自身作用域之外的变量的函数，当它和高阶函数结合时也就成为一了一种强大的工具。

带有状态的闭包实现:
~~~
extension Array {
	func accumulate<Result>(_ initialResult: Result, _ nextPartialResult:(Result, Element) -> Result) -> [Result] {
		var running = initialResult
		return map { next in
			running = nextPartialResult(running,next)
			return running
		}
	}
}
~~~

这个函数创建了一个中间变量running来存储每一步的值，然后使用map来从这个中间值逐步计算结果数组:
~~~
[1,2,3,4].accumulate(0,+) // [1,3,6,10]
~~~

#### filter

检查一个数组，然后将这个数组中符合一定条件的元素过滤出来并用它们创建一个新的数组。对数组进行循环并且根据条件过滤其中的模式可以用filter方法表示：

~~~
let nums = [1,2,3,4,5,6,7,8,9,10]
nums.filter {num in num % 2 == 0 } // [2,4,6,8,10]
nums.filter { $0 % 2 == 0 }
~~~

实现：
~~~
extensino Array {
	func filter(_ isIncluded:(Element) -> Bool) -> [Element] {
		var result:[Element] = []
		for x in self where isIncluded(x) {
			result.append(x)
		}
		return result
	}
}
~~~

需要所有结果时才考虑用filter

#### reduce

map和fliter都作用在一个数组上，并产生另一个新的、经过修改的数组。不过有时候，你可能会把所有元素合并为一个新的单一的值。比如，要是我们想将元素的值全部加起来，可以这样写：
~~~
let fibs = [0,1,2,3,5]
var total = 0
for num in fibs {
	total = total + num
}

total //12
~~~

reduce方法采用这种模式，并抽象出两部分；一个初始值(在这里是0),以及一个将中间值(total)与序列中的元素(num)进行合并的函数。使用reduce，我们可以将上面的例子重写为这样：
~~~
let sum = fibs.reduce(0) {total, num in total + num }//12
~~~

运算符也是函数，所以也可以写成这样：
~~~
fibs.reduce(0,+) // 12
~~~

reduce的输出值类型不必和元素的类型相同。举个例子，如果我们想将一个整数列表转换为一个字符串，并且每个数字后面跟一个逗号和空格，那么可以这样：
~~~
fibs.reduce(""){str.num in str + "\(num),"} // 0,1,1,2,3,5,
~~~

reduce的实现是这样的：
~~~
extensino Array {
	func reduce<Result>(_ initialResult: Result, _ nextPartialResult:(Result,Element) -> Result) -> Result {
		var result = initialResult
		for x in self {
			resutlt = nextPartialResult(result,x)
		}
		return result
	}
}
~~~


#### 一个展平的map

有时候我们想对一个数组进行map，但这个变形函数返回的是另一个数组，而不是单独的元素。

举个例子，假如我们有个叫做extracLinks的函数，它会读取一个markdown文件，并返回包含该文件中所有链接的URL的数组。这个函数是这样的：
~~~
func extractLinks(markdownFile: String) -> [URL]
~~~

如果我们有一堆markdown文件，并且想将这些文件中所有的链接都提取到一个单独的数组中的话，可以尝试使用markdownFile.map(extractLinks)。不过问题是这个方法返回的是一个包含了URL的数组的数组，其中每个元素都是一个文件中的URL数组。现在你可以在执行map得到了一个数组的数组之后，调用joined把这个二维数组展平(flatten)成一个一维数组：
~~~
let markdownFiles:[String] = //...
let nestedLinks = markdownFiles.map(extractLinks)
let links = nestedLinks.joined()
~~~

flatMap方法将变换和展平这两个操作合并为一个步骤。markdownFiles.flatMap(links)把所有markdown文件中的URL放到一个数组里面返回。

flatMap的函数签名看起来和map基本一致，只是它的变换函数返回的是一个数组。在实现中，它使用append（contentsOf:）代替了append(_ :),这样返回的数组是展平了:
~~~
extension Array {
	func flatMap<T>(_ transform:(Element) -> [T]) -> [T] {
		var result: [T] = []
		for x in self {
			result.append(contentsOf: transform(x))
		}
		return result
	}
}
~~~
flatMap的另一个常见使用情景是将不同数组里的元素进行合并。为了得到两个数组中元素的所有配对组合，我们可以对其中一个数组进行flatMap，然后在变换函数中对另一个数组进行map操作：
~~~
let suits = ["♠", "♥", "♣", "♦"]
let ranks = ["J","Q","K","A"]
let result = suits.flatMap { suit in
ranks.map { rank in (suit, rank)
} }
/*
[("♠", "J"), ("♠", "Q"), ("♠", "K"), ("♠", "A"), ("♥", "J"), ("♥",
"Q"), ("♥", "K"), ("♥", "A"), ("♣", "J"), ("♣", "Q"), ("♣", "K"), ("♣", "A"), ("♦", "J"), ("♦", "Q"), ("♦", "K"), ("♦", "A")]
*/
~~~

#### 使用forEach进行迭代

其与for循环的工作方式非常类似：把传入的函数对序列中的每个元素执行一次。和map不通，forEach不返回任何值，这一点特别适合执行那些带副作用的操作。

对集合中的每个元素都调用一个函数的话，使用forEach比较合适。比如你正在iOS上实现一个view controller，然后想把一个数组中的视图都加到当前view上的话，只需要写theViews.forEach(view.addSubview)就足够了。

在forEach中的return并不能让外部函数返回，它仅仅只是让闭包本身返回。
~~~
(1..<10).forEach{ number in
	printf(number)
	if number > 2 { return }
}
~~~

这段代码将会把输入的数字全部打印出来。return语句并不会终止循环，它做的仅仅是从闭包中返回。因此在forEach的实现会开始下一个循环迭代。这种时候，用for循环较好。


### 数组切片

想得到数组中除了首个元素以外的其他元素，可以这么做：
~~~
let slice = fibs[1...]
slice //[1,1,2,3,5]
type(of:slice) //ArraySlice<Int>
~~~

它将返回数组的一个切片(slice)，其中包含了原数组中从第二个元素开始的所有部分。切片类型只是数组的一种表示方式，它背后的数据仍然是原来的数组，只不过是用切片的方式进行表示。因为数组的元素不会被复制，所以创建一个切片的代价是很小的。

因为ArraySlice和Array都满足了相同的协议(当中最终的就是Collection协议)，所以两者具有的方法是一致的，因此你可以把切片当作数组来进行处理。如果你需要将切片转换为数组的话，是可以通过它传递给Array的构建方法来完整：
~~~
let newArray = Array(slice)
type(of:newArray) // Array<Int>
~~~

需要谨记的是切片和它背后的数组是使用相同的索引来引用元素的。因此，切片索引不需要从0开始。例如，在上面我们的fibs[1...]创建的切片的第一个元素的索引是1，因此错误地访问slice(0)元素会使我们的程序因为越界而崩溃。如果你操作切片的话，建议基于startIndex和endIndex属性做索引计算。

## 字典

字典包含键以及它们所对应的值，其中每个键都是唯一的。通过键来获取值所花费的平均时间是常量级的，作为对比，在数组中搜寻一个特定元素所花的时间将与数组尺寸成正比。

### 可变性

和数组一样，使用let定义的字典是不可变的，你不能向其中添加、删除或者修改条目。可以使用var定义一个可变字典。想要从字典移除一个值的话，要么用下标将对应的值设为nil，要么调用removeValue(forKey:)。后一种方法还会将被删除的值返回(如果删除的键不存在，则返回nil)

### 一些有用的字典方法

场景：将一个默认的设置字典和某个用户更改过的自定义设置字典合并。merge(_ : uniquingKeysWith:). 可以接受两个参数，第一个要进行合并的键值对，第二个是定义如何合并相同键的两个值的函数。可以使用该方法将一个字典合并至另一个字典中去。

~~~
var settings = defaultSettings
let overriddenSettings:[String: Setting] = ["Name",.text("Jane's iPhone")]
settings.merge(overriddenSettings, uniquingKeysWith: {$1})
settings
//["Name": Setting.text("Jane's iPhone"), "Airplane Mode": Setting.bool(false)]


~~~
使用了{$1}来作为合并两个值的策略。也就是说，如果某个键同时存在于settings和ovveriddenSettings中时，我们使用ovveriddenSettings中的值。

还可以从一个(Key,Value)键值对的序列中构建一个新的字典。如果我们能保证键是唯一的，那么就可以使用Dictionary(uniqueKeysWithValues:)。不过，对于一个序列中某个键可能存在多次的情况，就和上面一样，我们需要提供一个函数来对相同键对应的两个值进行合并。

比如，要计算序列中某个元素的出现次数，我们可以对每个元素进行映射，将它们和1对应起来，然后从得到的(元素，次数)的键值对序列中创建字典。如果我们遇到相同键下的两个值（也就是说，我们看到了同样的元素若干次），我们需要将次数用+累加起来就行了：

~~~
extension Sequence where Element: Hashable {
	var frequencies:[Element: Int] {
	let frequencyPairs = self.map { ($0, 1) }
	return Dictionary(frequencyPairs, uniquingKeysWith: +)
	}
}

let frequencies = "hello".frequencies // ["o":1, "h":1, "e":1, "l":2]
frequencies.filter { $0.value > 1} // ["l": 2]
~~~

对字典的值做映射。因为Dictionary是一个实现了Sequence的类型，所以它已有一个map方法来产生数组。不过有时候需要保持字典结构，只对其中的值进行变换，maoValues方法就是做这件事的：
~~~
let settingsAsStrings = settings.mapValues { setting -> String in
	switch setting {
	case .text(let text): return text
	case .int(let number): return String(number)
	case .bool(let value): return String(value)
	}
}

settingsAsStrings // ["Name": "Jane's iPhone", "AirPlane Mode": "false"]
~~~

### Hashable要求
字典其实就是哈希表。字典通过键的hashValue来为每个键在其底层作为存储的数组上指定一个位置。这也是Dictionary要求它的key类型需要遵守Hashable协议的原因。

标准库中的所有基本数据类型都是遵守Hashable协议的，它们包括字符串，整数，浮点数以及布尔值。另外，像是数组，集合和可选值这些类型，如果它们的元素都是可哈希的，那么它们自动成为可哈希的。

在很多情况下，编译器可以生成Hashable的实现，即使它不适用于某个特定的类型，标准库也带有让自定义类型可以挂钩的内置哈希函数。

对于枚举和结构体，只要它们是由可哈希的类型组成，那么Swift就可以帮我们自动合成Hashable协议所需要的实现。如果一个结构体的所有存储属性都是可哈希的，那么我们不用手动实现Hashable协议，结构体已经实现这个协议了。类似的，只要枚举包含可哈希的关联值，那么就可以自动实现了协议；对于那些没有关联值的枚举，甚至都不用显示声明要实现的Hashable协议。这不仅可以节省一开始实现的工作量，还可以在添加或者删除属性时自动更新实现。

如果不能利用自动Hashable合成(要么在实现一个类，要么处于哈希的目的，在自定义结构体中有几个属性需要被忽略)，那么首先需要让类型实现Equatable协议，然后可以实现hash(into:)方法来满足Hashable协议。这个方法接受一个Hasher类型参数，这个类型封装了一个通用的哈希函数，并在使用者向其提供数据时，捕获哈希函数的状态。它有一个接受任何可哈希值的combine方法。你应该通过调用combine方法的方式将类型的所有基本组件逐个传递给hasher。基本组件是那些构成类型实质的属性，你通常会想要排除那些可以被惰性重建的临时属性。

你应该使用相同的基本组件来进行相等性检查，因为必须遵循以下不变的原则：两个同样的实例(有你实现的==定义相同),必须拥有相同的哈希值。不过反过来不必为真：两个相同哈希值的实例不一定需要相等。不通哈希值的数量是有限的，然后很多可以被哈希的类型（比如字符串）的个数是无限的。
> 标准库的通用哈希函数使用一个随机种子作为其输入之一。也就是说，字符串"abc"的哈希值在每次程序执行时都会是不通的。随机种子是一种用来防止有针对性的哈希洪泛式拒绝服务攻击的安全措施。因为字典和集合是按照存储在哈希表中的顺序来迭代它们的元素，并且由于这个顺序是由哈希值决定的，所以这意味着相同的代码在每次执行时会产生不同的迭代顺序。如果需要哈希值每次都一样，例如为了测试，那么可以通过设置环境变量SWIFT_DETERMINISTIC_HASHING=1来禁用随机种子，但是你不应该在正式环境中这么做。

当使用不具有值语义的类型（比如可变的对象）作为字典的键时，需要特别小心。如果你在将一个对象作为字典后，改变了它的内容，它的哈希值和/或相等特性往往也会发生改变。这个时候你将无法在字典中找到它。这时字典会在错误的位置存储对象，这将导致字典内部存储的错误。对于值类型来说，因为字典中的键不会和复制的值共用存储，因此它也不会被外部改变，所以不存在这个问题。

## Set

通过哈希表实现的，集合中的元素也必须满足Hashable。Set遵守ExpressibleByArrayLiteral协议，也就是说，可以用数组字面量的方式初始化一个集合:
~~~
let naturals: Set = [1, 2, 3, 2]
~~~

### 集合代数

补集：
~~~
let iPods: Set = ["iPod touch", "iPod nano", "iPod mini", "iPod shuffle", "iPod Classic"]
let discontinuedIPods: Set = ["iPod mini", "iPod Classic", "iPod nano", "iPod shuffle"]
let currentIPods = iPods.subtracting(discontinuedIPods) // ["iPod touch"]
~~~

交集：
~~~
let touchscreen: Set = ["iPhone", "iPad", "iPod touch", "iPod nano"] 
let iPodsWithTouch = iPods.intersection(touchscreen)
// ["iPod nano", "iPod touch"]
~~~

并集：
~~~
var discontinued: Set = ["iBook", "Powerbook", "Power Mac"] discontinued.formUnion(discontinuedIPods)
discontinued
/*
["iPod shuffle", "iBook", "iPod Classic", "Powerbook", "iPod mini", "iPod nano", "Power Mac"]
*/
~~~

这里使用可变版本的formUnion来改变原来的集合（正因如此，我们需要将原来的集合用var声明）。几乎所有的集合操作都有可变以及不可变版本的形式，前一种都以form开头。

### 索引集合和字符集合

Set和OptionSet是标准库中唯一实现了SetAlgebra的类型，但是这个协议在Foundation中还被另外两个很有意思的类型实现了：IndexSet和CharacterSet。

IndexSet表示了一个由正整数组成的集合。当然你可以用Set<Int>来做这件事，但是IndexSet更加高效，因为它内部使用了一组范围列表进行实现。打个比方，现在你有一个含有1000个元素的table view，你想要一个集合来管理已经被用户选中的元素的索引。使用Set<Int>的话，根据选中的个数不同，最多可能会要存储1000个元素。而IndexSet不太一样，它会存储连续的范围，也就是说，在选取前500行的情况下，IndexSet里其实只存储了选择的首位和末位两个整数值。

同样地，CharacterSet是一个高效的存储Unicode编码点的集合，经常用来检查一个特定字符串是否只包含某个字符子集。

### 在闭包中使用集合

想要为Sequnce写一个扩展，来获取序列中所有的唯一元素，我们只需要将这些元素放到一个Set里，然后返回这个集合的内容就行了。不过，因为集合并没有定义顺序，所以这么做是不稳定的，输入的元素的顺序在结果中可能会不一致。为了解决这个问题，我们可以创建一个扩展来解决这个问题，在扩展方法内部我们还是使用Set来验证唯一性：
~~~
extension Sequence where Element: Hashable {
	func unique() -> [Element] {
		var seen: Set<Element> = []
		return filter { element in
			if seen.contains(element)
			return false
		} else {
			seen.insert(element)
			return true
		}
	}
}

[1,2,3,12,1,3,4,5,6,4,6].unique() // [1, 2, 3, 12, 4, 5, 6]
~~~

上面这个方法让我们可以找到序列中的所有不重复的元素，并且通过元素必须满足Hashable这个约束来维持它们原来的顺序。

## Range

范围代表的是两个值的区间，它由上下边界进行定义。你可以通过..<来创建一个不包含上边界的半开范围，或者使用...创建同时包含上下边界的闭合范围:

~~~
// 0 到 9, 不包含 10
let singleDigitNumbers = 0..<10
Array(singleDigitNumbers) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] //包含"z"
let lowercaseLetters = Character("a")...Character("z")
~~~

这些操作符还有一些前缀和后缀的变型版本，用来表示单边的范围：
~~~
let fromZero = 0...
let upToZ = ..<Character("z")
~~~

一共有五种不同的具体类型可以用来表示范围，每种类型都代表了对值的不同的约束。最常用的两种类型是Rang(由..<创建的半开范围)和ClosedRange(由...创建的闭合范围)。两者都有一个Bound的泛型参数：对于Bound的唯一要求是它必须遵守Comparable协议。

对范围最基本的操作都是检测它是否包含了某些元素：
~~~
singleDigitNumbers.contains(9) // true 
lowercaseLetters.overlaps("c"..<"f") // true
~~~

半开范围和闭合范围各有所用：
- 只有半开范围能表达空间隔(也就是下界和上界相等的情况， 比如5..<5)
- 只有闭合范围能包括其元素类型能表达的最大值(比如0..int.max)。而半开范围则要求范围上界是一个比自身所包含的最大值还要大1的值。


### 可数范围

范围看起来很自然地会是一个序列或者集合类型。并且确实可以遍历一个整数范围，或者像集合类型那样对它：
~~~
for i in 0..<10 {
print("\(i)", terminator: " ")
} // 0 1 2 3 4 5 6 7 8 9
singleDigitNumbers.last // Optional(9)
~~~

但并不是所有的范围都可以使用这种方式。比如，编译器不允许我们遍历一个Charactrer的范围：
~~~
// 错误:'Character' 类型没有实现 'Strideable' 协议。 
for c in lowercaseLetters {
	...
}

~~~

这是怎么回事呢？让Range满足集合类型协议是有条件的，条件是它的元素需要满足Strideable协议(你可以通过增加偏移来从一个元素移动到另一个)，并且步长(stride step)是整数：
~~~
extension Range: Sequence
	where Bound: Strideable, Bound.Stride: SignedInteger { /* ... */ }
extension Range: Collection, BidirectionalCollection, RandomAccessCollection
	where 
~~~

换句话说，为了能遍历范围，它必须是可数的。对于可数范围（满足了那些约束），因为对于Stride有整数类型这样一个约束，所以有效的边界包括整数和指针类型，但不能是浮点数类型。

### 范围表达式

所有这五种范围都满足RangeExpression协议。这个协议内容很简单，首先，它允许我们询问某个元素是否被包含在该范围中。其次，给定一个集合类型，它能够计算出表达式所指定的完整的Range：
~~~
public protocol RangeExpression {
	associatedtype Bound: Comparable
	func contains(_ element: Bound) -> Bool 
	func relative<C>(to collection: C) -> Range<Bound>
		where C: Collection, Self.Bound == C.Index }
~~~

对于下界缺失的部分范围，relative(to:)方法会把集合类型的startIndex作为范围下界。对于上界缺失的部分范围，同样，它会使用endIndex作为上界。

~~~
let arr = [1,2,3,4] 
arr[2...] // [3, 4] 
arr[..<1] // [1] 
arr[1...2] // [2, 3]
~~~

这种写法能够正常工作，是因为Collection协议里对应的下标操作符声明中，所接收的是一个实现了RangeExpression的类型，而不是上述五个具体的范围类型中的某一个。你甚至还可以将两个边界都省略掉，这样将会得到表示整个集合的一个切片：
~~~
arr[...] // [1, 2, 3, 4] 
type(of: arr) // Array<Int>

~~~

## 哨岗值

函数调用中返回一个"魔法"数来表示其并没有返回真实的值，这样的值被称为哨岗值。

### 通过枚举解决魔法数的问题

Swift在枚举中引入了"关联值"的概念。也就是说，枚举可以在它们的成员中包含另外的关联值。

~~~
extension Collection where Element: Equatable {
	func firstIndex(of element: Element) -> Optional<Index> {
		var idx = startIndex
		while idx != endIndex {
			if self[idx] == element {
				return .some(idx)
			}
			formIndex(after: &idx)
		}
		// 没有找到，返回.none
		return .none
	}

}
~~~

现在，用户就不会在没有检查的情况下，错误地使用一个值了:
~~~
var array = ["one", "two", "three"]
let idx = array.firstIndex(of: "four")
// 编译错误:remove(at:) 接受 Int，而不是 Optional<Int>。
 array.remove(at: idx)
~~~

相反，假设得到的结果不是.none,为了使用包装在可选值中的索引，你必须对其进行"解包":
~~~
var array = ["one","two","three"] 
switch array.firstIndex(of: "four") {
	case .some(let idx): 
		array.remove(at: idx)
	case .none:
		break // 什么都不做
}

~~~

## 可选值概览

### if let

使用if let 来进行可选值绑定(optional binding)要比上面使用switch语句要稍微好一些。if let语句会检查可选值是否为nil，如果不是nil，便会解包可选值。

你可以把布尔限定语句与if let搭配在一起使用。因此，如果要实现"查找到的位置是数组的第一个元素，就不删除它"这样的逻辑，可以这样：
~~~
if let idx = array.firstIndex(of: "four"),idx != array.startIndex {
	array.remove(at: idx)
}
~~~

你也可以在同一个if语句中绑定多个值。更赞的是，在后面的绑定中可以使用之前成功解包出来的结果。当你要连续调用多个返回可选值的函数时，这个功能就特别有用了。

### while let

while let 语句和if let 非常相似，它表示当一个条件返回nil时便终止循环。

标准库中的readLine函数从标准输入中读取内容，并返回一个可选字符串。当到达输入末尾时，这个函数将返回nil。所以，你可以使用while let 来实现一个非常基础的和Unix中cat 命令等价的功能：
~~~
while let line = readLine() {
	print(line)
}
~~~

和 if let 一样，你可以在可选绑定后面添加一个布尔值语句。如果你想要在EOF或者空行的时候终止循环的话，只需要加一个判断空字符串的语句就行了。要注意，一旦条件为false，循环就会停止

~~~
while let line = readLine(), !line.isEmpty {
	print(line)
}
~~~

where关键字

~~~
for i 0..<10 where i%2 == 0 {
	print(i, teminator: "")
} // 0 2 4 6 8
~~~

注意上面的where语句和while循环中的布尔语句工作方式有所不同。在while循环中，一旦值为false时，迭代就将停止。而在for循环中，它的工作方式和filter相似了。如果我们将上面的for循环用while重写的话，看起来是这样的：
~~~
var iterator2 = (0..<10).makeIterator()
while let i = iterator2.next() {
	guard i%2 == 0 else { continue }
	print(i)
}
~~~

### 双重可选值

一个可选值的包装类型也是一个可选值的情况，会导致可选值的嵌套。

~~~
let stringNumbers = ["1", "2", "three"]
let maybeInts = stringNumbers.map { Int($0) } // [Optional(1), Optional(2), nil]
~~~

现在得到一个元素类型为Optional<Int>(也就是Int？)的数组，这是因为Int.init(String)是可能失败的，只要字符串无法转换成整数。

于是，当使用for遍历maybeInts的时候，自然访问到的每个元素都是Int？了：
~~~
for maybeInt in maybeInts {
	//maybeInt 是一个Int？值
	//得到两个整数值和一个nil
}
~~~

由于next方法会把序列中的每个元素包装成可选值，所以iterator.next()函数返回的其实是一个Optional<Optional<Int>>值，或者说是一个Int？？。而while let 会解包并检查这个值是不是nil，如果不是，则绑定解包的值并运行循环体部分：
~~~
var iterator = maybeInts.makeIterator()
while let maybeInt = iterator.next() {
	print(maybeInt,terminator: " ")
}
// Optional(1) Optional(2) nil
~~~

当循环到达最后一个值，也就是从"three"转换而来的nil时，从next返回的其实是一个非nil的值，这个值是.some(nil)。while let将这个值解包，并将解包结果(也就是nil)来绑定到maybeInt上。如果没有嵌套可选值的话，这个操作将无法完成。

如果想对非nil的值做for循环的话，可以使用case来进行模式匹配：
~~~
for case let i? in maybeInts {
	//i 将是Int值， 而不是Int？
	print(i, terminator: " ")
}
// 1 2

//或者只对nil进行循环
for case nil in maybeInts {
	// 将对每个nil执行一次
	print("No value")
}
~~~

这里使用了x？这个模式，它只会匹配那些非nil的值，这个语法是.some(x)的简写形式，所以该循环还可以被写成:
~~~
for case let .some(i) in maybeInts {
	print(i)
}
~~~

基于case的模式匹配可以让我们把在switch的匹配中用到的规则同样应用到if，for和while上去。最有用的场景是结合可选值，但是也有一些其他的使用方式，比如：

~~~
let j = 5
if case 0..<10 = j {
	print("\(j)在范围内")
} //5在范围内
~~~

### if var and while var

除了let以外，你还可以使用var来搭配if、while和for。这让你可以在语句块中改变变量：
~~~
let number = "1"
if var i = Int(number) {
	i += 1
	print(i)
} // 2
~~~

不过注意，i会是一个本地的复制。任何对i的改变将不会影响到原来的可选值。可选值是值类型，解包一个可选值做的事情是将它里面的值复制出来。

### 解包后可选值的作用域

在Objective-C中，对nil发消息什么都不会发生。Swift里，我们可以通过"可选链"来达到同样的效果:
> delegate?.callback()

~~~
let dictOfFunctions:[String:(Int, Int) -> Int] = [
	"add":(+),
	"subtract":(-)
] 

dictOfFunctions["add"]?(1, 1) //Optional(2)


var a:Int? = 5
a? = 10
a //Optional(10)

var b:Int? = nil
b? = 10
b //nil
~~~

#### nil合并运算符

很多时候，你都会想在解包可选值的同时，为nil的情况设置一个默认值。而这正是nil合并运算符的功能：
~~~
let stringteger = "1"
let number = Int(stringeger) ?? 0
~~~

如果字符串可以被转换为一个整数的话，number将会是那个解包后的整数值。如果不能的话，Int.init将返回nil，默认值0会被用来赋值给number。也就是说lhs ?? rhs做的事情类似于这样的代码lhs != nil ? lhs! : rhs。

> array.first ?? 0 // 1

这个解决方法漂亮且清晰，"从数组中获取第一个元素"这个意图被放在最前面，之后是通过??操作符连接的使用默认值的语句，它代表"这是一个默认值"。

~~~
extension Array {
	subscript(guarded idx: Int) -> Element? {
		guard (startInex..<endIndex).conteains(idx) else {
			return nil
		}

		return self[idx]
	}
}

array.count > 5 ? array[5] : 0

array[guarded: 5] ?? 0
~~~

#### 在字符串插值中使用可选值

Swift的?? 运算符不支持这种类型不匹配的操作，确实，它无法决定当表达式两侧不共享同样的基础类型时，到底应该使用哪一个类型。不过，只是为了在字符串插值中使用可选值这一特殊目的的话，添加一个我们自己的运算符也很简单。

~~~
infix operator ???: NilCoalescingPrecedence

public func ???<T>(optional: T?, defaultValue: @autoclosure () -> String) -> String
{
	switch optional {
		case let value?: return String(describing: value)
		case nil: return defaultValue()
	}
}
~~~

这个函数接受左侧的可选值T?和右侧的字符串。如果可选值不是nil，我们将它解包，然后返回它的字符串描述。否则，我们将传入的默认支付串返回。@autoclosure标注确保了只有当需要的时候，我们才会对第二个表达式进行求值。

> print("Body temperature: \(bodyTemperature ??? "n/a")") // Body temperature: 37.0

#### 可选值map

假设，我们要把第一个字符数组的第一个元素来转换成字符串:

~~~
let characters:[Character] = ["a", "b", "c"]
String(characters[0]) // a

extension Optional {
	func map<U>(transform:(Wrapped) -> U) -> U? {
		guard let value = self else {return nil}
		return transform(value)
	}
}
~~~

当你想要的就是一个可选值结果时，Optional.map就非常有用。假设你要为数组实现一个变种的reduce方法，这个方法不接受初始值，而是直接使用数组中的首个元素作为初始值(在一些语言中，这个函数可能被叫做reduce1，但是Swift里我们有重载，所以也将它叫做reduce就行了)。
因为数组可能会是空的，这种情况下没有初始值，结果只能是可选值，因为可选值为nil时，可选值map也会返回nil，所以我们可以不使用guard，而就用一个return来重写我们的这个reduce:
~~~
extension Array {
	func reduce_alt(_ nextPartialResult: (Element, Element) -> Element) -> Element? {
		return first.map {
			dropFirst().reduce($0.nextPartialResult)
		}
	}
}
~~~

#### 可选值flatMap

如果你对一个可选值调用map，但是你的转换函数本身也返回可选值的话，最终结果将是一个双重嵌套的可选值。举个例子，比如你想要获取数组的第一个字符串元素，并将它转换为数字。首先你使用数组上的first，然后用map将它转换为数字：

~~~
let stringNumber = ["1", "2", "3", "foo"]
let x = stringNumbers.first.map{ Int($0) } //Optional(Optional(1))
~~~

问题在于，map返回可选值(因为firs可能会是nil)，并且Int(String)也返回可选值(字符串可能不是一个整数)，最后x的结果将会是Int??。

flatMap可以把结果展评为单个可选值。这样一来，y的类型将会是Int？：
> let y = stringNumbers.first.flatMap{ Int($0) } //Optional(1)

~~~
extension Optional {
	func flatMap<U>(transform: (Wrapped) -> U?) ->U? {
		if let value = self, let transformed = transform(value) {
			return transformed
		}

		return nil
	}
}
~~~

#### 使用compactMap过滤nil

如果你的序列中包含可选值，可能你只会对那些非nil值感兴趣。实际上，你可能只想忽略掉它们。 设想在一个字符串数组中你只想要处理数字。在for循环中，用可选值模式匹配可以很简单地就实现：
~~~
let numbers = ["1", "2", "3", "foo"]
var sum = 0 
for case let i? in numbers.map({Int($0)}) {
	sum += i
}

sum //6
~~~

实际上，你想要的版本应该是一个可以将那些nil过滤出去并将非nil值进行解包的map。标准库中序列的compactMap正是你想要的：
> numbers.compactMap{ Int($0)}.refuce(0, +) // 6

为了实现我们自己compactMap，先转换整个数组，然后过滤出非nil值，最后解包每个被过滤出来的元素：
~~~
extension Sequence {
	func compactMap<B>(_ transform:(Element) -> B?) -> [B] {
		return lazy.map(transform).filter {$0 != nil}.map{$0!}
	}
}
~~~

在这个实现中，使用了lazy来将数组的实际创建推迟到了使用前的最后一刻。这可能只是一个小的优化，但在处理较大的序列时还是值得的。使用lazy可以避免多个作为中间结果的数组的内存分配。

#### 可选值判等

首先，只有当Wrapped类型实现了Equatable协议，那么Optional才会也实现Equatable协议：
~~~
extension Optional: Equatable where Wrapped: Equatable {
	static func ==(lhs: Wrapped?, rhs: Wrapped?) -> Bool {
		switch (lhs, rhs) {
			case (nil, nil): return true
			case let(x?, y?): return x==y
			case (_?, nil),(nil, _?): return false
		}
	}
}
~~~

Swift代码也一直依赖这个隐式转换。例如，使用键作为下标在字典中查找时，因为键有可能不存在，所以返回值是可选值。对于用下标读取和写入时，所需要的类型是相同的。也就是说，在使用下标进行赋值时，我们其实需要传入一个可选值。

附带提一句，如果你想知道当使用下标操作为字典的某个键赋值nil会发生什么的话，答案就是这个键会从字典中移除。有时候这会很有用，但是这也意味着你在使用字典存储可选值类型时需要小心一些:

~~~
var dictWithNils: [String: Int?] = [
	"one": 1,
	"two": 2,
	"none": nil
]

dictWithNils["two"] = nil
dictWithNils // ["one": Optional(1), "none": nil]

~~~

它会把"two这个键移除。

为了改变这个键的值，你可以使用下面中的任意一个。它们都可以正常工作:

~~~
dictWithNils["two"] = Optional(nil) 
dictWithNils["two"] = .some(nil) 
dictWithNils["two"]? = nil

dictWithNils // ["one": Optional(1), "none": nil, "two": nil]
~~~

### 强制解包的时机

当你能确定你的某个值不可能是nil时可以使用感叹号，你应当会希望如果它意外是nil的话，程序应当直接挂掉。

下面这段代码会根据特定的条件从字典中找到满足这个条件的值所对应的所有的键:
~~~
let ages = [
	"Tim": 53, "Angela": 54, "Craig":44, "Jony": 47, "Chris": 37, "Michael": 34,
]

ages.keys
	.filter { name in ages[name]! < 50 }
	.sorted()

// ["Chris", "Craig", "Jony", "Michael"]
~~~
同样，这里使用 ! 是非常安全的 — 因为所有的键都来源于字典，所以在字典中找不到这个键是 不可能的。

不过你也可以通过一些手段重写这几句代码，来把强制解包移除掉。利用字典本身是一个键值 对的序列这一特性，你可以对序列先进行过滤，然后再通过映射来返回键的序列:
~~~
ages.filter { (_, age) in age < 50 } .map { (name, _) in name } .sorted()
// ["Chris", "Craig", "Jony", "Michael"]

~~~

#### 改进强制解包的错误信息

其实，你可能会留一个注释来提醒为什么这里要使用强制解包。那为什么不把这个注释直接作 为错误信息呢?这里我们加了一个 !! 操作符，它将强制解包和一个更具有描述性质的错误信息 结合在一起，当程序意外退出时，这个信息也会被打印出来:

~~~
infix operator !!

func !!<T>(wrapped: T?, failureText: @autoclosure () => String) -> T {
	if let x = wrapped { return x }
	fatalError(failureText())
}
~~~

现在你可以写出更能描述问题的错误信息了，它还包括了你期望的被解包的值:

~~~
let s = "foo"
let i = Int(s) !! "Expecting integer, got \"\(s)\""
~~~

#### 在调试版本中进行断言

说实话，选择在发布版中让应用崩溃还是很大胆的行为。通常，你可能会选择在调试版本或者测试版本中进行断言，让程序崩溃，但是在最终产品中，你可能会把它替换成像是零或者空数组这样的默认值。

我们可以实现一个疑问感叹号 !? 操作符来代表这个行为。我们将这个操作符定义为对失败的解 包进行断言，并且在断言不触发的发布版本中将值替换为默认值

~~~
infix operator !?

func !?<T: ExpressibleByIntegerLiteral>(wrapped: T?,failureText:@autoclosure() -> String) -> T 
{
	assert(wrapped != nil, failureText())
	return wrapped ?? 0
}
~~~

现在，下面的代码将在调试时触发断言，但在发布版本中打印 0:

~~~
let s = "20"
let i = Int(s) !? "Expectiong integer, got \"\(s)\""
~~~

对其他字面量转换协议进行重载，可以覆盖不少能够有默认值的类型:

~~~
func !?<T: ExpressibleByArrayLiteral>(wrapped: T?, failureText: @autoclosure () -> String) -> T
{
	assert(wrapped != nil, failureText()) 
	return wrapped ?? []
}

func !?<T: ExpressibleByStringLiteral>(wrapped: T?, failureText: @autoclosure () -> String) -> T
{
	assert(wrapped != nil, failureText()) return wrapped ?? ""
}

~~~

如果你想要显式地提供一个不同的默认值，或者是为非标准的类型提供这个操作符，我们可以定义一个接受元组为参数的版本，元组包含默认值和错误信息:

~~~
func !?<T>(wrapped: T?, nilDefault: @autoclosure () -> (value: T, text: String)) -> T
{
	assert(wrapped != nil, nilDefault().text) return wrapped ?? nilDefault().value
}

// 调试版本中断言，发布版本中返回 5 
Int(s) !? (5, "Expected integer")
~~~

因为对于返回Void的函数，使用可选链进行调用时将返回Void？，所以利用这一点，你也可以写一个非泛型的版本来检测一个可选链调用碰到nil，且无操作的情况：
~~~

func !?(wrapped: ()?, failureText: @autoclosure () -> String) { 
	assert(wrapped != nil, failureText())
}

var output: String? = nil
output?.write("something") !? "Wasn't expecting chained nil here"

~~~

> 想要挂起一个操作我们有三种方式。首先，fatalError 将接受一条信息，并且无条件 地停止操作。第二种选择，使用 assert 来检查条件，当条件结果为 false 时，停止执 行并输出信息。在发布版本中，assert 会被移除掉，也就是说条件不会被检测，操作也永远不会挂起。第三种方式是使用 precondition，它和 assert 有一样的接口，但是 在发布版本中不会被移除，也就是说，只要条件被判定为 false，执行就会被停止。

### 隐式解包可选值

当可选值 是 nil 的时候强制解包会造成应用崩溃，那你到底为什么会要用到隐式可选值呢?好吧，实际上 有两个原因:

原因 1:暂时来说，你可能还需要到 Objective-C 里去调用那些没有检查返回是否存在的代码; 或者你会调用一个没有针对 Swift 做注解的 C 语言的库。

隐式解包可选值还存在的唯一原因其实是为了能更容易地和 Objective-C 与 C 一起使用

原因 2:因为一个值只是很短暂地为 nil，在一段时间后，它就再也不会是 nil。
最常见的情况就是两阶段初始化 (two-phase initialization)。当你的类准备好被使用时，所有 的隐式解包可选值都将有一个值。这就是 Xcode 和 Interface Builder 在 view controller 的生 命周期中使用它们的方式:在 Cocoa 和 Cocoa Touch 中，view controller 会延时创建他们的 view，所以在 view controller 自身已经被初始化，但是它的 view 还没有被加载的这段时间窗 口内，view 的对象的 outlet 引用还没有被创建。

#### 隐式可选值行为

虽然隐式解包的可选值在行为上就好像是非可选值一样，不过你依然可以对它们使用可选链， nil 合并，if let，map 或者将它们与 nil 比较，所有的这些操作都是一样的:
~~~
var s: String! = "Hello" 
s?.isEmpty // Optional(false) 
if let s = s { print(s) } // Hello 
s=nil
s ?? "Goodbye" // Goodbye
~~~

## 函数

要理解Swift中的函数和闭包，你需要切实弄明白三件事:
1、函数可以像Int或者String那样被赋值给变量，也可以作为另一个函数的输入参数，或者另一个函数的返回值来使用
2、函数能够捕获存在于其局部作用之外的变量
3、有两种方法可以创建函数，一种是使用func关键字，另一种是{}.在Swift中，后一种被称为闭包表达式。


### 函数可以被赋值给变量，也能够作为函数的输入和输出

为什么函数可以作为变量使用的这种能力如此关键呢？因为它让你很容易写出"高阶"函数，高阶函数将函数作为参数的能力使得它们在很多方面都非常有用。

### 函数可以捕获存在于它们作用域之外的变量

当函数引用了在其作用域之外的变量时，这个变量就被捕获了，它们将继续存在，而不是在超过作用域后被摧毁。

每次调用这个函数时，计数器将会增加：
~~~
func counterFunc() -> (Int) -> String { 
	var counter = 0
	func innerFunc(i: Int) -> String {
		counter += i // counter 被捕获
		return "Running total: \(counter)" }
	return innerFunc 
}
~~~

一般来说，因为counter是counterFunc的局部变量，它在return语句执行之后就应该离开作用域并被摧毁。但因为innerFunc捕获了它，所以Swift运行时将一直保证它的存在，直到捕获它的函数被销毁为止。

~~~
let f = counterFunc()
f(3) // Running total: 3
f(4) // Running total: 7
~~~

如果我们再次调用counterFunc()函数，将会生成并捕获一个新的counter变量:
~~~
let g = counterFunc() 
g(2) // Running total: 
2 g(2) // Running total: 4
~~~

你可以将这些函数以及它们所捕获的变量想象为一个类的实例，这个类拥有一个单一的方法(也就是这里的函数)以及一些成员变量(这里的被捕获的变量)。

在编程术语里，一个函数和它所捕获的变量环境组合起来称为闭包。上面f和g都是闭包的例子，因为它们捕获并使用了一个在它们作用域之外的非局部变量counter。

### 函数可以使用{}来声明为闭包表达式

在Swift中，定义函数的方法有两种。一种是使用func关键字。另一种方式是使用闭包表达式。

~~~
func doubler(i:Int) -> Int {
	return i * 2
}

[1, 2, 3, 4].map(doublerAlt) // [2, 4, 6, 8]
~~~

使用闭包表达式的语法来写相同的函数，像之前那样将它传给map:

~~~
let doublerAlt = { (i: Int) -> Int in return i*2 }
[1, 2, 3, 4].map(doublerAlt) // [2,4,6,8]
~~~

使用闭包表达式来定义的函数可以被想成函数的字面量，就像1是整数字面量，"hello"是字符串字面量那样。与func相比，它的区别在于闭包表达式是匿名的，它们没有被赋予一个名字。使用它们的方式只能是在它们被创建时将其赋值给一个变量，或者是将它们传递给另一个函数或方法。

其实还有第三种使用匿名函数的方法: 你可以在定义一个表达式的同时，对它进行调用。这个方法在定义那些初始化代码多于一行的属性时会很有用

Swift中的可以让代码更加简洁的特性：

- 如果你将闭包作为参数传递，并且你不再使用这个闭包做其他事情的话，就没有必要先将它存储到一个局部变量中。可以想象一下比如5*i这样的数值表达式，你可以把它直接传递给一个接受Int的函数，而不必先将它计算并存储到变量里。
- 如果编译器可以从上下文中推断出类型的话，你就不需要指明它了。
- 如果闭包表达式的主体部分只包括一个单一的表达式的话，它将自动返回这个表达式的结果，你可以不写return，
- Swift会自动为函数的参数提供简写形式，$0代表第一个参数，$1代表第二个参数，以此类推。
- 如果函数的最后一个函数是闭包表达式的话，你可以将这个闭包表达式移动函数调用的圆括号的外部。这样的尾随闭包语法在多行闭包表达式中表现非常好，因为它看起来更接近于装配了一个普通的函数定义，或者是像if(expr) {} 这样的执行块的表达形式。
- 如果一个函数除了闭包表达式外没有别的参数，那么调用的时候在方法名后面圆括号也可以一并省略。

~~~
[1, 2, 3].map( { (i: Int) -> Int in return i * 2 } ) [1, 2, 3].map( { i in return i * 2 } )
[1, 2, 3].map( { i in i * 2 } )
[1, 2, 3].map( { $0 * 2 } )
[1, 2, 3].map() { $0 * 2 } 
[1,2,3].map{$0*2}
~~~

如果你在尝试提供闭包表达式时遇到一些谜一样的错误的话，将闭包表达式写成上面的例子中第一种包括类型的完整形式，往往是个好主意。

还有一些时候，Swift会要求你用更明确的方式进行调用。假设你要得到一个随机数数组，一种快读的方法就是通过Range.map方法，并在map的函数中生成并返回随机数。这里，无论如何你都要为map的函数提供一个参数。或者明确使用_告诉编译器你承认这里有一个参数，但并不关心它究竟是什么:
~~~
(0..<3).map{_ in Int.random(in: 1..<100)} //[53,63,88]
~~~

#### 函数的灵活性

我们从定义一个Person类型开始。因为我们想要展示Objective-C强大的运行时的工作方式，所以我们将它定义为NSObject的子类。我们将这个类标记为@objcMembers，这样它的所有成员都将在Objective-C中可见:

~~~
@objcMembers
final class Persion: NSObject {
	let first: String
	let last: String
	let yearOfBirth: Int
	init(first: String, last: String, yearOfBirth: Int) {
		self.first = first
		self.last = last
		self.yearOfBirth = yearOfBirth
		//super.init() 在这里被隐式调用
	}
}
~~~

接下来我们定义一个数组，其中包含了不同名字和出生年份的人：
~~~
let people = [
	Person(first: "Emily", last: "Young", yearOfBirth: 2002), 
	Person(first: "David", last: "Gray", yearOfBirth: 1991), 
	Person(first: "Robert", last: "Barnes", yearOfBirth: 1985), 
	Person(first: "Ava", last: "Barnes", yearOfBirth: 2000), 
	Person(first: "Joanne", last: "Miller", yearOfBirth: 1994), 
	Person(first: "Ava", last: "Barnes", yearOfBirth: 1998),
]
~~~

我们想要对着个数组进行排序，规则是先按照姓排序，再按照名排序，最后是出生年份。排序应该遵照用户的区域设置。我们可以使用NSSortDescriptor对象来描述如何排序对象， 通过它可以表达出各个排序标准(使用localizedStandardCompare来进行遵循区域设置的排序):
~~~

let lastDescriptor = NSSortDescriptor(key: #keyPath(Person.last), ascending:true,selector:#selector(NSString.localizedStandardCompare(_:)))
let firstDescriptor = NSSortDescriptor(key: #keyPath(Person.first), ascending: true,
selector: #selector(NSString.localizedStandardCompare(_:)))
let yearDescriptor = NSSortDescriptor(key: #keyPath(Person.yearOfBirth), ascending: true)
~~~

要对数组进行排序，我们使用NSArray的sortedArray(using:)方法。这个方法可以接受一系列排序描述符。为了确定两个元素的顺序，它会先使用第一个描述符，并检查其结果。如果两个元素在第一个描述符下相同，那么它将使用第二个描述符，以此类推：
~~~
let descriptors = [lastDescriptor, firstDescriptor, yearDescriptor] (people as NSArray).sortedArray(using: descriptors)
/*
[Ava Barnes (1998), Ava Barnes (2000), Robert Barnes (1985),
David Gray (1991), Joanne Miller (1994), Emily Young (2002)] */

~~~

排序描述符用到了Objective-C的两个运行时特性：首先，key是Objective-C的键路径，它其实是一个包含属性名称的链接。不要把它和Swift4引入的原生的(强类型的)键路径搞混。

如何用Swift的sort来复制这个功能呢？要复制这个排序的部分功能是很简单的:
~~~
var strings = ["Hello", "hallo", "Hallo", "hello"]
strings.sort { $0.localizedStandardCompare($1) == .orderedAscending }
strings //["hallo", "Hallo", "hello", "Hello"]
~~~

如果只是想用对象的某一个属性进行排序的话，也很简单:
~~~
people.sorted { $0.yearOfBirth < $1.yearOfBirth }
~~~

要同时排序姓和名，我们可以用标准库的lexicographicallyPrecedes方法来进行实现。这个方法接受两个序列，并对它们执行一个电话簿方式的比较，也就是说，这个比较将顺次从两个序列中各取一个元素来进行比较，知道发现不相等的元素。所以，我们可以用姓和名构建两个数组，然后使用lexcicongraphicallyPrecedes来比较它们。我们还需要一个函数来执行这个比较，这里我们把使用了localizedStandardCompare的比较代码放到这个函数中:
~~~
people.sorted { p0, p1 in
	let left = [p0.last, p0.first]
	let right = [p1.last, p1.first]
	return left.lexicographicallyPrecedes(right) {
		$0.localizedStandardCompare($1) == .orderedAscending
	}
}
~~~


## 函数作为数据

我们不会选择去写一个更复杂的函数来进行排序，先回头看看现状。排序描述符的方式要清晰不少，但是它用到了运行时编程。我们写的函数没有使用运行时编程，不过它们不太容易写出来或者读懂。

除了把排序信息存储在类里，我们可以定义一个描述对象顺序的函数。其中，最简单的一种实现就是接受两个对象作为参数，并在它们顺序正确的时候，返回true。这个函数的类型正是标准库中sort(by:)和sorted(by:)的参数类型。接下来，让我们先定义一个泛型别名来表达这种函数形式的排序描述符:
~~~
// 一个排序断言，当第一个值应当在第二个值之前时，返回'true'
typealias SortDescriptor<Root> = (Root, Root) -> Bool
~~~

现在，就可以用这个别名定义比较Person对象的排序描述符了。它可以比较出生年份，也可以比较姓的字符串:
~~~
let sortByYear:SortDescriptor<Person> = { $0.yearOfBirth < $1.yearOfBirth }
let sortByLastName:SortDescriptor<Persion> = {
	$0.last.localizedStandarCompare($1.last) == .orderedAscending
} 
~~~

这个函数的第一个参数是一个名为key的函数，此函数接受一个正在排序的数组的元素，并返回这个排序描述符所处理的属性的值。然后，我们使用第二个参数areInIncreasingOrder比较key返回的结果。最后，用SortDescriptor把这两个参数包装一下，就是要返回的排序描述符了:
~~~
//key函数，根据输入的参数返回要进行比较的元素
//by进行比较的断言
//通过用by比较key返回值的方式构建SortDescriptor函数

func sortDescriptor<Root, Value> (
	key: @escaping(Root) -> Value,
	by areInIncreasingOrder: @escaping(Value, Value) -> Bool)
	-> SortDescriptor<Root>
{
	return { areInIncreasingOrder(key($0), key($1))}
}
~~~

key函数描述了如何深入一个Root类型的元素，并提取出一个和特定排序步骤相关的Value类型的值。因为借鉴了泛型参数名字Root和Value，所以它和Swift4引入的Swift原生键路径有很多相同之处。

有了这个函数，我们就可以用另外一种方式来定义sortByYear了：
~~~
let sortByYearAlt:SortDescriptor<Person> = sortDescriptor(key:{$0.yearOfBirth}, by: <)
people.sorted(by: sortByYearAlt)
~~~

甚至，我们还可以为所有实现了Comparable的类型定义一个重载版本：
~~~
func sortDescriptor<Root, Value>(key: @escaping(Root) -> Value) -> SortDescriptor<Root> where Value:Comparable {
	return { key($0) < key($1)}
}

let sortByYearAlt2:SortDescriptor<Person> = sortDescriptor(key: { $0.yearOfBirth })

~~~

这两个sortDescriptor都使用了返回布尔值的排序函数，因为这是标准库中对于比较断言的约定。但另一方面，Foundation中像是localizedStandardCompare这样的API，返回的却是一个包含升序，降序，相等的ComparisonResult。给sortDescriptor增加这种支持也是很简单：
~~~
func sortDescriptor<Root, Value>(
key: @escaping (Root) -> Value,
ascending: Bool = true,
by comparator: @escaping (Value) -> (Value) -> ComparisonResult) -> SortDescriptor<Root>
{
return { lhs, rhs in
let order: ComparisonResult = ascending ? .orderedAscending
: .orderedDescending
return comparator(key(lhs))(key(rhs)) == order }
}

~~~

## 函数作为代理

### 结构体上实现代理
