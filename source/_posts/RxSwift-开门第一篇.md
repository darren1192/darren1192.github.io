---
title: RxSwift-开门第一篇
date: 2018-09-18 23:07:11
tags: RxSwift
---
## 开篇扯淡：
&emsp;&emsp;为什么入RxSwift这个坑？因为我作为一个iOSer，主要语言是swift，日常开发已经很少使用OC。随着产品代码量日益增加，准备从MVC转成MVVM，而MVVM最搭配的就是RxSwift。所以，作为我个人来说，入RxSwift这个坑，还是比较合情合理的。
## 正经内容：
### ReactiveX是什么
[ReactiveX](http://reactivex.io/)（简写: Rx）是一个可以帮助我们简化异步编程的框架.
它拓展了[观察者模式](https://zh.wikipedia.org/wiki/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F)。使你能够自由组合多个异步事件，而不需要去关心线程，同步，线程安全，并发数据以及I/O阻塞。
### 什么是RxSwift
[RxSwift](https://github.com/ReactiveX/RxSwift) 是 [Rx](https://github.com/Reactive-Extensions/Rx.NET) 的 **Swift** 版本，它尝试将原有的一些概念移植到 iOS/macOS 平台。
### 为什么要使用RxSwift
>复合 - Rx 就是复合的代名词
复用 - 因为它易复合
清晰 - 因为声明都是不可变更的
易用 - 因为它抽象的了异步编程，使我们统一了代码风格
稳定 - 因为 Rx 是完全通过单元测试的  

当然了，上面都是假大空，具体来说
在编写代码时我们经常会需要检测某些值的变化，然后进行相应的处理。比如说按钮的点击事件、textFiled值的变化、tableView中cell的点击事件等等。在日常搬砖中，我们针对不同的情况，需要采用不同的事件传递方法去处理，比如**delegate**、**notifinotion**、**target-action**、**KVO**等等。
而在RxSwift中，让程序里的事件传递响应方法做到统一，全部采用Rx的“信号链”方式。举个🌰：
__Target Action__
```
button.addTarget(self, action: #selector(buttonEvent), for: .touchUpInside)
```
```
@objc func buttonEvent(){
    print("button Event")
}
```
用Rx实现
```
button.rx.tap.subscribe(onNext: { () in
    print("button event")
}, onError: nil, onCompleted: nil, onDispo
sed: nil)
```
这样一来，代码逻辑清晰可见。
当然了，为了MVVM，还是很有学习RxSwift的必要的。
#### RxSwift地址与安装
传送门：[GitHub](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FReactiveX%2FRxSwift)
这里面啥都有，无需多言。
#### RxSwift 与 RxCocoa
RxSwift在日常使用中，需要import
```
import RxCocoa
import RxSwift
```
这个两个库的作用分别是：
- RxSwift：它只是基于 Swift 语言的 Rx 标准实现接口库，所以 RxSwift 里不包含任何 Cocoa 或者 UI方面的类。
- RxCocoa：是基于 RxSwift针对于 iOS开发的一个库，它通过 Extension 的方法给原生的比如 UI 控件添加了 Rx 的特性，使得我们更容易订阅和响应这些控件的事件。

既然已经有了这个RxSwift开篇，希望自己坚持下去。虽然不太喜欢写文章，但这也不失为监督自己的一个方法。__笨鸟先飞，知行合一。__
最后，感谢那些“巨人”，对于RxSwift的学习，完全是站在巨人的肩膀上，感谢他们的文章，指引了我的方向。尤其感谢[RxSwift中文文档](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/)和[航哥swift](http://www.hangge.com/)，他们的文章，对我的帮助很大，感谢。
