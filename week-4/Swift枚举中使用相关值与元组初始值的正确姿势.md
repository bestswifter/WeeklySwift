>文章来源简书：http://www.jianshu.com/p/fc55450d133c

### 实现枚举相关值的Equatable协议
最近，在Swift中，我遇到了一些需要比较枚举中两个实例是否相等的情况。可是，这些枚举中用到了相关值（`associated values`）。乍一看，我好像没有直接的方法去解决这个问题。

你可能知道，如果你想要去比较一种类型的两个实例，那么，这种类型必须遵守`Equatable`协议。换句话说，你必须在这种类型中定义＝＝操作符。如果枚举没有相关值（`associated values`）或者它有个原始值，Swift 标准库中已经定义了＝＝操作符。例如：

	enum Math: Double {
	    case Pi = 3.1415
	    case Phi = 1.6180
	    case Tau = 6.2831
	}
	
	Math.Pi == Math.Pi // true
	Math.Tau == Math.Phi // false
	Math.Tau != Math.Phi // true
	
	// Without a raw-value
	enum CompassPoint: Equatable {
	    case North
	    case South
	    case East
	    case West
	}
	
	CompassPoint.North == CompassPoint.North // true
	CompassPoint.South == CompassPoint.East // false

我们正在比较的Case可以开箱即用，因为含有原始值的枚举类型已经隐式遵守了`RawRepresentable`协议。Swift标准库提供了满足通用范型与`RawRepresentable`类型的＝＝操作符的实现。

	// Used to compare 'Math' enum
	func ==<T : RawRepresentable where T.RawValue : Equatable>(lhs: T, rhs: T) -> Bool
	
	// Used to compare 'CompassPoint' enum
	func ==<T : Equatable>(lhs: T?, rhs: T?) -> Bool
	
我们可以轻松的看透其中的原理。对于`RawRepresentable`类型，只要`rawValue`遵守`Equatable`协议，那么这个函数要做的是比较每个类型的原始值。如果没有原始值，那么不同枚举成员的值就是它们自身。但是，如果枚举类型具有相关值（`associated values`）那么必须自己去实现＝＝操作符。考虑下面的例子：

	enum Barcode {
	    case UPCA(Int, Int)
	    case QRCode(String)
	    case None
	}
	
	// Error: binary operator '==' cannot be applied to two Barcode operands
	Barcode.QRCode("code") == Barcode.QRCode("code")

如果你对Swift的模式匹配精通的话，那么遵循`Equatable`协议是很简单的。

	extension Barcode: Equatable {
	}
	
	func ==(lhs: Barcode, rhs: Barcode) -> Bool {
	    switch (lhs, rhs) {
	    case (let .UPCA(codeA1, codeB1), let .UPCA(codeA2, codeB2)):
	        return codeA1 == codeA2 && codeB1 == codeB2
	
	    case (let .QRCode(code1), let .QRCode(code2)):
	        return code1 == code2
	
	    case (.None, .None):
	        return true
	
	    default:
	        return false
	    }
	}
	
	Barcode.QRCode("code") == Barcode.QRCode("code") // true
	Barcode.UPCA(1234, 1234) == Barcode.UPCA(4567, 7890) // false
	Barcode.None == Barcode.None // true
	
即使对于Swift2.2，语法仍然是有点难读、难记。我们必须对每个case进行模式匹配，然后拿出其中的相关值进行直接的比较。

现在我们就可以比较我们的自定义枚举了！

### 在枚举类型中如何使用元组作为原始值呢？

可以像下面的代码一样使用原始值去定义一个枚举类型吗？


	enum ErrorCode: (Int, String) {
	    case Generic_Error = (0, "Unknown")
	    case DB_Error = (909, "Database")
	}
	
答案是不可以的，原始值类型必须满足一个条件就是遵守`Equatable`协议。

那么，如果我们就是想要达到刚才代码的效果，可以怎么做呢？

**找到的替代方法一：**

	struct ErrorCode {
	    static let Generic_Error = (0, "Unknown")
	    static let DB_Error = (909, "Database")
	}
现在我们可以在整个项目中使用像`ErrorCode.Generic_Erro`这样的值。

**找到的替换方案二：**

	enum ErrorCode: Int{
	    case Generic = 0
	    case DB = 909
	    
	    var description: String {
	        switch self {
	        case .Generic:
	            return "Unknown"
	        case .DB:
	            return "Database"
	        }
	    }
	}


可以按照自己的场景选择合适的方案。