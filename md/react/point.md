1. refs 将不会透传下去。这是因为 ref 不是 prop 属性。就像 key 一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件。

2. 不可变数据的力量
3. 深入源码剖析componentWillXXX为什么UNSAFE 
    从v16.3.0开始如下三个生命周期钩子被标记为UNSAFE。
    * componentWillMount
    * componentWillRecieveProps
    * componentWillUpdate

    究其原因，有如下两点
    * 这三个钩子经常被错误使用，并且现在出现了更好的替代方案（这里指新增的getDerivedStateFromProps与getSnapshotBeforeUpdate）。
    * React从Legacy模式迁移到Concurrent模式后，这些钩子的表现会和之前不一致。
4. React.Component 和 React.PureComponent 、React.memo 的区别
    * React.Component 是没有做任何渲染优化的，但凡调用this.setState 就会执行render的刷新操作。
    * React.PureComponent 是继承自Component，并且对重写了shouldComponentUpdate周期函数，对 state 和 props 做了浅层比较，当state 和 props 均没有改变时候，不会render，仅可以用在ClassComponent中
    * React.memo 功能同React.PureComponent，React.memo 为高阶组件，只适用于函数组件，而不适用 class 组件。
    **React.memo 仅检查 props 变更。如果函数组件被 React.memo 包裹，且其实现中拥有 useState 或 useContext 的 Hook，当 context 发生变化时，它仍会重新渲染。**
    **React.memo第二个函数与 class 组件中 shouldComponentUpdate() 方法不同的是，如果 props 相等，areEqual 会返回 true；如果 props 不相等，则返回 false。这与 shouldComponentUpdate 方法的返回值相反。**
5. useEffect 和 useLayoutEffect 的区别
    与 componentDidMount、componentDidUpdate 不同的是，在浏览器完成布局与绘制之后，传给 useEffect 的函数会延迟调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因此不应在函数中执行阻塞浏览器更新屏幕的操作。
    虽然 useEffect 会在浏览器绘制后延迟执行，但会保证在任何新的渲染前执行。React 将在组件更新前刷新上一轮渲染的 effect。
    其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。
    尽可能使用标准的 useEffect 以避免阻塞视觉更新。