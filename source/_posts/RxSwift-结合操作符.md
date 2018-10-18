---
title: RxSwift 结合操作符
date: 2018-09-28 21:47:21
tags: RxSwift
---
结合操作（或者称合并操作）指的是将多个`Observable`序列进行组合，拼装成一个新的`Observable`序列。
#### 1. startWith
该方法会在`Observable`序列开始之前插入一些事件元素。即发出事件消息之前，会先发出这些预先插入的事件消息。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-1e67a4bb28af2c6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

Observable.of(2, 3)
        .startWith(1)
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
1
2
3
```
#### 2. concat
- `concat`操作符将多个`Observables`按顺序串联起来，当前一个`Observable`元素发送完毕后，后一个 Observable 才可以开始发出元素。
- `concat`将等待前一个 Observable 产生完成事件后，才对后一个`Observable`进行订阅。如果后一个是“热”`Observable`，在它前一个`Observable`产生完成事件前，所产生的元素将不会被发送出来。

![image.png](https://upload-images.jianshu.io/upload_images/2855070-8a8303ad65144b1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
et disposeBag = DisposeBag()

let subject1 = BehaviorSubject.init(value: "1")
let subject2 = BehaviorSubject.init(value: "A")

let variable = Variable.init(subject1)
variable.asObservable()
        .concat()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)

subject1.onNext("2")
subject1.onNext("3")

variable.value = subject2
subject2.onNext("I would be ignored")
subject2.onNext("B")

subject1.onCompleted()
subject2.onNext("C")
```
结果如下：
```
next(1)
next(2)
next(3)
next(B)
next(C)
```
#### 3. merge
- 通过使用`merge`操作符你可以将多个`Observables`合并成一个，当某一个`Observable`发出一个元素时，他就将这个元素发出。
- 如果，某一个`Observable`发出一个`onError`事件，那么被合并的`Observable`也会将它发出，并且立即终止序列。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-4982880854a4f8ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let subject1 = PublishSubject<Int>()
let subject2 = PublishSubject<Int>()

Observable.of(subject1, subject2)
        .merge()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

subject1.onNext(20)
subject1.onNext(40)
subject1.onNext(60)
subject2.onNext(1)
subject1.onNext(80)
subject1.onNext(100)
subject2.onNext(1)
```
结果如下：
```
20
40
60
1
80
100
1
```
#### 4. zip
`zip`操作符将多个(最多不超过8个)`Observables`的元素通过一个函数组合起来，然后将这个组合的结果发出来。它会严格的按照序列的索引数进行组合。例如，返回的`Observable`的第一个元素，是由每一个源`Observables`的第一个元素组合出来的。它的第二个元素 ，是由每一个源`Observables`的第二个元素组合出来的。它的第三个元素 ，是由每一个源`Observables`的第三个元素组合出来的，以此类推。它的元素数量等于源`Observables`中元素数量最少的那个。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-ae17319dd7d0a940.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let first = PublishSubject<String>()
let second = PublishSubject<String>()

Observable.zip(first, second) { $0 + $1 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

first.onNext("1")
second.onNext("A")
first.onNext("2")
second.onNext("B")
second.onNext("C")
second.onNext("D")
first.onNext("3")
first.onNext("4")
```
结果如下：
```
1A
2B
3C
4D
```
#### 5. combineLatest
`combineLatest`操作符将多个`Observables`中最新的元素通过一个函数组合起来，然后将这个组合的结果发出来。这些源`Observables`中任何一个发出一个元素，他都会发出一个元素（前提是，这些`Observables`曾经发出过元素）。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-2fab45bfd4b8c814.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let first = PublishSubject<String>()
let second = PublishSubject<String>()

Observable.combineLatest(first, second) { $0 + $1 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

first.onNext("1")
second.onNext("A")
first.onNext("2")
second.onNext("B")
second.onNext("C")
second.onNext("D")
first.onNext("3")
first.onNext("4")
```
结果如下：
```
1A
2A
2B
2C
2D
3D
4D
```
#### 6. withLatestFrom
`withLatestFrom`操作符将两个`Observables`中最新的元素通过一个函数组合起来，然后将这个组合的结果发出来。当第一个`Observable`发出一个元素时，就立即取出第二个`Observable`中最新的元素，通过一个组合函数将两个最新的元素合并后发送出去。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-9bfeb60b798296e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let subject1 = PublishSubject<String>()
let subject2 = PublishSubject<String>()

subject1.withLatestFrom(subject2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

subject1.onNext("A")
subject2.onNext("1")
subject1.onNext("B")
subject1.onNext("C")
subject2.onNext("2")
subject1.onNext("D")

```
结果如下：
```
1
1
2
```
