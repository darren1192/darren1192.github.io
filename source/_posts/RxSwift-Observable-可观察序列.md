---
title: RxSwift Observable-可观察序列
date: 2018-09-20 22:25:46
tags: RxSwift
---
[上一篇](https://darren1192.github.io/2018/09/19/RxSwift-%E6%A0%B8%E5%BF%83/#more)已经介绍了`Observable`是什么，现在简单介绍一下它怎么创建，以及`RxSwift`里面`Observable`存在的一些特征序列。
### 常见的创建方法
####  just() 方法
该方法通过传入一个默认值完成初始化，并指定了当前`Observable`所发出事件携带的数据类型
```
let observable = Observable<Int>.just(1)
```
#### of()方法
该方法可以接受多个参数来创建实例，但这些参数必须是同类型
```
let observable = Observable.of("A", "B", "C")
```
#### from()方法
该方法只接收数组作为参数，并抽取出数组里的元素来作为数据流中的元素
```
let observable = Observable.from(["A", "B", "C"])
```
#### never()方法
该方法创建一个永远不会发出`Event`（也不会终止）的 `Observable`序列
```
let observable = Observable<Int>.never()
```
#### empty()方法
该方法创建一个空内容的`Observable`序列
```
let observable = Observable<Int>.empty()
```
#### error() 方法
该方法创建一个不做任何操作，而是直接发送一个错误的`Observable`序列
```
enum MyError: Error {
    case A
    case B
}

let observable = Observable<Int>.error(MyError.A)
```
#### range() 方法
该方法通过指定起始和结束数值，创建一个以这个范围内所有值作为初始值的`Observable`序列
```
let observable = Observable.range(start: 1, count: 5)
```
#### repeatElement() 方法
该方法创建一个可以无限发出给定元素的`Event`的`Observable`序列
```
let observable = Observable.repeatElement(1)
```
#### generate() 方法
该方法创建一个只有当提供的所有的判断条件都为 true 的时候，才会给出动作的`Observable`序列
```
let observable = Observable.generate(
    initialState: 0,
    condition: { $0 <= 10 },
    iterate: { $0 + 2 }
)
```
#### interval() 方法
这个方法创建的`Observable`序列每隔一段设定的时间，会发出一个索引数的元素。而且它会一直发送下去
```
let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
observable.subscribe { event in
    print(event)
//next(0) next(1) next(2)......
}
```
#### timer() 方法
- 创建的`Observable`序列在经过设定的一段时间后，产生唯一的一个元素
```
//10秒种后发出唯一的一个元素0
let observable = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
observable.subscribe { event in
    print(event)
}
```
- 创建的`Observable`序列在经过设定的一段时间后，每隔一段时间产生一个元素
```
//延时10秒种后，每隔1秒钟发出一个元素
let observable = Observable<Int>.timer(10, period: 1, scheduler: MainScheduler.instance)
    observable.subscribe { event in
    print(event)
}
```
#### create()方法
该方法接受一个`block`形式的参数，任务是对每一个过来的订阅进行处理
```
let observable = Observable<String>.create { (observer) -> Disposable in
    observer.onNext("test")
    observer.onCompleted()
    return Disposables.create()
}
observable.subscribe { (element) in
    print(element)
}
//next(test)
//completed
```
#### deferred() 方法
该个方法相当于是创建一个`Observable`工厂，通过传入一个`block`来执行延迟`Observable`序列创建的行为，而这个`block`里就是真正的实例化序列对象的地方
```
var time = 0
let factory: Observable<Int> = Observable.deferred {
    time += 1
    return Observable.just(time)
}
factory.subscribe { (event) in
    print(event)
}
factory.subscribe { (event) in
    print(event)
}
factory.subscribe { (event) in
    print(event)
}
factory.subscribe { (event) in
    print(event)
}
/* 
打印结果
next(1)
completed
next(2)
completed
next(3)
completed
next(4)
completed
*/
```
### 特征序列
` Swift`是一个强类型语言，它相对弱类型语言更加严谨。我们可以通过类型来判断出，实例有哪些特征。同样的在`RxSwift` 里面`Observable`也存在一些特征序列，这些特征序列可以帮助我们更准确的描述序列。并且它们还可以给我们提供语法糖，让我们能够用更加优雅的方式书写代码，他们分别是`Single`、`Completable`、`Maybe`、`Driver`、`ControlEvent`、`ControlProperty`
####  Single
`Single`,在`RxSwift`中,对它的解释是*Represents a push style sequence containing 1 element*，它要么只能发出一个元素，要么产生一个`error`事件
- 发出一个元素或一个`error`事件
- 不会共享状态变化
附：
```
public enum SingleEvent<Element> {
case success(Element)
case error(Swift.Error)
}
```
比如说，我们利用`Single`实现一个网络请求，返回成功的结果或失败:
```
func getReop(_ repo: String) -> Single<[String: Any]>{
    return Single<[String: Any]>.create(subscribe: { (single) -> Disposable in
        let url = URL.init(string: "https://api.github.com/repos/\(repo)")!
        let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) in
            if let error = error{
                single(.error(error))
                return
            }
            guard let data = data,
                let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                let result = json as? [String: Any] else{
                    single(.error(DataError.cantParseJSON))
                    return
                }
                single(.success(result))
        })
        task.resume()
        return Disposables.create {
            task.cancel()
        }
    })
}
//与数据相关的错误类型
enum DataError: Error {
    case cantParseJSON
}
```
当我们想调用这个方法的时候:
```
let disposeBag = DisposeBag()
getReop("ReactiveX/RxSwift").subscribe(onSuccess: { (json) in
    print("Json:\(json)")
}) { (error) in
    print("error:\(error)")
}.disposed(by: disposeBag)
```

#### Completable
`Completable`，在`RxSwift`中，对它的解释*Represents a push style sequence containing 0 elements.*可以理解为表示包含0个元素的推送样式序列，它要么产生`completed`事件，要么产生`error`事件。
- 不会发出任何元素
- 只会发出一个`completed`事件或者一个`error`事件
- 不会共享状态变化
附：
```
public enum CompletableEvent {
    case error(Swift.Error)  
    case completed
}
```
举个🌰:
```
func cacheLocally() -> Completable {
    return Completable.create(subscribe: { (completable) -> Disposable in
    //将数据缓存到本地（这里掠过具体的业务代码，随机成功或失败）
    let success = (arc4random() % 2 == 0)
    guard success else {
        completable(.error(CacheError.failedCaching))
        return Disposables.create{}
    }
    completable(.completed)
    return Disposables.create()
    })
}
enum CacheError: Error {
    case failedCaching
}
```
调用方法：
```
cacheLocally().subscribe(onCompleted: {
    print("completed")
}) { (error) in
    print("error:\(error)")
}.disposed(by: disposeBag)
```

#### Maybe
`MayBe`介于`Single`和`Completable`之间，它要么只能发出一个元素，要么产生一个`completed`事件，要么产生一个`error`事件。
- 发出一个元素、或者一个`completed`事件、或者一个`error` 事件
- 不会共享状态变化
附：
```
public enum MaybeEvent<Element> {
case success(Element)
case error(Swift.Error)
case completed
}
```
举个🌰：
```
func generateString() -> Maybe<String>{
    return Maybe<String>.create(subscribe: { (maybe) -> Disposable in
    maybe(.success("success"))
    maybe(.completed)
    maybe(.error(StringError.failedGenerate))
    return Disposables.create()
    })
}
enum StringError: Error {
    case failedGenerate
}
```
调用方法：
```
generateString().subscribe(onSuccess: { (element) in
    print("success:\(element)")
}, onError: { (error) in
    print("error:\(error)")
}) {
    print("completed")  
}.disposed(by: disposeBag)
```
#### Driver
`Driver`准确来说是`RxCocoa`的特征序列，目标是提供一种简便的方式在 UI 层编写响应式代码。
##### 为什么使用Driver
[这部分我们引用RxSwift中文文档内容](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/observable/driver.html)
我们举个例子来说明一下，为什么要使用`Driver`
这是文档简介页的例子:
```
let results = query.rx.text
                    .throttle(0.3, scheduler: MainScheduler.instance)   
                    .flatMapLatest { query in
                        fetchAutoCompleteItems(query)
                    }

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) {
    (_, result, cell) in
    cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```
这段代码的主要目的是：
- 取出用户输入稳定后的内容
- 向服务器请求一组结果
- 将返回的结果绑定到两个 UI 元素上：`tableView`和 显示结果数量的`label`
代码存在的问题：
- 如果`fetchAutoCompleteItems`的序列产生了一个错误（网络请求失败），这个错误将取消所有绑定，当用户输入一个新的关键字时，是无法发起新的网络请求
- 如果`fetchAutoCompleteItems`在后台返回序列，那么刷新页面也会在后台进行，这样就会出现异常崩溃
- 返回的结果被绑定到两个 UI 元素上。那就意味着，每次用户输入一个新的关键字时，就会分别为两个 UI 元素发起 HTTP 请求，这并不是我们想要的结果
一个更好的方案是这样的：
```
let results = query.rx.text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
    .observeOn(MainScheduler.instance)  // 结果在主线程返回
    .catchErrorJustReturn([])           // 错误被处理了，这样至少不会终止整个序列
    }
    .share(replay: 1)                           // HTTP 请求是被共享的

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) {
    (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```
在一个大型系统内，要确保每一步不被遗漏是一件不太容易的事情。所以更好的选择是合理运用编译器和特征序列来确保这些必备条件都已经满足。
以下是使用`Driver`优化后的代码：
```
let results = query.rx.text.asDriver()        // 将普通序列转换为 Driver
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .asDriver(onErrorJustReturn: [])  // 仅仅提供发生错误时的备选返回值
    }

results
    .map { "\($0.count)" }
    .drive(resultCount.rx.text)               // 这里改用 `drive` 而不是 `bindTo`
    .disposed(by: disposeBag)                 // 这样可以确保必备条件都已经满足了

results
    .drive(resultsTableView.rx.items(cellIdentifier: "Cell")) {
    (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```
首先第一个`asDriver`方法将`ControlProperty`转换为`Driver`
然后第二个变化是:
```
.asDriver(onErrorJustReturn: [])
```
任何可被监听的序列都可以被转换为 Driver，只要他满足 3 个条件:
- 不会产生 error 事件
- 一定在 MainScheduler 监听（主线程监听）
- 共享状态变化
那么要如何确定条件都被满足？通过 Rx 操作符来进行转换。asDriver(onErrorJustReturn: []) 相当于以下代码
```
let safeSequence = xs
    .observeOn(MainScheduler.instance)       // 主线程监听
    .catchErrorJustReturn(onErrorJustReturn) // 无法产生错误
    .share(replay: 1, scope: .whileConnected)// 共享状态变化
return Driver(raw: safeSequence)           // 封装
```
最后使用`drive`而不是`bindTo`。
#### ControlEvent
`ControlProperty`是专门用来描述 UI 控件属性，拥有该类型的属性都是被观察者（`Observable`），它也是`RxCocoa`的特征序列
`ControlProperty`具有以下特征：
- 不会产生`error`事件
- 一定在`MainScheduler`订阅（主线程订阅）
- 一定在`MainScheduler`监听（主线程监听）
- 共享状态变化
举个🌰：
```
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {

    @IBOutlet weak var textField: UITextField!
    @IBOutlet weak var label: UILabel!
    //负责对象销毁
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        //将textField输入的文字绑定到label上
        textField.rx.text
            .bind(to: label.rx.text)
            .disposed(by: disposeBag)
        }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
有人可能会纳闷，这跟`ControlProperty`有什么关系，没看到它的影子啊。我们查看`textField.rx.text`的`text`方法：
```
extension Reactive where Base : UITextField {
public var text: ControlProperty<String?> {
        return value
    }

public var value: ControlProperty<String?> {
    return base.rx.controlPropertyWithDefaultEvents(
        getter: { textField in
        textField.text
    },
        setter: { textField, value in
            if textField.text != value {
            textField.text = value
            }
        }
        )
    }
}
```
原来UITextField 的`rx.text`属性类型便是 `ControlProperty<String?>`
同时，这段代码也给我们启示，为控件添加属性，可以采用`extension Reactive where Base : UITextField`的方法。
#### ControlEvent
`ControlEvent`专门用于描述 UI 控件所产生的事件，它具有跟`ControlProperty`一样的特征：
- 不会产生`error`事件
- 一定在`MainScheduler`订阅（主线程订阅）
- 一定在`MainScheduler`监听（主线程监听）
- 共享状态变化
举个🌰：
```
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {

    let disposeBag = DisposeBag()

    @IBOutlet weak var button: UIButton!

    override func viewDidLoad() {

        //订阅按钮点击事件
        button.rx.tap
            .subscribe(onNext: {
                print("blick")
            }).disposed(by: disposeBag)
        }
}
```
可能也有人会疑问，`ControlEvent`在哪？查看tap方法，会看到源码（`UIButton+Rx.swift`），这个时候就会发现 UIButton 的`rx.tap`方法类型便是`ControlEvent<Void>`：
```
import RxSwift
import UIKit

extension Reactive where Base: UIButton {
public var tap: ControlEvent<Void> {
    return controlEvent(.touchUpInside)
    }
}
```
至此，我们就对`Observable`有了一个简单的介绍。
