---
title: RxSwift 核心
date: 2018-09-19 20:49:45
tags: RxSwift
---
在这篇中，简单介绍一下RxSwift的核心内容，首先献祭上一张图：
![RxSwiftCore.png](https://upload-images.jianshu.io/upload_images/2855070-90edfb06fc5c1b73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Observable:产生事件
- Observer: 响应事件
- Operator:创建变化组合事件
- Disposable:管理绑定（订阅）的生命周期
- Schedulers:线程队列调配
## Observable 可被监听的序列
Observable实质上就是一个Sequence。序列分为有穷序列和无穷序列，主要就是用来形成一条数据流。比如说：我们通过nickname和password向服务器请求用户信息，它就是一个有穷数列，当数据传输完毕，这个序列就闭合了。当我们去监测某个按钮的点击，我们不知道会点击多少次，这就是无穷序列。而我们所有的操作产生的数据都会通过Observable传输。
Observable<T> 这个类就是Rx 框架的基础，我们称它为可观察序列。它的作用就是可以异步地产生一系列的 Event（事件），即一个 Observable<T> 对象会随着时间推移不定期地发出 event(element : T) 。
举个🌰：
```
let observable = Observable.of("A", "B", "C")
observable.subscribe(onNext: { element in
print(element)
}, onError: { error in
print(error)
}, onCompleted: {
print("completed")
}, onDisposed: {
print("disposed")
})
```
这里subscribe后面的onNext,onError, onCompleted 分别响应我们创建 json 时，构建函数里面的onNext,onError, onCompleted 事件。我们称这些事件为 Event。
Event的定义如下：
```
public enum Event<Element> {
/// Next element is produced.
case next(Element)
/// Sequence terminated with an error.
case error(Swift.Error)
/// Sequence completed successfully.
case completed
}
```
可以看出Event就是一个枚举，也就是说一个 Observable 是可以发出 3 种不同类型的 Event 事件。
- next:序列产生了一个新的元素
- error:创建序列时产生了一个错误，导致序列终止
- completed:序列的所有元素都已经成功产生，整个序列已经完成
我们通过这些Event实现业务逻辑。
##  Observer 观察者
观察者（Observer）的作用就是监听事件，然后对这个事件做出响应，或者说任何响应事件的行为都是观察者。比如：
- 当前视频播放到30分钟时，弹出视频小剧场(广告)，后者就是观察者。
- 当前温度高于30度，打开空调降温，后者就是观察者。
- 当你点击视频拍摄时，系统弹出弹框问你是否同意使用摄像头，后者就是观察者。
- ······
举个🌰：
```
tap.subscribe(onNext: { [weak self] in
self?.showAlert()
}, onError: { error in
print("发生错误： \(error.localizedDescription)")
}, onCompleted: {
print("任务完成")
})
```
在这里，弹出提示框就是观察者。
创建观察者最直接的方法就是在 Observable 的 subscribe 方法后面描述，事件发生时，需要如何做出响应。而观察者就是由后面的 onNext，onError，onCompleted的这些闭包构建出来的。此外，我们还有AnyObserver、Binder方式创建观察者。
## Operator  操作符
操作符可以帮助大家创建新的序列，或者变化组合原有的序列，从而生成一个新的序列。可以对比一下Swift中的高阶函数。具体使用方法暂不做介绍，只举个🌰：
```
let obserable = Observable<Int>.of(1,2,3,4,5,6,7,8,9,10)
obserable.filter{ $0 > 5}
.subscribe(onNext: { print($0) })
//6 7 8 9 10
```

## Disposable  可被清除的资源
一个 Observable 序列被创建出来后它不会马上就开始被激活从而发出 Event，而是要等到它被某个人订阅了才会激活它。激活之后要一直等到它发出了.error或者 .completed的 event 后，它才被终结。而如果我们想提前释放这些资源或者取消订阅的话，可以对返回的可被清除的资源（Disposable） 调用 dispose 方法：
```
var disposable: Disposable?

override func viewWillAppear(_ animated: Bool) {
super.viewWillAppear(animated)

self.disposable = textField.rx.text.orEmpty
.subscribe(onNext: { text in print(text) })
}

override func viewWillDisappear(_ animated: Bool) {
super.viewWillDisappear(animated)

self.disposable?.dispose()
}
```
当然了，这个方法并不建议。因为Swift是使用ARC管理内存，所有推荐使用清除包（DisposeBag） 来实现这种订阅管理机制。我们可以把一个 DisposeBag对象看成一个垃圾袋，把用过的订阅行为都放进去，这个DisposeBag 就会在自己快要dealloc 的时候，对它里面的所有订阅行为都调用 dispose()方法
```
let disposeBag = DisposeBag()

//第1个Observable，及其订阅
let observable1 = Observable.of("A", "B", "C")
observable1.subscribe { event in
print(event)
}.disposed(by: disposeBag)

//第2个Observable，及其订阅
let observable2 = Observable.of(1, 2, 3)
observable2.subscribe { event in
print(event)
}.disposed(by: disposeBag)

```
此外，还有一种takeUntil，这里就不做过多描述了。
## Schedulers 调度器
Schedulers 是 Rx 实现多线程的核心模块，它主要用于控制任务在哪个线程或队列运行。
在Swift中，我们写一个GCD这么写：
```
// 后台取得数据，主线程处理结果
DispatchQueue.global(qos: .userInitiated).async {
let data = try? Data(contentsOf: url)
DispatchQueue.main.async {
self.data = data
}
}
```
如果用RxSwift去实现，就这么写:
```
let rxData: Observable<Data> = ...

rxData
.subscribeOn(ConcurrentDispatchQueueScheduler(qos: .userInitiated))
.observeOn(MainScheduler.instance)
.subscribe(onNext: { [weak self] data in
self?.data = data
})
.disposed(by: disposeBag)
```
在这里，我们用subscribeOn来决定数据序列的构建函数在哪个 Scheduler 上运行，用observeOn来决定在哪个 Scheduler 监听这个数据序列。比如说：在后台发起网络请求，然后解析数据，最后在主线程刷新页面。你就可以先用 subscribeOn切到后台去发送请求并解析数据，最后用 observeOn切换到主线程更新页面。此外，我们来应该了解：
- MainScheduler：代表主线程。如果你需要执行一些和 UI 相关的任务，就需要切换到该 Scheduler 运行。
- SerialDispatchQueueScheduler：抽象了串行 DispatchQueue。如果你需要执行一些串行任务，可以切换到这个 Scheduler 运行。
- ConcurrentDispatchQueueScheduler：抽象了并行 DispatchQueue。如果你需要执行一些并发任务，可以切换到这个 Scheduler 运行。
- OperationQueueScheduler：抽象了 NSOperationQueue。它具备 NSOperationQueue 的一些特点，例如，你可以通过设置maxConcurrentOperationCount，来控制同时执行并发任务的最大数量。

