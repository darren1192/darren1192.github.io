---
title: RxSwift Observer-观察者
date: 2018-09-21 23:18:04
tags: RxSwift
---
我们在[之前](https://darren1192.github.io/2018/09/19/RxSwift-%E6%A0%B8%E5%BF%83/#more)已经了解了什么是`Observer`观察者，这篇我们了解一下怎么创建观察者以及特征观察者(`AnyObserver`、`Binder`)。
#### 在 subscribe 方法中创建
创建观察者最直接的方法就是在`Observable`的`subscribe`方法后面描述当事件发生时，需要如何做出响应。举个🌰：
```
let observable = Observable.of(1,2,3,4,5)
observable.subscribe(onNext: { element in
    print(element)
}, onError: { error in
    print(error)
}, onCompleted: {
    print("completed")
})
```
运行结果如下：
![运行结果.png](https://upload-images.jianshu.io/upload_images/2855070-b7c14b064cc5ab89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 在bind方法中创建
我们创建一个定时生成索引数的`Observable`序列，并将索引数不断显示在`label`标签上
举个🌰：
```
import UIKit
import RxCocoa
import RxSwift

class ViewController: UIViewController {

    @IBOutlet weak var label: UILabel!
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        observable.map{ "当前索引数:\($0)" }
                .bind { [weak self](text) in
                    self?.label.text = text
                }.disposed(by: disposeBag)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
运行结果：
![bind结果.png](https://upload-images.jianshu.io/upload_images/2855070-c9f8b9f5593b1ffd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
除了以上创建方法外，我们还可以使用其他的方式，比如`AnyObserver`和`Binder `
#### 使用AnyObserver创建观察者
`AnyObserver`可以用来描叙任意一种观察者
##### 配合`subscribe `方法使用
```
let observer: AnyObserver<Int> = AnyObserver { event in
    switch event{
    case .next(let data):
        print(data)
    case .error(let error):
        print(error)
    case .completed:
        print("completed")
    }
}
let observable = Observable.of(1,2,3,4,5)
observable.subscribe(observer)
```
运行结果如下：
![运行结果.png](https://upload-images.jianshu.io/upload_images/2855070-b7c14b064cc5ab89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 配合bindTo方法使用
```
import UIKit
import RxCocoa
import RxSwift

class ViewController: UIViewController {

    @IBOutlet weak var label: UILabel!
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        let observer: AnyObserver<String> = AnyObserver{
            [weak self] event in
            switch event{
            case .next(let text):
                self?.label.text = text
            default:
            break
            }
        }
        let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        observable.map{"当前索引数:\($0)"}
                .bind(to: observer)
                .disposed(by: disposeBag )
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```
#### Binder
Binder 主要有以下两个特征：
- 不会处理错误事件
- 确保绑定都是在给定`Schedule`上执行（默认`MainScheduler`）

一旦产生错误事件，在调试环境下将执行`fatalError`，在发布环境下将打印错误信息
在上面更新`label`文字的例子中，更好的方式就是使用`Binder`。理由有二：
- UI的更新在主线程完成
- 只处理`next`事件
```
import UIKit
import RxCocoa
import RxSwift

class ViewController: UIViewController {

    @IBOutlet weak var label: UILabel!
    let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        let observer: Binder<String> = Binder(label){
            (view,text) in
            view.text = text
        }
        let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        observable.map{ "当前索引数:\($0)"}
                .bind(to: observer)
                .disposed(by: disposeBag)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
下面，我们再去实现另外一段代码：
```
let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
observable.map{ $0 % 2 == 0}
        .bind(to: label.rx.isHidden)
        .disposed(by: disposeBag)
```
此时label会不断的消失、出现。这段代码里我们又操作了什么？查看`label.rx.isHidden`中`isHidden`可以发现
```
extension Reactive where Base: UIView {
    public var isHidden: Binder<Bool> {
        return Binder(self.base) { view, hidden in
            view.isHidden = hidden
        }
    }
}
```
其实`RxCocoa`在对许多 UI 控件进行扩展时，就利用`Binder`将控件属性变成观查者，我们也可以用这种方式来创建自定义的 UI 观察者。





