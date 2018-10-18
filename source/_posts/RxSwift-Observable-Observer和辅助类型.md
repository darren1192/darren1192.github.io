---
title: RxSwift Observable&Observer和辅助类型
date: 2018-09-22 23:49:42
tags: RxSwift
---
在我们日常开发中，有一些既可是`Observable`又可是`Observer`。举个🌰：
```
let observable = textField.rx.text
observable.subscribe(onNext: { text in show(text: text) })
```
在这行代码中，`textField`当前文本就是一个`Observable`，当用户在`textField`中输入时，就会`show`文本内容。
再举个🌰：
```
let disposeBag = DisposeBag()
let observer = textField.rx.text
let observable = Observable<String>.just("A")
observable.bind(to: observer)
        .disposed(by: disposeBag)
```
这个时候，屏幕上就会显示：
![textField.png](https://upload-images.jianshu.io/upload_images/2855070-a113479a0f2d92f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时，`textField`的当前文本就是`Observer`。
此外，框架中还有一些辅助类型，既可是`Observable`又可是`Observer`。
#### 1. AsyncSubject
`AsyncSubject `将在`Observable`产生完成事件后，发出最后一个元素（仅仅只有最后一个元素）。如果`Observable`没有发出任何元素，只有一个完成事件，那`AsyncSubject`也只有一个完成事件。如果`Observable`因`error`中止，那么`AsyncSubject`只会将`error`发送出来，不会发出其它元素。
举个🌰：
```
let disposeBag = DisposeBag()
//创建一个AsyncSubject
let subject = AsyncSubject<String>()
//订阅subject
subject.subscribe{ print($0) }
    .disposed(by: disposeBag)
subject.onNext("B")
subject.onNext("C")
subject.onNext("D")
subject.onCompleted()
```
这个时候输出：
```
next(D)
completed
```

此时我们再改造一下：
```
let subject = AsyncSubject<String>()
subject.subscribe{ print($0) }
    .disposed(by: disposeBag)
subject.onNext("B")
subject.onNext("C")
subject.onNext("D")
//SubjectError 自己定义的enum Error
subject.onError(SubjectError.error)
```
就会输出：
```
error(error)
```
#### 2. PublishSubject
`PublishSubject`是将对观察者发送`订阅后产生的元素`，而在订阅前发出的元素将不会发送给观察者。
- `PublishSubject`是最普通的`Subject`，它不需要初始值就能创建
- `PublishSubject`的订阅者从他们开始订阅的时间点起，可以收到订阅后`Subject`发出的新`Event`，而不会收到他们在订阅前已发出的`Event`

举个🌰：
```
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()
//由于当前没有订阅，所以不输出
subject.onNext("🐶")
//第一次订阅
subject.subscribe(onNext: { (element) in
    print("第一次订阅:\(element)")
}, onCompleted: {
    print("completed")
}).disposed(by: disposeBag)
//当前有一个订阅，输出
subject.onNext("🐱")

//第二次订阅
subject.subscribe(onNext: { (element) in
    print("第二次订阅:\(element)")
}, onCompleted: {
    print("completed")
}).disposed(by: disposeBag)
//当前有两个订阅 输出
subject.onNext("🐹")

//结束subject
subject.onCompleted()
//再次发出.next事件
subject.onNext("🐯")
```
显示如下：
```
第一次订阅:🐱
第一次订阅:🐹
第二次订阅:🐹
第一次订阅:completed
第二次订阅:completed
第三次订阅:completed
```

####  3. ReplaySubject
`ReplaySubject`将对观察者发送全部的元素，无论观察者是何时进行订阅的。这里存在多个版本的`ReplaySubject`，有的只会将最新的n个元素发送给观察者，有的只会将限制时间段内最新的元素发送给观察者。如果把`ReplaySubject`当作观察者来使用，注意不要在多个线程调用`onNext`,`onError`或`onCompleted`。这样会导致无序调用，将造成意想不到的结果。
- `ReplaySubject`在创建时候需要设置一个`bufferSize`，表示它对于它发送过的`event`的缓存个数
- 比如一个`ReplaySubject`的`bufferSize`设置为 2，它发出了 3 个`.next`的`event`，那么它会将后两个（最近的两个）`event`给缓存起来。此时如果有一个`subscriber`订阅了这个 `ReplaySubject`，那么这个`subscriber`就会立即收到前面缓存的两个`.next`的`event`
- 如果一个`subscriber`订阅已经结束的`ReplaySubject`，除了会收到缓存的`.next`的`event`外，还会收到那个终结的`.error`或者`.complete`的`event`

举个🌰：
```
let disposeBag = DisposeBag()
//创建
let subject = ReplaySubject<String>.create(bufferSize: 0)
//第1次订阅subject
subject.subscribe{ print("第一次订阅:\($0)") }
    .disposed(by: disposeBag)
//发送.next事件
subject.onNext("A")
subject.onNext("B")
//第二次订阅
subject.subscribe{ print("第二次订阅:\($0)" )}
    .disposed(by: disposeBag)
//发送.next事件
subject.onNext("C")
subject.onNext("D")
```
输出结果：
```
第一次订阅:next(A)
第一次订阅:next(B)
第一次订阅:next(C)
第二次订阅:next(C)
第一次订阅:next(D)
第二次订阅:next(D)
```
当`bufferSize`改成1时，结果就变成了：
```
第一次订阅:next(A)
第一次订阅:next(B)
第二次订阅:next(B)
第一次订阅:next(C)
第二次订阅:next(C)
第一次订阅:next(D)
第二次订阅:next(D)

```
如果是2的话，结果就变成了：
```
第一次订阅:next(A)
第一次订阅:next(B)
第二次订阅:next(A)
第二次订阅:next(B)
第一次订阅:next(C)
第二次订阅:next(C)
第一次订阅:next(D)
第二次订阅:next(D)
```

#### 4. BehaviorSubject
`BehaviorSubject `会把`Observable `最新元素发出来（如果不存在最新的元素，就发出默认元素）。然后将随后产生的元素发送出来。如果`Observable `因为`error`事件而中止，则不会发出任何元素，将`error`事件发出来。
- `BehaviorSubject`需要通过一个默认初始值来创建
- 当一个订阅者来订阅它的时候，这个订阅者会立即收到 `BehaviorSubjects`上一个发出的`event`。之后就跟正常的情况一样，它也会接收到`BehaviorSubject`之后发出的新的`event`

举个🌰：
```
let disposeBag = DisposeBag()
//创建一个BehaviorSubject
let subject = BehaviorSubject.init(value: "🐭")
//第一次订阅
subject.subscribe{ print("第一次订阅:\($0)") }
    .disposed(by: disposeBag)
//发送.next事件
subject.onNext("🐯")
//发送error事件
subject.onError(NSError(domain: "local", code: 0, userInfo: nil))
//第二次订阅
subject.subscribe{ print("第二次订阅:\($0)") }
    .disposed(by: disposeBag)
```
输出结果：
```
第一次订阅:next(🐭)
第一次订阅:next(🐯)
第一次订阅:error(Error Domain=local Code=0 "(null)")
第二次订阅:error(Error Domain=local Code=0 "(null)")
```
#### 4. Variable
在`RxSwift`中，`Variable `相当于`Swift`中的`var`。
- `Variable`其实就是对`BehaviorSubject`的封装，所以它也必须要通过一个默认的初始值进行创建。
- `Variable`具有`BehaviorSubject`的功能，能够向它的订阅者发出上一个`event`以及之后新创建的`event`。
- 不同的是，`Variable`还会把当前发出的值保存为自己的状态。同时它会在销毁时自动发送`.complete`的`event`，不需要也不能手动给`Variables`发送`completed`或者`error`事件来结束它。
- 简单地说就是`Variable`有一个`value`属性，我们改变这个`value`属性的值就相当于调用一般`Subjects`的`onNext()`方法，而这个最新的`onNext()`的值就被保存在`value`属性里了，直到我们再次修改它。
- `Variables`本身没有`subscribe()`方法，但是所有`Subjects`都有一个`asObservable()`方法。我们可以使用这个方法返回这个`Variable`的`Observable`类型，拿到这个`Observable`类型我们就能订阅它了。

举个🌰：
```
let disposeBag = DisposeBag()
let variable = Variable.init("A")
variable.value = "B"
//第一次订阅
variable.asObservable().subscribe{ print("第一次订阅:\($0)") }
    .disposed(by: disposeBag)
//修改value
variable.value = "C"
//第二次订阅
variable.asObservable().subscribe{ print("第二次订阅:\($0)") }
    .disposed(by: disposeBag)
//修改value
variable.value = "D"
```
结果如下：
```
第一次订阅:next(B)
第一次订阅:next(C)
第二次订阅:next(C)
第一次订阅:next(D)
第二次订阅:next(D)
第一次订阅:completed
第二次订阅:completed
```
