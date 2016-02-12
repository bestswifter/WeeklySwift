本文翻译自[Swift: CGRect, CGSize & CGPoint](https://medium.com/swift-programming/swift-cgrect-cgsize-cgpoint-5f4196da9cf8#.hjh2zzp0j)

在我转向Swift后，我逐渐避免写出具有OC风格的swift代码并开始真正利用上这门语言的优点。

但最近我发现在处理`CGGeometry`结构体时，我依然使用了丑陋的，非Swift风格的代码。`CGGeometry`结构体指的是：

> CGRect, CGSize, CGPoint

## C风格语法，披着狼皮的羊

我有一种强烈的感觉，许多Swift程序员都会对此感到内疚，请写过下面这种代码的同学举手：

```swift
let rect = CGRectMake(0, 0, 100, 100)
let point = CGPointMake(0, 0)
let size = CGSizeMake(100, 100)
```

别担心，这并不是什么应该羞愧的事情。

这么写的问题在于它不符合Swift的语法风格，虽然可以正常运行，但是它看上去更像是一段OC，甚至是Java代码。

一个有经验的iOS开发者一眼就能看出这几行代码的含义，他们不需要`CGGeometry `结构体的初始化函数中的参数名。但Swift希望对所有新手很友好，如果他们第一眼看到这些代码一定会不知所措，因为他们无法理解这些数字的含义。因此，我们应该使用Swift版本的代码：

```swift
let rect = CGRect(x: 0, y: 0, width: 100, height: 100)
let size = CGSize(width: 100, height: 100)
let point = CGPoint(x: 0, y: 0)
```

尽管代码变得略微冗长一些，但是它现在具备良好的可读性。而且另一个额外的好处在于，参数不必局限于`CGFloat`类型了，我们还可以使用`Int`和`Double`类型的参数。查看一下`CGRect`结构体的定义和`CGRectMake`函数的定义就很容理解了：

```swift
extension CGRect {
    public init(x: CGFloat, y: CGFloat, width: CGFloat, height: CGFloat)
    public init(x: Double, y: Double, width: Double, height: Double)
    public init(x: Int, y: Int, width: Int, height: Int)
}

public func CGRectMake(x: CGFloat, 
					 _ y: CGFloat, 
				 _ width: CGFloat, 
				_ height: CGFloat) -> CGRect

```

## Zero

你或许还在使用这样的代码：

```swift
let rect = CGRectZero
let size = CGSizeZero
let point = CGPointZero
```

是时候更新一下自己的知识体系，改用swift风格的语法了。别担心，新的语法只需要增加一个字符：

```swift
let rect = CGRect.zero
let size = CGSize.zero
let point = CGPoint.zero
```

这样写的好处在于Xcode的代码高亮机制会将`.zero`高亮显示，这样它会更醒目，降低你的认知负荷。

## 获取值

如果你曾经或任然是一名优秀的OC开发者，你应该写过这样的代码来获取结构体中的值：

```swift
CGRect frame = CGRectMake(0, 0, 100, 100)
CGFloat width = CGRectGetWidth(frame)
CGFloat height = CGRectGetHeight(frame)
CGFloat maxX = CGRectGetMaxX(frame)
CGFloat maxY = CGRectGetMaxY(frame)
```

不过不妨先思考一下，为什么不能直接获取值呢？比如这样：

```swift
CGFloat width = frame.size.width
CGFloat height = frame.size.height
```

> For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

> — [Apple, CGGeometry Reference Documentation](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CGGeometry/)

（译者注：）考虑两个CGRect，第一个`origin`是[0,0]，`size`是[10, 10]。第二个的`origin`是[10,10]，`size`是[-10, -10]。两者其实是等价的，无论是OC还是Swift都支持这两种写法。问题在于，在OC中，获取结构体的属性就直接获取到它的真实值了，这个值有可能为负。所以OC建议我们调用系统API而不是直接获取值，而在Swift中，`width`、`height`等被设计为计算属性，自然就不存在这样的问题了。

由于系统提供了完备的API，很多人觉得不直接获取值也不是什么问题。不过Swift提供了简单的点语法表达式，将我们从这种不美观的API中解放出来：

```swift
let frame = CGRect(x: 0, y: 0, width: 100, height: 100)
let width = frame.width
let height = frame.height
let maxX = frame.maxX
let maxY = frame.maxY
```

## 可变性

```swift
let frame = CGRect(x: 0, y: 0, width: 100, height: 100)
let view = UIView(frame: frame)
view.frame.origin.x += 10
```

现在，你不仅可以直接修改结构体中的某个值，还可以直接替换整个子结构体：

```swift
let view = UIView(frame: .zero)
view.frame.size = CGSize(width: 10, height: 10)
view.frame.origin = CGPoint(x: 10, y: 10)
```

单单这个特性就足够我们放弃OC，投入Swift的怀抱了。我们不必非得重新创建一个结构体实例再修改了，不到两年前，我们OC开发者还被迫写下这样的代码：

```objc
CGRect frame = CGRectMake(0, 0, 100, 100);
UIView *view = [[UIView alloc] initWithFrame: frame];
CGRect newFrame = view.frame;
newFrame.size.width = view.frame.origin.x + 10;
view.frame = newFrame;
```

我不知道你怎么想，但是写出这样的代码快要把我逼疯了。根据view的`frame`重新创建结构体，修改它，然后再把view的`frame`改成这个结构体，再见了，再也不见！

## 最后说一句

以上这些改变还适用于UIKit中的其他结构体：

```swift
// UIEdgeInsets 
var edgeInsets = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
edgeInsets.top += 10

// UIOffset
var offset = UIOffset(horizontal: 10, vertical: 10)
offset.vertical += 10
```

示范代码可以在[作者的github](https://github.com/andyyhope/Blog_CGGeometry)上找到