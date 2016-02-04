又是周五了，周末不要浪，一起学点Swift！

本周再次为大家带来了一些Swift的小技巧，都是些奇淫巧计，不知道也无妨，但Swift最吸引我的一点就是它的简洁易用。主要内容有：

1. `private(set)`语法
2. 分号的使用
3. 利用`String`类型初始化方法简化`UITableViewCell`的`reuseIdentifier`
4. 简洁的声明多个变量
5. 压轴推荐：Xcode断点调试小技巧

我为这篇博客制作了一个demo，您可以去在我的github上clone下来：[SwiftTips](https://github.com/bestswifter/MySampleCode/tree/master/SwiftTips)，如果觉得有帮助还望给个star以示支持。

## private(set)

出于代码安全性的考虑，如果一个类的属性不会被其他类使用，那么可以把它声明为`private`。更进一步我们可以使用`private(set)`关键字告诉编译器，这个类对外可读但是不可写，比如：

```swift
// In other swift file
struct Person {
    private(set) var name = "Unknown"
}

// In main.swift
// 可以获取name属性的值
print(person.name)

// 报错，不能在PrivateSet.swift文件外对name属性赋值
//person.name = "newName"
```

这个属性只能在文件内部被读写，即使是在结构体的定义外也可以。但是在别的文件中就不能对其赋值了。

需要强调的一点是，只有`private(set)`关键字，并没有`private(get)`关键字。

## 利用好分号

分号在Swift中几乎退出了历史舞台，但在某些情况下使用分号也是不错的选择。

假设在函数的开头有一个`guard`判断，如果判断不成立则退出函数，并输出一些调试信息，过去的版本可以这样写：

```swift
func doSomething() {
    let error: AnyObject? = nil
    guard error == nil
        else {
            print("Error information")
            return
        }
}
```

如果使用分号，可以简化代码，它把代码压缩在一行语句中，简洁又不失可读性：

```swift
func doSomething() {
    let error: AnyObject? = nil
    guard error == nil else { print("Error information"); return }
}
```

## reuseIdentifier

给cell一个`reuseIdentifier `是一件挺麻烦的事情，首先不能瞎起名字，比如`let reuseIdentifier = "reuse"`。一旦同一个`UITableView`中有两种或更多cell，事情就比较麻烦了。

这就要求我们为`reuseIdentifier `赋值是要考虑到字符串的具体含义，比如代码可能是这样的：

```swift
let reuseIdentifier = "TableViewCommentCellIndentifer"
```

作为一个喜欢偷懒的人，为了节省起名字以及输入这些字符串的时间，我们完全可以这样写：

```swift
let reuseIdentifier = String(TableViewCell)
```

这里的`TableViewCell `是自定义的`UITableViewCell `的子类，把它传入字符串的构造函数中得到的结果是"TableViewCell"，一切显得那么和谐简介。

关于字符串初始化函数的规则，可以参考我的这篇文章：[你其实真的不懂print("Hello,world")](http://www.jianshu.com/p/abb55919c453)

## 简洁声明多个变量

对于一些相互有关联的变量，相比于在每行中声明一个，还有一种更简洁美观的方式：

```swift
var (top, left, width, height) = (0.0, 0.0, 100.0, 50.0)
//rect.width = width
```

## Xcode断点调试小技巧

好吧，我承认上一个tip的实用性不是很强，有点凑数字之嫌，下面重点介绍一些调试方面的技巧作为补偿。

可能很多人都知道使用断点能够大幅度提高开发效率，其实Xcode断点还有更多的功能值的发掘。比如我们可以在触发断点时不终止程序运行（如果不需要单步调试的话）：

![断点处继续运行](http://images.bestswifter.com/20160201/breakpoint.png)

勾选最后一个选项后，程序就不会在断点处终止了。

其他的几个选项也很有用处，第一个表示在什么情况下才会触发断点，第二个选项表示前几次不触发断点。

在**Action**选项中，我们可以选择触发断点时的执行事件，比如这里我加入了播放声音，以及向控制台打印字符串"This is a message to console"，同时还会调用调试命令`p i`，表示输出i的值：

```swift
func customDebug() {
    for i in 1..<10 {
        // 这里加断点
    }
}
```

代码运行后的结果是：

```swift
(Int) $R8 = 9
This is a message to console
```

如果您运行了demo，还会听到清脆的“叮”的一声。