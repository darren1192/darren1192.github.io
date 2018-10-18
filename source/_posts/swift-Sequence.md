---
title: swift 内联序列函数sequence
date: 2018-09-13 20:24:08
tags: swift 冷门方法
---
在看RxSwift文章的时候，有人说到了Sequence。恕小弟才疏学浅，在日常搬砖中没有使用到它，所以在此记录一下。 
sequence是在swift3就出现的。使用它们可以返回一个无限序列。我们可以给他们一个初始值，或者初始状态，然后他们便会以懒加载的方式应用到一个闭包。方法如下：
```
1.public func sequence<T>(first: T, next: @escaping (T) -> T?) -> UnfoldSequence<T, (T?, Bool)>
2.public func sequence<T, State>(state: State, next: @escaping (inout State) -> T?) -> UnfoldSequence<T, State>
```
### 方法1：
```
public func sequence<T>(first: T, next: @escaping (T) -> T?) -> UnfoldSequence<T, (T?, Bool)>
```
first:从序列返回的第一个元素
next:一个闭包，它接受前一个sequence元素并返回下一个元素
举个🌰，我们想打印从1-100范围内所有的偶数
```
for i in sequence(first: 1, next: { $0 * 2}){
    if i > 100{
        break
    }
    print(i)
}
```
或者，我们也可以把处理逻辑，状态判断放在next闭包里面，🌰：
```
for _ in sequence(first: 1, next: {
    print($0)
    let value = $0 * 2
    return value <= 100 ? value : nil
}){}
```
在文档中还有一个🌰，从某一个树节点一直向上遍历到根节点
```
for node in sequence(first: leaf, next: { $0.parent }) {
    // node is leaf, then leaf.parent, then leaf.parent.parent, etc.
}
```
### 方法2：
```
public func sequence<T, State>(state: State, next: @escaping (inout State) -> T?) -> UnfoldSequence<T, State>
```
这个方法是设置个初始状态（可变的），后面将其传 入next 闭包改变状态，并获取下一个序列，依次类推。举个🌰：我们制定X轴Y轴最大值，列出内部所有的整点：
```
// 实现方法
func cartesianSequence(xCount: Int, yCount: Int) -> UnfoldSequence<PointType, Int>{
    // assert断言
    assert(xCount > 0 && yCount > 0, "必须使用正整数创建序列")
    return sequence(state: 0, next: { (index: inout Int) -> PointType? in
    guard index < xCount * yCount else {
        return nil
        }
    defer{
        index += 1
    }
    return (x: index % xCount, y :index / xCount)
    })
}
// 调用方法
for point in cartesianSequence(xCount: 3, yCount: 3){
    print("x:\(point.x), y:\(point.y)")
    /*
    x:0, y:0 x:1, y:0 x:2, y:0
    x:0, y:1 x:1, y:1 x:2, y:1 
    x:0, y:2 x:1, y:2 x:2, y:2
    */
}
```
