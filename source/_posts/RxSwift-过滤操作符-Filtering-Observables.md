---
title: RxSwift 过滤操作符(Filtering Observables)
date: 2018-09-26 21:30:34
tags: RxSwift
---
过滤操作指的是从源`Observable`中选择特定的数据发送。
#### 1. filter
`filter`操作符将通过你提供的判定方法过滤一个`Observable`。
![filter.png](https://upload-images.jianshu.io/upload_images/2855070-7403d9c225aec4cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let disposeBag = DisposeBag()
Observable.of(2, 30, 22, 5, 60, 1)
        .filter{ $0 > 10 }
        .subscribe(onNext: { print( $0 )})
        .disposed(by: disposeBag)
```
输出结果：
```
30
22
60
```
#### 2. distinctUntilChanged
`distinctUntilChanged`操作符将阻止`Observable`发出相同的元素。如果后一个元素和前一个元素是相同的，那么这个元素将不会被发出来。如果后一个元素和前一个元素不相同，那么这个元素才会被发出来。
![distinctUntilChanged.png](https://upload-images.jianshu.io/upload_images/2855070-0c5afa37f9610704.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 2, 1, 3)
        .distinctUntilChanged()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
输出结果：
```
1
2
1
3
```
#### 3. single
- 限制只发送一次事件，或者满足条件的第一个事件。
- 如果存在有多个事件或者没有事件都会发出一个`error`事件。
- 如果只有一个事件，则不会发出`error`事件

![single.png](https://upload-images.jianshu.io/upload_images/2855070-29bfd143ec29394e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4, 4, 4, 5)
        .single()
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)

Observable.of("A")
        .single()
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
1
Unhandled error happened: Sequence contains more than one element.
subscription called from:
A
```
#### 4. elementAt
`elementAt`操作符将拉取`Observable`序列中指定索引数的元素，然后将它作为唯一的元素发出。
![elementAt.png](https://upload-images.jianshu.io/upload_images/2855070-b8ac9409cc9bf25d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4)
        .elementAt(2)
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
3
```
#### 5. ignoreElements
- 该操作符可以忽略掉所有的元素，只发出`error`或`completed`事件。
- 如果我们并不关心`Observable`的任何元素，只想知道`Observable`在什么时候终止，那就可以使用`ignoreElements`操作符。

![ignoreElements.png](https://upload-images.jianshu.io/upload_images/2855070-bcadae47ddc7f71e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4, 4, 4, 5)
        .ignoreElements()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
```
结果如下：
```
completed
```
#### 6.take
该方法实现仅发送`Observable`序列中的前`n`个事件，在满足数量之后会自动`.completed`
![take.png](https://upload-images.jianshu.io/upload_images/2855070-ba890655a1787c13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4)
        .take(2)
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
1
2
```
#### 7. takeLast
该方法实现仅发送`Observable`序列中的后`n`个事件
![takeLast.png](https://upload-images.jianshu.io/upload_images/2855070-3e60ff6e411e234c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4)
        .takeLast(1)
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
```
结果如下：
```
4
```
#### 8. skip
该方法用于跳过源`Observable`序列发出的前`n`个事件。
![skip.png](https://upload-images.jianshu.io/upload_images/2855070-439c4618a9901d45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()
Observable.of(1, 2, 3, 4)
        .skip(2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
结果如下：
```
3
4
```
#### 9. Sample
- `Sample`除了订阅源`Observable`外，还可以监视另外一个`Observable`， 即`notifier`。
每当收到`notifier`事件，就会从源序列取一个最新的事件并发送。而如果两次`notifier`事件之间没有源序列的事件，则不发送值。

![sample.png](https://upload-images.jianshu.io/upload_images/2855070-509212ce29f1c102.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
let disposeBag = DisposeBag()

let source = PublishSubject<Int>()
let notifier = PublishSubject<String>()

source.sample(notifier)
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)

source.onNext(1)

notifier.onNext("A")

source.onNext(2)

notifier.onNext("B")
notifier.onNext("C")

source.onNext(3)
source.onNext(4)

notifier.onNext("D")

source.onNext(5)

notifier.onCompleted()
```
结果如下：
```
1
2
4
5
```
#### 10. debounce
- `debounce`操作符可以用来_过滤掉高频产生的元素_，它只会发出这种元素：该元素产生后，一段时间内没有新元素产生。
- 换句话说就是，队列中的元素如果和下一个元素的间隔小于了指定的时间间隔，那么这个元素将被过滤掉。
- `debounce`常用在_用户输入_的时候，不需要每个字母敲进去都发送一个事件，而是稍等一下取最后一个事件。

![debounce.png](https://upload-images.jianshu.io/upload_images/2855070-b313fa33f47012ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let disposeBag = DisposeBag()

//定义好每个事件里的值以及发送的时间
let times = [
[ "value": 1, "time": 0.1 ],
[ "value": 2, "time": 1.1 ],
[ "value": 3, "time": 1.2 ],
[ "value": 4, "time": 1.2 ],
[ "value": 5, "time": 1.4 ],
[ "value": 6, "time": 2.1 ]
]

//生成对应的 Observable 序列并订阅
Observable.from(times)
        .flatMap { item in
            return Observable.of(Int(item["value"]!))
                            .delaySubscription(Double(item["time"]!),scheduler: MainScheduler.instance)
                }
        .debounce(0.5, scheduler: MainScheduler.instance) //只发出与下一个间隔超过0.5秒的元素
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
```
运行结果......为啥我啥也没打印出来？？？
![u=4240739968,2514380758&fm=27&gp=0.jpg](https://upload-images.jianshu.io/upload_images/2855070-bafa66b6f115f0c9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个留在下回思考......
但据可靠信息，咳咳咳，打印出来的应该是
```
1
5
6
```

