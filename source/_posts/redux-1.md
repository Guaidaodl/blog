---
title: Android 开心明星脸 -- Redux(1) - 传递 ViewModel
tags: 
- Android
- Redux
- MVVM
date: 2020-02-27 18:21:58
categories: dev
---

{% post_link redux-0 '上一篇文章' %}中我列出了关于 MVVM 实践中的问题. 其中第 4 个问题是

> 4. 那嵌套很深的组件如何获取 ViewModel?

为什么跳过了第二个和第三个问题呢? 难道有什么深意? 不, 只是因为我看的书先讨论到了这个问题

其实说到这个问题, 作为一个资深的面向 Github 编程的 Java 程序员, 我脑海中的复现的第一个词是依赖注入.
DI 确实是最容易想到, 也可能是最方便的做法. 不过我们还是先看看 Redux 的方案, 然后再回头看看 DI 的
方案.

# 学习

## 单例, 全局变量
这个其实最容易想到. 既然整个应用都只有一个 Store, 那我直接把这个 Store 做成单例. 那每一个组件 
需要的时候就可以直接拿到了. 上一篇的 
[demo](https://github.com/Guaidaodl/Android-Redux/tree/master/Andux-Base) 就把 
Store 保存在一个全局变量中.

这样做在一个小 demo 中当然没有问题, 但是在大项目中明显是行不通的, 组件与 Store 强耦合, 无法复用.

## react-redux
前面说到 Redux 的核心概念和代码都比较简单, 如果想要比较好的工作需要需要一些其他库的辅助. 
react-redux 就是其中一个, 该库主要是解决 react 组件如何和 redux 绑定问题. 
下面我们来分析这个库的实现思路.

### 容器组件和展示组件
从上一篇的例子可以看出, 一个组件要工作主要有两个职责:

1. 与 Store 交互. 从 Store 中读取自己需要的信息(props), 在合适的时机派发 action 等.
2. 根据读取的信息来渲染界面.

其实可以把两个功能分开到不同的组件. 然后通过组合的方式来实现功能. 通常把负责与 Store 交互的组件叫做容器组件
(Container Component), 负责渲染的组件成为展示组件(Presentational Component). 
也叫聪明组件(Smart Component) 和傻瓜组件(Dumb Component). 如果了解 Flutter, 这两种组件可以映射到
StatefulWidge 和 StatelessWidget.

通过把一个简单的组件划分成两层以后, 实际负责渲染的组件跟 Store 解耦. 更容易拿来复用. 

### Provider
React 有一个叫 context 特性, 只要一棵组件树的根节点提供了 Context, 那其他的子节点就可以访问到这个 Context.
react-redux 这个库利用了这个特性, 将组件树的根节点换成了一个定义好的 Provider, 这个 Provider
的功能就是提供一个含有 store 的 Context.

经过上面的两个操作. 像上一个 demo 那种简单的组件结构变化如下.

![图片](/images/Redux-1-1.png)

可以看到变得复杂了一点, 不过最底下的两个 Counter 可以**复用**了.

# 模仿

这个模仿起来还挺麻烦的. 主要两个原因, 一是 react 和 Android 传统 UI 在设计上有巨大的区别.
而是因为 Java 不能动态创建类型, 而 react-redux 库中用到了这项技术.

- 设计上的区别
  React 中有一个非常核心的概念是 VDOM(Virtual DOM), VDOM 的创建和修改的代价比较小, 所以 react
  框架可以在每次 State 更新的时候重新创建一个 VDOM 树, VDOM 的层次增加也不容易引起性能问题. 
  但是在 Android 之中, 并没有与 VDOM 近似的概念(排除 Flutter 与不成熟的 Jetpack Compose),
  每次都重建整个 View Tree 的话代价巨大, 增加层次也有很大的性能问题. 因此在上一篇的 demo 中, 
  当 Store 通知 State 变化时并没有重新一个新的 View Tree, 而是更新现有的 View.

  因为同样的原因, 也不能直接像 react-redux 这样通过增加 Provider 和 Container 来解决问题.
  其实 Provider 和 Container 的指责都和 Android 传统的 View 不一样, 
  我猜可能因为 react 有 Flutter 的万物皆 Widget 类似的原则, 所以才使用了现在的方法.

- 动态创建类型
  在 react-redux 库中 Container Component 的代码比较模式化, 所以该库提供了一个 connect 
  方法来动态创建 Container, 函数的定义如下: 
  ``` javascript
  function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
  ```
  其返回值也是一个函数, 再调用这函数, 并传入一个需要包装的 Presentational Component 
  就可以动态产生一个 Containter Component. 用法如下:
  ``` javascript
  connect(mapStateToProps, mapDispatchToProps)(Counter)
  ```
  这种操作在静态类型的语言中实在是太难..别说是 Java, 我估计 Dart 都做不到.
  
所以这次的模仿只能透过现象看本质. 看能不能模仿一下设计思路了.

## Provider
前面提到 Provider 其实就是利用 context 这个特性. 正好 Android 当中的所有 View 也都有 context 
这个属性, 通常就是对应的 Activity, 所以这里我们只需要让 Activity 实现一个接口, 让 Container  
Component 对应的东西可以取得 Store 就好. 

实现很简单, 首先定义一个接口.

``` kotlin
interface StoreProvider<StoreType: Store> {
    val store: Store
}
```

然后让 MainActivity 实现这个接口.

``` kotlin
class MainActivity : AppCompatActivity(), StoreProvider<CounterStore> {
    override fun getStore() = store
    
    // ...
}
```

那么在 View 需要 store 的时候就可以通过 context 来获得
``` kotlin
val context = getContext()
if (context is StoreProvider<CounterStore>) {
    val store = context.store
    //...
}
```

但是呢, 前面又说到 Containter Component 如果用 View 来模仿的话, 性能会有问题. Presentational
Component 又不会有读取 store 的操作, 这个方案的价值很低.


