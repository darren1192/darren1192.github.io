---
title: RxSwift 变换操作(Transforming Observables)
date: 2018-09-25 21:52:23
tags: RxSwift
---
变换操作指的是对原始的`Observable`序列进行一些转换，类似于 Swift 中`CollectionType`的各种转换
#### 1. buffer
`buffer`操作符将缓存`Observable`中发出的新元素，当元素达到某个数量，或者经过了特定的时间，它就会将这个元素集合发送出来。
![buffer.png](https://upload-images.jianshu.io/upload_images/2855070-76c121cb942f7c65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
🌰：
```
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()

subject.buffer(timeSpan: 1, count: 3, scheduler: MainScheduler.instance)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
subject.onNext("A")
subject.onNext("B")
subject.onNext("C")

subject.onNext("🐯")
subject.onNext("🐭")
subject.onNext("🐱")
```
运行结果如下：
```
next(["A", "B", "C"])
next(["🐯", "🐭", "🐱"])
```
有一点疑问，对于`buffer`在`RxSwift`的解释是
>Projects each element of an observable sequence into a buffer that's sent out when either it's full or a given amount of time has elapsed, using the specified scheduler to run timers

`RxSwift`中，它的意思是当缓冲区满了或超过了给定的时间时，就会发出缓冲区。可是如果我们把上述代码稍微改变一下
```
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()

subject.buffer(timeSpan: 1, count: 3, scheduler: MainScheduler.instance)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
subject.onNext("A")
subject.onNext("B")
subject.onNext("C")

//  subject.onNext("🐯")
subject.onNext("🐭")
subject.onNext("🐱")
```
运行结果如下：
```
next(["A", "B", "C"])
```
并没有后面的内容。因为我在很多文章里都看到说如果规定时间内不够要求的数量也会发出（有几个发几个，一个都没有发空数组 []）。所以，这点值得注意。
#### 2. window
- `window`操作符和`buffer`十分相似。不过`buffer`是周期性的将缓存的元素集合发送出来，而`window`周期性的将元素集合以`Observable`的形态发送出来。
同时`buffer`要等到元素搜集完毕后，才会发出元素序列。而`window`可以实时发出元素序列。(这一点证明了`buffer`还是要元素搜集完毕后才会发出序列)
![window.png](https://upload-images.jianshu.io/upload_images/2855070-4658ffd7d827e9c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先我们先这么写：
```
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()

subject.window(timeSpan: 1, count: 3, scheduler: MainScheduler.instance)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)

subject.onNext("A")
subject.onNext("B")
subject.onNext("C")

subject.onNext("🐯")
subject.onNext("🐭")
subject.onNext("🐱")
```
运行结果如下：
```
next(RxSwift.AddRef<Swift.String>)
next(RxSwift.AddRef<Swift.String>)
next(RxSwift.AddRef<Swift.String>)
next(RxSwift.AddRef<Swift.String>)
......
```
可见他在不断打印`next(RxSwift.AddRef<Swift.String>)`
如果把代码改成这样：
```
let disposeBag = DisposeBag()
let subject = PublishSubject<String>()

subject.window(timeSpan: 1, count: 3, scheduler: MainScheduler.instance)
    .subscribe(onNext: {
        print("subscribe:\($0)")
        $0.asObservable()
            .subscribe{ print($0) }
            .disposed(by: disposeBag)
    })
    .disposed(by: disposeBag)

subject.onNext("A")
subject.onNext("B")
subject.onNext("C")

subject.onNext("🐯")
subject.onNext("🐭")
subject.onNext("🐱")
```
运行结果如下：
```
subscribe:RxSwift.AddRef<Swift.String>
next(A)
next(B)
next(C)
completed
subscribe:RxSwift.AddRef<Swift.String>
next(🐯)
next(🐭)
next(🐱)
completed
subscribe:RxSwift.AddRef<Swift.String>
completed
subscribe:RxSwift.AddRef<Swift.String>
completed
```
#### 3.map
`map`操作符将`Observable`的每个元素应用你提供的转换方法，然后返回含有转换结果的`Observable`。
![map.png](https://upload-images.jianshu.io/upload_images/2855070-def03bf478a9770d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1,2,3)
        .map{ $0 * 2 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
运行结果如下：
```
2
4
6
```
其实，swift中也有一个高级函数`map`，可以对数组中的每一个元素做一次处理
```
let array = [1, 2, 3]
let nums = array.map {
    return $0 * 2
}
print(nums)
```
结果得出：
```
[2, 4, 6]
```
#### 4. flatMap
- `map`在做转换的时候容易出现“升维”的情况。即转变之后，从一个序列变成了一个序列的序列。
- 而`flatMap`操作符会对源`Observable`的每一个元素应用一个转换方法，将他们转换成`Observables`。 然后将这些`Observables`的元素合并之后再发送出来。即又将其 "拍扁"（降维）成一个`Observable`序列。
- 这个操作符是非常有用的。比如当`Observable`的元素本生拥有其他的`Observable`时，我们可以将所有子`Observables`的元素发送出来。

![flatMap.png](https://upload-images.jianshu.io/upload_images/2855070-f40ff203991060c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let disposeBag = DisposeBag()
let first = BehaviorSubject.init(value: "A")
let second = BehaviorSubject.init(value: "B")
let variable = Variable.init(first)

variable.asObservable()
        .flatMap{ $0 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
first.onNext("1")
variable.value = second
second.onNext("2")
first.onNext("3")
```
打印结果如下：
```
A
1
B
2
3
```
我们依然可以在`Swift`中找到高级函数`flatMap`，它相比`map`有两点不同
- `flatMap`返回后的数组中不存在`nil`，同时它会把`Optional`解包
- `flatMap`还能把数组中存有数组的数组（二维数组、N维数组）一同打开变成一个新的数组
- 也能把两个不同的数组合并成一个数组，这个合并的数组元素个数是前面两个数组元素个数的乘积

只举一个🌰：
```
let array = [[1, 2, 3], [4, 5, 6]]
let nums = array.flatMap{ $0 }
print(nums)
```
运行结果如下：
```
[1, 2, 3, 4, 5, 6]
```
#### 5. flatMapLatest
`flatMapLatest`与`flatMap`的唯一区别是：`flatMapLatest`只会接收最新的`value`事件
![flatMapLatest.png](https://upload-images.jianshu.io/upload_images/2855070-ac2d4f8e45ea1ebb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将上述代码中`flatMap`改为`flatMapLatest`
```
let disposeBag = DisposeBag()
let first = BehaviorSubject.init(value: "A")
let second = BehaviorSubject.init(value: "B")
let variable = Variable.init(first)

variable.asObservable()
        .flatMapLatest{ $0 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
first.onNext("1")
variable.value = second
second.onNext("2")
first.onNext("3")
```
运行结果如下：
```
A
1
B
2
```
#### 6. concatMap
`concatMap`操作符将源`Observable`的每一个元素应用一个转换方法，将他们转换成`Observables`。然后让这些`Observables` 按顺序的发出元素，当前一个`Observable`元素发送完毕后，后一个`Observable`才可以开始发出元素。等待前一个`Observable `产生完成事件后，才对后一个`Observable`进行订阅
![concatMap.png](https://upload-images.jianshu.io/upload_images/2855070-5969d4c9a2013318.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
let first = BehaviorSubject.init(value: "A")
let second = BehaviorSubject.init(value: "B")
let variable = Variable.init(first)

variable.asObservable()
        .concatMap{ $0 }
        .subscribe(onNext: { print($0) })   
        .disposed(by: disposeBag)
first.onNext("1")
first.onNext("2")

variable.value = second
second.onNext("3")
second.onNext("4")

first.onCompleted()
second.onNext("5")
```
运行结果如下：
```
A
1
2
4
5
```
#### 7. scan
`scan`操作符将对第一个元素应用一个函数，将结果作为第一个元素发出。然后，将结果作为参数填入到第二个元素的应用函数中，创建第二个元素。以此类推，直到遍历完全部的元素。
![scan.png](https://upload-images.jianshu.io/upload_images/2855070-83cf372dd5b1cbe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4, 5)
        .scan(0) { acum, elem  in
            acum + elem
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
运行结果如下：
```
1
3
6
10
15
```
#### 8. groupBy
- `groupBy`操作符将源`Observable`分解为多个子`Observable`，然后将这些子`Observable`发送出来。
- 也就是说该操作符会将元素通过某个键进行分组，然后将分组后的元素序列以`Observable`的形态发送出来。
![groupBy.png](https://upload-images.jianshu.io/upload_images/2855070-4193d29d366c8e24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable<Int>.of(0, 1, 2, 3, 4, 5)
        .groupBy { (element) -> String in
            return element % 2 == 0 ? "偶数" : "奇数"
        }
        .subscribe { (event) in
            switch event {
            case .next(let group):
                group.asObservable().subscribe({ (event) in
                    print("key:\(group.key)  event:\(event)")
                })
                .disposed(by: disposeBag)
            default:
                print("")
            }
}.disposed(by: disposeBag)
```
运行结果如下：
```
key:偶数  event:next(0)
key:奇数  event:next(1)
key:偶数  event:next(2)
key:奇数  event:next(3)
key:偶数  event:next(4)
key:奇数  event:next(5)
key:奇数  event:completed
key:偶数  event:completed
```
