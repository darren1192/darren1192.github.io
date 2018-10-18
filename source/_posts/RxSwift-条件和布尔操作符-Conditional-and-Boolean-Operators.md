---
title: RxSwift 条件和布尔操作符(Conditional and Boolean Operators)
date: 2018-09-27 21:31:24
tags: RxSwift
---
条件和布尔操作会根据条件发射或变换`Observables`，或者对他们做布尔运算。
#### 1.amb
当你传入多个`Observables`到`amb`操作符时，它将取其中一个`Observable`：第一个产生事件的那个`Observable`，可以是一个`next`，`error`或者`completed`事件。`amb`将忽略掉其他的`Observables`。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-84522567d0c59097.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let subject1 = PublishSubject<Int>()
let subject2 = PublishSubject<Int>()
let subject3 = PublishSubject<Int>()

subject1.amb(subject2)
        .amb(subject3)
        .subscribe{ print($0.element!) }
        .disposed(by: disposeBag)

subject2.onNext(1)
subject1.onNext(20)
subject2.onNext(2)
subject1.onNext(40)
subject3.onNext(0)
subject2.onNext(3)
subject1.onNext(60)
subject3.onNext(0)
subject3.onNext(0)
```
运行结果：
```
1
2
3
```
#### 2. takeWhile
该方法依次判断`Observable`序列的每一个值是否满足给定的条件。 当第一个不满足条件的值出现时，它便自动完成。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-df28e3c3f9466030.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 3, 2, 1)
        .takeWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
运行结果：
```
1
2
3
```
#### 3. takeUntil
`takeUntil`操作符将镜像源`Observable`，它同时观测第二个`Observable`。一旦第二个`Observable`发出一个元素或者产生一个终止事件，那个镜像的`Observable`将立即终止。

![takeUntil.png](https://upload-images.jianshu.io/upload_images/2855070-0445e18465f29f93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let subject1 = PublishSubject<String>()
let subject2 = PublishSubject<String>()

subject1.takeUntil(subject2)
        .subscribe { print($0) }
        .disposed(by: disposeBag)

subject1.onNext("A")
subject1.onNext("B")
subject2.onNext("我可以让它中止")
subject1.onNext("C")
```
结果如下：
```
next(A)
next(B)
completed
```
#### 4. skipUntil
- 同上面的`takeUntil`一样，`skipUntil`除了订阅源`Observable`外，通过`skipUntil`方法我们还可以监视另外一个`Observable`， 即`notifier` 。
- 与`takeUntil`相反的是。源`Observable`序列事件默认会一直跳过，直到`notifier`发出值或 complete 通知

![image.png](https://upload-images.jianshu.io/upload_images/2855070-5063471ccc8c9bc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let subject = PublishSubject<String>()
let notifier = PublishSubject<String>()

subject.skipUntil(notifier)
        .subscribe (onNext: { print($0) })
        .disposed(by: disposeBag)

subject.onNext("A")
subject.onNext("B")
notifier.onNext("接收消息")
subject.onNext("C")
subject.onNext("D")
subject.onNext("E")
notifier.onNext("接收消息")
subject.onNext("F")
```
结果如下：
```
C
D
E
F
```
#### 5. skipWhile
`skipWhile`操作符可以让你忽略源`Observable`中头几个元素，直到元素的判定为否。
![image.png](https://upload-images.jianshu.io/upload_images/2855070-cf1e93804e5b79e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

Observable.of(1, 3, 6, 4, 7, 2)
        .skipWhile{ $0 < 5 }
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
6
4
7
2
```
