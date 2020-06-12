---
title: RxJava源码解析（一）
date: 2020-06-12 18:13:12
tags:
---
### 前言
因为 rxjava 现有的功能十分强大，如果直接看 rxjava 源码可能被其他复杂的逻辑绕晕，所以这边文章通过自己写一个简单的 rxjava 库来了解 rxjava 的核心功能是怎么实现的。

rxjava 的核心思想是观察者模式。观察者模式中有两个重要的对应观察者和被观察者，在 rxjava 中也代表着订阅者和数据发布者，我们先新建两个接口表示这两个对象。

```kotlin
interface Observer<T> {

    fun onSubscribe()

    fun onNext(t : T)

    fun onComplete()

}
```
```kotlin
interface ObservableSource<T> {

    fun subscribe(observer : Observer<T>)

}
```
当我们需要观察数据(订阅数据)的时候调用 ObservableSource.subscribe(Observer) 。然后我们在具体的实现数据发布者(ObservableSource) 的逻辑。 
```
interface ObservableOnSubscribe<T> {

    fun subscribe(observer: Observer<T>)

}
```
当订阅开始时，调用 observableOnSubscribe 的 subscribe 去发布数据
```
class ObservableCreate<T>(private val observableOnSubscribe: ObservableOnSubscribe<T>) : SimpleObservable<T>() {

    override fun subscribe(observer: Observer<T>) {
        observableOnSubscribe.subscribe(observer)
    }

}
```

``` 然后通过 SimpleObservable 的静态方法去返回这些数据发布者。
open abstract class SimpleObservable<T> : ObservableSource<T> {
    companion object {
        fun <T> create(observableOnSubscribe: ObservableOnSubscribe<T>): SimpleObservable<T> {
            return ObservableCreate(observableOnSubscribe)
        }

        fun <T> fromArray(vararg items: T): SimpleObservable<T> {
            return create(object : ObservableOnSubscribe<T> {
                override fun subscribe(observer: Observer<T>) {
                    observer.onSubscribe()
                    for (item in items) {
                        observer.onNext(item)
                    }
                    observer.onComplete()
                }

            })
        }

        fun <T> fromIterable(source: Iterable<T>): SimpleObservable<T> {
            return create(object : ObservableOnSubscribe<T> {
                override fun subscribe(observer: Observer<T>) {
                    observer.onSubscribe()
                    var iterator = source.iterator()
                    while (iterator.hasNext()) {
                        observer.onNext(iterator.next())
                    }
                    observer.onComplete()
                }
            })
        }

        fun <T> just(item: T): SimpleObservable<T> {
            return fromArray(item)
        }

        fun <T> just(item1: T, item2: T): SimpleObservable<T> {
            return fromArray(item1, item2)
        }
    }
}
```
