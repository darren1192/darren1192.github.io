---
title: RxSwift 操作符决策树
date: 2018-09-30 21:59:24
tags: RxSwift
---
之前列举了很多操作符的用法，还有很多我们没有列举的。其实写了那么多操作符有时候我还是会忘记选择哪一个。这个时候，我发现在[RxSwift中文文档](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree.html)有一篇关于选择操作符的文章。这一篇水文纯搬运。
### 决策树

**我想要创建一个 `Observable`**

*   产生特定的一个元素：[just](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/just.html)
*   经过一段延时：[timer](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/timer.html)
*   从一个序列拉取元素：[from](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/from.html)
*   重复的产生某一个元素：[repeatElement](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/repeatElement.html)
*   存在自定义逻辑：[create](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/create.html)
*   每次订阅时产生：[deferred](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/deferred.html)
*   每隔一段时间，发出一个元素：[interval](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/interval.html)
*   在一段延时后：[timer](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/timer.html)
*   一个空序列，只有一个完成事件：[empty](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/empty.html)
*   一个任何事件都没有产生的序列：[never](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/never.html)

**我想要创建一个 `Observable` 通过组合其他的 `Observables`**

*   任意一个 `Observable` 产生了元素，就发出这个元素：[merge](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/merge.html)
*   让这些 `Observables` 一个接一个的发出元素，当上一个 `Observable` 元素发送完毕后，下一个`Observable` 才能开始发出元素：[concat](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/concat.html)
*   组合多个 `Observables` 的元素
*   当每一个 `Observable` 都发出一个新的元素：[zip](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/zip.html)
*   当任意一个 `Observable` 发出一个新的元素：[combineLatest](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/combineLatest.html)

**我想要转换 `Observable` 的元素后，再将它们发出来**

*   对每个元素直接转换：[map](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/map.html)
*   转换到另一个 `Observable`：[flatMap](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/flatMap.html)
*   只接收最新的元素转换的 `Observable` 所产生的元素：[flatMapLatest](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/flatMapLatest.html)
*   每一个元素转换的 `Observable` 按顺序产生元素：[concatMap](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/concatMap.html)
*   基于所有遍历过的元素： [scan](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/scan.html)

**我想要将产生的每一个元素，拖延一段时间后再发出：[delay](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/delay.html)**

**我想要将产生的事件封装成元素发送出来**

*   将他们封装成 `Event<Element>`：[materialize](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/materialize.html)
*   然后解封出来：[dematerialize](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/dematerialize.html)

**我想要忽略掉所有的 `next` 事件，只接收 `completed` 和 `error` 事件：[ignoreElements](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/ignoreElements.html)**

**我想创建一个新的 `Observable` 在原有的序列前面加入一些元素：[startWith](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/startWith.html)**

**我想从 `Observable` 中收集元素，缓存这些元素之后在发出：[buffer](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/buffer.html)**

**我想将 `Observable` 拆分成多个 `Observables`：[window](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/window.html)**

*   基于元素的共同特征：[groupBy](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/groupBy.html)

**我想只接收 `Observable` 中特定的元素**

*   发出唯一的元素：[single](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/single.html)

**我想重新从 `Observable` 中发出某些元素**

*   通过判定条件过滤出一些元素：[filter](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/filter.html)
*   仅仅发出头几个元素：[take](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/take.html)
*   仅仅发出尾部的几个元素：[takeLast](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/takeLast.html)
*   仅仅发出第 n 个元素：[elementAt](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/elementAt.html)
*   跳过头几个元素
*   跳过头 n 个元素：[skip](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/skip.html)
*   跳过头几个满足判定的元素：[skipWhile](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/skipWhile.html)，[skipWhileWithIndex](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/skipWhile.html)
*   跳过某段时间内产生的头几个元素：[skip](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/skip.html)
*   跳过头几个元素直到另一个 `Observable` 发出一个元素：[skipUntil](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/skipUntil.html)
*   只取头几个元素
*   只取头几个满足判定的元素：[takeWhile](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/takeWhile.html)，[takeWhileWithIndex](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/takeWhile.html)
*   只取某段时间内产生的头几个元素：[take](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/take.html)
*   只取头几个元素直到另一个 `Observable` 发出一个元素：[takeUntil](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/takeUntil.html)
*   周期性的对 `Observable` 抽样：[sample](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/sample.html)
*   发出那些元素，这些元素产生后的特定的时间内，没有新的元素产生：[debounce](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/debounce.html)
*   直到元素的值发生变化，才发出新的元素：[distinctUntilChanged](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/distinctUntilChanged.html)
*   并提供元素是否相等的判定函数：[distinctUntilChanged](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/distinctUntilChanged.html)
*   在开始发出元素时，延时后进行订阅：[delaySubscription](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/delaySubscription.html)

**我想要从一些 `Observables` 中，只取第一个产生元素的 `Observable`：[amb](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/amb.html)**

**我想评估 `Observable` 的全部元素**

*   并且对每个元素应用聚合方法，待所有元素都应用聚合方法后，发出结果：[reduce](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/reduce.html)
*   并且对每个元素应用聚合方法，每次应用聚合方法后，发出结果：[scan](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/scan.html)

**我想把 `Observable` 转换为其他的数据结构：as...**

**我想在某个 [Scheduler](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/schedulers.html) 应用操作符：[subscribeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/subscribeOn.html)**

*   在某个 [Scheduler](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/schedulers.html) 监听：[observeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/observeOn.html)

**我想要 `Observable` 发生某个事件时, 采取某个行动：[do](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/do.html)**

**我想要 `Observable` 发出一个 `error` 事件：[error](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/error.html)**

*   如果规定时间内没有产生元素：[timeout](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/timeout.html)

**我想要 `Observable` 发生错误时，优雅的恢复**

*   如果规定时间内没有产生元素，就切换到备选 `Observable` ：[timeout](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/timeout.html)
*   如果产生错误，将错误替换成某个元素 ：[catchErrorJustReturn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/catchError.html)
*   如果产生错误，就切换到备选 `Observable` ：[catchError](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/catchError.html)
*   如果产生错误，就重试 ：[retry](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/retry.html)

**我创建一个 `Disposable` 资源，使它与 `Observable` 具有相同的寿命：[using](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/using.html)**

**我创建一个 `Observable`，直到我通知它可以产生元素后，才能产生元素：[publish](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/publish.html)**

*   并且，就算是在产生元素后订阅，也要发出全部元素：[replay](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/replay.html)
*   并且，一旦所有观察者取消观察，他就被释放掉：[refCount](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/refCount.html)
*   通知它可以产生元素了：[connect](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/connect.html)

笨鸟先飞，执行合一。基本了解`RxSwift`后，就应该动手写一些BUG了。
