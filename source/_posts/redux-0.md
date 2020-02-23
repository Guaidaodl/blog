---
title: Android 开心明星脸 -- Redux(0) - 初体验
tags: 'Android, Redux, MVVM'
date: 2020-02-23 23:47:41
---


# 开心明星脸
首先解释一下这个系列的名称的来源, 开心明星脸是很久以前在东南电视台播放的一个综艺
节目. 主要内容就是请找一些长得跟明星很想的普通人, 比如低配刘德华, 走调张学友之类.

# 为什么会有这个系列

最近正在尝试使用 MVVM 的架构, 遇到很多实践上的问题, 想从业内的一些经验中学习. Android 这边
MVVM 起步比较晚, 可参考的不是很多. 于是想从在 MVVM 上迭代了很久的前端框架上学习一下经验. 
然后在 Android 上简单的模仿.

暂定的目标是 React+Redux 和 Angular.js

# 实践上遇到的问题.

我觉得 MVVM 的核心关键在于如何管理状态(ViewModel), 下面是一些我想知道答案的问题:

1. 一个复杂的页面应该有一个 ViewModel 统一管理, 还是有多个 ViewModel?
2. 如果有多个 ViewModel, 那多个 ViewModel 之前如何同步数据?
3. 单一的 ViewModel 如何处理复杂度增加问题?
4. 那嵌套很深的组件如何获取 ViewModel?
5. 异步任务如何和 ViewModel 协调?
6. 弹个 Toast 显示失败信息这个一次性任务如何通过 ViewModel 实现?

下面就正式开始, 因为相比 Angular.js, Redux 要简单得多, 所以先从 Redux 开始.

# Redux

关于 redux 的信息主要参考了 [深入浅出 React 和 Redux](https://book.douban.com/subject/27033213/) 
这本书.

## Redux 的基本原则

Redux 是 React 上一个比较流行的状态管理库. 从 Dan Abramov 改进 Flux 而来. 有三大原则:

- 唯一数据源(Single Source Of Truth)
- 保持状态只读(State is read-only)
- 数据改变只能通过纯函数来完成(Changes are mde with pure functions)

三个原则中的第一条正好解决了我的第一个疑问. Redux 是从 Flux 发展改进而来. Flux 采用了多个 
Store 的方案, 但是多个 Store 之前的依赖关系十分难以管理, 所以 Redux 采用了单一 Store 
的方案.

## Redux 的核心概念

### State
状态, 应该是一个只读不可变的对象.

### Action
操作.

### Store
存储状态(State)的对象. 对外主要有以下几个功能:

1. 获取当前的 state.
2. 订阅状态变化.
3. 分发(dispatch) Action.

### Reducer
Redux 的核心概念. 也是 Redux 中名字的来源. Store 只负责分发 action, 真正处理 action 的
就是 Reducer. Reducer 其实就是一个函数. 接受 state 和 action 两个参数, 并返回一个新的 
State. Redux 框架强调了 reducer 最好是**纯函数**.

# 简单实践
其实 redux 的核心概念比较简单. 所以模仿起来也比较简单.

## 技术方案

### State
State 一个普通不可变的 Java 对象, 因为要经常会有利用旧 State, 小改一个属性来创建一个新的 state
这种操作, JS 里可以用 [spread operator](https://www.techiediaries.com/react-spread-operator-props-setstate/) 
在 Java 里可能有点麻烦, 但是如果使用 Kotlin, 则 data class 完美匹配这个需求.

### Action
这个没有什么好说的. 就是一个普通对象. 考虑到所有的 action 类型应该是已知的, 可以使用 kotlin 的
sealed class.

### Store
可以对 LiveData 进行一些简单的扩展来实现.

### Reducer
一个单方法接口就可以.

总体来说是比较轻量级的, Redux 的源码其实也不是很多. 只是提出核心的概念, 如果想要真正用起来还需要很多辅助的库. 

## 实现
因为要实现的功能比较少, 其实就只有 Store 是需要定义接口与创建方法. 利用 LiveData 可以比较简单的实现. 
核心代码如下, 完整代码在[这里](https://github.com/Guaidaodl/Android-Redux/blob/master/Andux-Base/app/src/main/java/me/guaidaodl/andux/core/Store.kt)

``` Kotlin
/**
 * 跟 Lifecycle 绑定的 Store
 */
interface Store<State, Action> {

    val state: State

    fun dispatch(action: Action)

    // 照抄 LiveData 里的 observer 相关的接口.

    fun observe(owner: LifecycleOwner, observer: Observer<in State>)
    fun observeForever(observer: Observer<in State>)
    fun removeObserver(observer: Observer<in State>)
    fun removeObservers(owner: LifecycleOwner)
}

/**
 * 创建利用 ViewModel 和 LiveData 来实现 Store
 */
fun <State, Action> createStore(initState: State, reducer: (State, Action) -> State): Store<State, Action> {
    return object: Store<State, Action> {
        private val liveState = MutableLiveData(initState)

        override val state: State get() = liveState.value!!

        override fun dispatch(action: Action) {
            liveState.value = reducer(state, action)
        }

        // ...
    }
}
```
在实现 createStore 函数的时候考虑过是否直接把 Store 写一个 ViewModel 的子类, 二者功能十分类似.
但是考虑到 Redux 强调单一数据源, 一个 store 的生命周期可能横跨多个页面. 最后还是放弃了这个想法.

另外看了一些 Android 官方关于 ViewModel 和 LiveData 的文章, 感觉思路跟 Flux 差不多.


## 简单 Demo
这里用刚才写的组件写了一个简单的 [demo](https://github.com/Guaidaodl/Android-Redux/tree/master/Andux-Base). 
因为主要关注点在状态管理上, 数据和 UI 绑定的代码没有用任何框架, 长得比较丑, 不用在意.


# 总结
Redux 的概念很简单也很容易实现. 不过现在的内容只解决了是单一数据源还是多个数据源的问题.
后续的问题的答案还有待继续探究.

其实这里的单数据源在我理解里并不是一个应用只有一个 Store 的意思, 而是在同一业务的业务周期内单一.
不过这样又会涉及到很多问题, 比如一个页面多个 Tab, 每一个 Tab 都是单独的业务怎么处理等. 
还是需要继续研究在业务逻辑复杂的情况下如何使用 Redux.


