# react-routr v4的的组件被redux的connect包裹导致子路由没有渲染

最近在使用react-router v4的时候，通过redux的connect包裹组件，导致组件所在的子路由没有发生渲染。
花费了一番精力，总算找到问题所在。做个笔记，避免将来再踩坑。



要明白问题的缘由，需要先明白react-router v4的实现。

在Router组件里，会产生一个叫做router的context对象。当Router监听到路由发生改变的时候，会调用`setState`函数。

当调用`setState`函数时，会触发子路由，也就是`Route`组件的`componentWillReceiveProps`事件，从而通知`Route`是否挂载对应组件。

通常的结构是这样的：

```html
<Router>

  <MyComponent>

    <Route>

      <ChildComponent>

```



但用connect包裹的时候，connect会创建一个叫做Connect的组件进行一次包裹。这个时候结构变为这样：

```html
<Router>

  <connect>

    <Connect>

      <MyComponent>

        <Route>

          <ChildComponent>
```

原本一切都没问题，但`Connect`组件做了`pure render`处理。在这种情况下，当路由发生改变，`Router`的setState无法通知到`Route`的`componentWillReceiveProps`事件。因为这个时候`Connect`的props和state都没有发生改变，这样就中断了`Router`和`Route`的通信机制，从而导致子组件无法渲染。

解决办法是自己写个connect函数，不实现`pure render`。

那么`pure render`是否就无法使用了呢？当然不是。

我们可以声明一个基类，在基类的shouldComponentUpdate方法里加入对于context的判断。当然这需要每个组件都用自己写的`connect`包裹就是了