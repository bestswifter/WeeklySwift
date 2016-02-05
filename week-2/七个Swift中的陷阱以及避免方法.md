Swift正在完成一个惊人的壮举，它正在改变我们在苹果设备上编程的方式，引入了很多现代范例，例如：函数式编程和相比于OC这种纯面向对象语言更丰富的类型检查。

Swift语言希望通过采用安全的编程模式去帮助开发者避免bug。然而这也会不可避免的产生一些人造的陷阱，他们会在编译器不报错的情况下引入一些Bug。这些陷阱有的已经在[Swift book](https://swift.org/documentation/#the-swift-programming-language)中提到，有一些还没有。这里有七个我在去年遇到的陷阱，它们涉及Swift协议扩展、可选链和函数式编程。

## 协议扩展：强大但是需要谨慎使用

一个Swift类可以去继承另一个类，这种能力是强大的。继承将使类之间的特定关系更加清晰，并且支持细粒度代码分享。但是，Swift中如果不是引用类型的话（如：结构体、枚举），就不能具有继承关系。然而，一个值类型可以继承协议，同时协议可以继承另一个协议。虽然协议除了类型信息外不能包含其他代码，但是协议扩展（`protocol extension`）可以包含代码。照这种方式，我们可以用继承树来实现代码的分享共用，树的叶子是值类型（结构体或枚举类），树的内部和根是协议和与他们对应的扩展。

但是Swift协议扩展的实现依然是一片新的、未开发的领域，尚存在一些问题。代码并不总是按照我们期望的那样执行。因为这些问题出现在值类型（结构体与枚举）与协议组合使用的场景下，我们将使用类与协议组合使用的例子去说明这种场景下不存在陷阱。当我们重新改为使用值类型和协议的时候将会发生令人惊奇的事。

开始介绍我们的例子：`classy pizza`

假设这里有使用两种不同谷物制作的三种`Pizza`：
	
	enum Grain  { case Wheat, Corn }
	
	class  NewYorkPizza  { let crustGrain: Grain = .Wheat }
	class  ChicagoPizza  { let crustGrain: Grain = .Wheat }
	class CornmealPizza  { let crustGrain: Grain = .Corn  }


我们可以通过`crustGrain `属性取得披萨所对应的原料

	NewYorkPizza().crustGrain 	// returns Wheat
    ChicagoPizza().crustGrain 	// returns Wheat
	CornmealPizza().crustGrain 	// returns Corn
	
因为大多数的`Pizza`是用小麦(`wheat`)做的，这些公共代码可以放进一个超类中作为默认执行的代码。

	enum Grain { case Wheat, Corn }
	
	class Pizza {
        var crustGrain: Grain { return .Wheat }
        // other common pizza behavior
	}
	class NewYorkPizza: Pizza {}
	class ChicagoPizza: Pizza {}

这些默认的代码可以被重载去处理其它的情况（用玉米制作）

	class CornmealPizza: Pizza {
        override var crustGain: Grain { return .Corn }
	}

哎呀！这代码是错的，并且很幸运的是编译器发现了这些错误。你能发现这个错误么？我们在第二个crustGain中少写了`r`。Swift通过显式的标注`override`避免这种错误。比如在这个例子中，我们用到了`override`，但是拼写错误的"crustGain"其实并没有重写任何属性，下面是修改后的代码：

	class CornmealPizza: Pizza {
   		 override var crustGrain: Grain { return .Corn }
	}
	
现在它可以通过编译并成功运行：

	NewYorkPizza().crustGrain 		// returns Wheat
	ChicagoPizza().crustGrain 		// returns Wheat
	CornmealPizza().crustGrain 		// returns Corn
	
同时`Pizza`超类允许我们的代码在不知道`Pizza`具体类型的时候去操作`pizzas`。我们可以声明一个`Pizza`类型的变量。
	
	var pie: Pizza
但是通用类型`Pizza`仍然可以去得到特定类型的信息。

	pie =  NewYorkPizza();		pie.crustGrain	 // returns Wheat
	pie =  ChicagoPizza();  	pie.crustGrain	 // returns Wheat
	pie = CornmealPizza();  	pie.crustGrain	 // returns Corn

Swift的引用类型在这个Demo中工作的很好。但是如果这个程序涉及到并发性、竞争条件，我们可以使用值类型来避免这些。让我们来试一下值类型的Pizza吧！

这里和上面一样简单，只需要把class修改为struct即可：
	
	enum Grain { case Wheat, Corn }
	
	struct  NewYorkPizza 	{ let crustGrain: Grain = .Wheat }
	struct  ChicagoPizza 	{ let crustGrain: Grain = .Wheat }
	struct CornmealPizza 	{ let crustGrain: Grain = .Corn  }
	
执行

	NewYorkPizza()	.crustGrain 	// returns Wheat
    ChicagoPizza()	.crustGrain 	// returns Wheat
	CornmealPizza()	.crustGrain 	// returns Corn
	
当我们使用引用类型的时候，我们通过一个超类Pizza来达到目的。但是对于值类型将要求一个协议和一个协议扩展来合作完成。

	protocol Pizza {}

	extension Pizza {  var crustGrain: Grain { return .Wheat }  }
	
	struct  NewYorkPizza: Pizza { }
	struct  ChicagoPizza: Pizza { }
	struct CornmealPizza: Pizza {  let crustGain: Grain = .Corn }
	
这段代码可以通过编译，我们来测试一下：

	NewYorkPizza().crustGrain 		// returns Wheat
	 ChicagoPizza().crustGrain 		// returns Wheat
	CornmealPizza().crustGrain 		// returns Wheat  What?!

对于执行结果，我们想说`cornmeal pizza`并不是`Wheat`制作的，返回结果出现错误！哎呀！我把
`struct CornmealPizza: Pizza {  let crustGain: Grain = .Corn }`
中的	`crustGrain`写成了`crustGain`，再一次忘记了`r`，但是对于值类型这里没有`override`关键字去帮助编译器去发现我们的错误。没有编译器的帮助，我们不得不更加小心的编写代码。

> **⚠️ 在协议扩展中重写协议中的属性时要仔细核对**

ok，我们把这个拼写错误改正过来：

	struct CornmealPizza: Pizza {  let crustGrain: Grain = .Corn }

重新执行

	 NewYorkPizza().crustGrain 		// returns Wheat
	 ChicagoPizza().crustGrain 		// returns Wheat
	 CornmealPizza().crustGrain 	// returns Corn  Hooray!	

为了在讨论`Pizza`的时候不需要担心到底是`New York`, `Chicago`, 还是 `cornmeal`，我们可以使用`Pizza`协议作为变量的类型。

	var pie: Pizza
	
这个变量能够在不同种类的`Pizza`中去使用

	pie =  NewYorkPizza(); pie.crustGrain  // returns Wheat
	pie =  ChicagoPizza(); pie.crustGrain  // returns Wheat
	pie = CornmealPizza(); pie.crustGrain  // returns Wheat    Not again?!

为什么这个程序显示`cornmeal pizza` 包含`wheat`？Swift编译代码的时候忽略了变量的目前实际值。代码只能够使用编译时期的知道的信息，并不知道运行时期的具体信息。程序中可以在编译时期得到的信息是`pie`是`pizza`类型，`pizza`协议扩展返回`wheat`，所以在结构体`CornmealPizza`中的重写起不到任何作用。虽然编译器本能够在使用静态调度替换动态调度时，为潜在的错误提出警告，但它实际上并没有这么做。这里的粗心将带来巨大的陷阱。

在这种情况下，Swift提供一种解决方案，除了在协议扩展中（`extension`）定义`crustGrain `属性之外，还可以在协议中声明。
	
	protocol  Pizza {  var crustGrain: Grain { get }  }
	extension Pizza {  var crustGrain: Grain { return .Wheat }  }

在协议内声明变量并在协议拓展中定义，这样会告诉编译器关注变量`pie`运行时的值。

在协议中一个属性的声明有两种不同的含义，静态还是动态调度，取决于是否这个属性在协议扩展中定义。

补充了协议中变量的声明后，代码可以正常运行了：

	pie =  NewYorkPizza();  pie.crustGrain	 // returns Wheat
	pie =  ChicagoPizza();  pie.crustGrain	 // returns Wheat
	pie = CornmealPizza();  pie.crustGrain	 // returns Corn    Whew!


>⚠️ **在协议扩展中定义的每一个属性，需要在协议中进行声明**

然而这个设法避免陷阱的方式并不总是有效的。

**导入的协议不能够完全扩展。**

框架（库）可以使一个程序导入接口去使用，而不必包含相关实现。例如苹果提供给我们提供了需要框架，实现了用户体验、系统设施和其他功能。Swift的扩展允许程序向导入的类、结构体、枚举和协议中添加自己的属性(这里的属性并不是存储属性)。通过协议拓展添加的属性，就好像它原来就在协议中一样。但实际上定义在协议拓展中的属性并非一等公民，因为通过协议拓展无法添加属性的声明。


我们首先实现一个框架，这个框架定义了Pizza协议和具体的类型

	// PizzaFramework:
	
	public protocol Pizza { }

	public struct  NewYorkPizza: Pizza  { public init() {} }
	public struct  ChicagoPizza: Pizza  { public init() {} }
	public struct CornmealPizza: Pizza  { public init() {} }

导入框架并且扩展Pizza

	import PizzaFramework

	public enum Grain { case Wheat, Corn }
	
	extension Pizza         { var crustGrain: Grain { return .Wheat	} }
	extension CornmealPizza { var crustGrain: Grain { return .Corn	} }
	
和以前一样，静态调度产生一个错误的答案

	var pie: Pizza = CornmealPizza()
	pie.crustGrain                            // returns Wheat   Wrong!

这个是因为（与刚才的解释一样）这个`crustGrain`属性并没有在协议中声明，而是只是在扩展中定义。然而，我们没有办法对框架的代码进行修改，因此也就不能解决这个问题。因此，想要通过扩展增加其他框架的协议属性是不安全的。

> ⚠️ **不要对导入的协议进行扩展，新增可能需要动态调度的属性**


正像刚才描述的那样，框架与协议扩展之间的交互，限制了协议扩展的效用，但是框架并不是唯一的限制因素，同样，类型约束也不利于协议扩展。

**Attributes in restricted protocol extensions: declaration is no longer enough**



回顾一下此前Pizza的例子：

	enum Grain { case Wheat, Corn }

	protocol  Pizza { var crustGrain: Grain { get }  }
	extension Pizza { var crustGrain: Grain { return .Wheat }  }
	
	struct  NewYorkPizza: Pizza  { }
	struct  ChicagoPizza: Pizza  { }
	struct CornmealPizza: Pizza  { let crustGrain: Grain = .Corn }

让我们用Pizza做一顿饭。不幸的是，并不是每顿饭都会吃pizza，所以我们使用一个通用的`Meal`结构体来适应各种情况。我们只需要传入一个参数就可以确定进餐的具体类型。
	
	struct Meal: MealProtocol {
   		let mainDish: MainDishOfMeal
	}

结构体`Meal`继承自`MealProtocol`协议，它可以测试`meal`是否包含谷蛋白。

	protocol MealProtocol {
	    typealias MainDish_OfMealProtocol
	    var mainDish: MainDish_OfMealProtocol {get}
	    var isGlutenFree: Bool {get}
	}
	
为了避免中毒，代码中使用了默认值（不含有谷蛋白）

	extension MealProtocol {
	    var isGlutenFree: Bool  { return false }
	}

Swift中的 `Where`提供了一种方式去表达约束性协议扩展。当主菜是`pizza`的时候，我们知道`pizza`有`scrustGrain`属性，我们就可以访问这个属性。如果没`where`这里的限制，我们在不是`Pizza`的情况下访问`scrustGrain`是不安全的。

	extension MealProtocol  where  MainDish_OfMealProtocol: Pizza {
	    var isGlutenFree: Bool  { return mainDish.crustGrain == .Corn }
	}

一个带有`Where`的扩展叫做约束性扩展。

让我们做一份美味的`cornmeal Pizza`

	let meal: Meal = Meal(mainDish: CornmealPizza())

结果：
	
	meal.isGlutenFree	// returns false
	// 根据协议拓展，理论上应该返回true	

正像我们在前面小节演示的那样，当发生动态调度的时候，我们应该在协议中声明，并且在协议扩展中进行定义。但是约束性扩展的定义总是静态调度的。为了防止由于意外的静态调度而引起的bug：

> ⚠️ **如果一个新的属性需要动态调度，避免使用约束性协议扩展**

## 使用可选链赋值和副作用

Swift可以通过静态地检查变量是否为`nil`来避免错误，并使用一种方便的缩略表达式，可选链，用于忽略可能出现的`nil`。这一点也正是Objective-C的默认行为。

不幸的是，如果可选链中被赋值的引用有可能为空，就可能导致错误，考虑下面这段代码，`Holder`中存放一个整数：

```swift
class Holder  {
    var x = 0
}

var n = 1
var h: Holder? = nil
h?.x = n++
```

在这段代码的最后一行中，我们把`n++`赋值给h的属性。除了赋值以外，变量n还会自增，我们称此为**副作用**。

变量n最终的值会取决于h是否为nil。如果h不为nil，那么赋值语句执行，`n++`也会执行。但如果h为nil，不仅赋值语句不会执行，`n++`也不会执行。为了避免没有发生副作用导致的令人惊讶的结果，我们应该：

> ⚠️ **避免把一个有副作用的表达式的结果通过可选链赋值给等号左边的变量**

## 函数编程陷阱

由于Swift的支持，函数式编程的优点得以被带入苹果的生态圈中。Swift中的函数和闭包都是一等公民，不仅方便易用而且功能强大。不幸的是，其中也有一些我们需要小心避免的陷阱。

比如，inout参数会在闭包中默默的失效。

Swift的inout参数允许函数接受一个参数并直接对参数赋值，Swift的闭包支持在执行过程中引用被捕获的函数。这些特性有助于我们写出优雅易读的代码，所以你也许会把它们结合起来使用，但这种结合有可能会导致问题。

我们重写`crustGrain `属性来说明inout参数的使用，为简单起见，开始时先不使用闭包：

```swift
enum Grain {
    case Wheat, Corn
}

struct CornmealPizza {
    func setCrustGrain(inout grain: Grain)  {
        grain = .Corn
    }
}
```

为了测试这个函数，我们给它传一个变量作为参数。函数返回后，这个变量的值应该从Wheat变成了Corn：

```swift
let pizza = CornmealPizza()
var grain: Grain = .Wheat
pizza.setCrustGrain(&grain)
grain		// returns Corn
```

现在我们尝试在函数中返回闭包，然后在闭包中设置参数的值：

```swift
struct CornmealPizza {
    func getCrustGrainSetter() -> (inout grain: Grain) -> Void {
        return { (inout grain: Grain) in
            grain = .Corn
        }
    }
}
```

使用这个闭包只需要多一次调用：

```swift
var grain: Grain = .Wheat
let pizza = CornmealPizza()
let aClosure = pizza.getCrustGrainSetter()
grain			// returns Wheat (We have not run the closure yet)
aClosure(grain: &grain)
grain			// returns Corn
```

到目前为止一切正常，但如果我们直接把参数传进`getCrustGrainSetter`函数而不是闭包呢？

```swift
struct CornmealPizza {
    func getCrustGrainSetter(inout grain: Grain)  ->  () -> Void {
        return { grain = .Corn }
    }
}
```

然后再试一次：

```swift
var grain: Grain = .Wheat
let pizza = CornmealPizza()
let aClosure = pizza.getCrustGrainSetter(&grain)
print(grain)				// returns Wheat (We have not run the closure yet)
aClosure()
print(grain)				// returns Wheat  What?!?
```

inout参数在传入闭包的作用域外时会失效，所以：

> ⚠️ **避免在闭包中使用inout参数**

这个问题在Swift文档中提到过，但还有一个与之相关的问题值得注意，这与创建的闭包的等价方法：柯里化有关。

在使用柯里化技术时，inout参数显得前后矛盾。

在一个创建并返回闭包的函数中，Swift为函数的类型和主体提供了一种简洁的语法。尽管这种柯里化看上去仅是一种缩略表达式，但它与inout参数结合使用时却会给人们带来一些惊讶。为了说明这一点，我们用柯里化语法实现上面那个例子。函数没有声明为返回一个闭包，而是在第一个参数列表后加上了第二个参数列表，然后在函数体内省略了显式的闭包创建：

```swift
struct CornmealPizza {
    func getCrustGrainSetterWithCurry(inout grain: Grain)() -> Void {
        grain = .Corn
    }
}
```

和显式创建闭包时一样，我们调用这个函数然后返回一个闭包：

```swift
var grain: Grain = .Wheat
let pizza = CornmealPizza()
let aClosure = pizza.getCrustGrainSetterWithCurry(&grain)
```

在上面的例子中，闭包被显式创建但没能成功为inout参数赋值，但这次就成功了：

```swift
aClosure()
grain				// returns Corn
```

这说明在柯里化函数中，inout参数可以正常使用，但是显式的创建闭包时就不行了。

> ⚠️ **避免在柯里化函数中使用inout参数，因为如果你后来将柯里化改为显式的创建闭包，这段代码就会产生错误**

## 总结：七个避免

* 在协议扩展中重写协议中的属性时要仔细核对
* 在协议扩展中定义的每一个属性，需要在协议中进行声明
* 不要对导入的第三方协议进行属性扩展，那样可能需要动态调度
* 如果一个新的属性需要动态调度，避免使用约束性协议扩展
* 避免把一个有副作用的表达式的结果通过可选链赋值给等号左边的变量
* 避免在闭包中使用inout参数
* 避免在柯里化函数中使用inout参数，因为如果你后来将柯里化改为显式的创建闭包，这段代码就会产生错误
