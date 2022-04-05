## React 18

主要内容：

1. 支持 concurrent 模式，新的API，如 startTranstion useDefferdValue
2. 改进已有属性，如：自动批量处理、SSR支持Suspense与Lazy等

#### 1. ReactDom渲染方式

从原先的 `ReactDom.render(Component, Container)` 变为 `ReactDom.createRoot(Container).render(Component)`

现在最新的是RC版本，引入ReactDom 还需要 `import ReactDom from 'react-dom/client'` 方式引入

**注意** React 18 中不再支持 `ReactDOM.render`。(会有Warning)

新的 `root.render()` 方法中也没有回调，因为在使用Suspense时，它通常不会有预期的结果。

最后，服务端渲染的方式由 hydrate 升级为 hydrateRoot 。（`hydrateRoot(Container, Component)`）

#### Concurrent 模式

React 16 就已经有 Concurrent 模式了，但是之前属于实验性质的。到 React 18，Concurrent 算正式使用。

其实，官方文档中说 React 18 中并不存在 Concurrent Mode，只有用于并发渲染的并发新特性。其可以帮助应用保持响应，并根据用户的设备性能和网速进行适当的调整。

* 一个更急迫的更新可以中断已经开始的渲染（高优先级任务先行，低优先级任务后行），衍生API：useDeferredValue
* React 可以在全部数据到达之前就在内存中开始渲染，然后跳过空白加载状态，衍生API：startTransition useTransition

Concurrent模式可以自定义更新任务优先级并且能够通知到React，React再来处理不同优先级的更新任务，当然，优先处理高优先级任务，并且低优先级任务可以中断。

为了支持并发特性，主要提供了三个新的API:

* `useDeferredValue`
* `startTransition`
* `useTransition`

##### startTransition

```javascript
import { startTransition } from "react";

// 紧急更新：
setInputValue(input);

// 标记回调函数内的更新为非紧急更新：
startTransition(() => {
  setSearchQuery(input);
});
```

简单来说，被 startTransition 包裹的 setState 触发的渲染被标记为不紧急渲染，意味着他们可以被其他紧急渲染所抢占。这种渲染优先级的调整手段可以帮助我们解决各种性能伪瓶颈，提升用户体验。

##### useDeferredValue

```javascript
function Page() {
  const [filters, mergeFilter] = useMergeState(defaultFilters);
  const deferedFilters = React.useDeferedValue(filters);
  return (
    <Fragment>
      <Filters filters={filters} >
      <List filters={deferedFilters} >
    </Fragment>
  );
}
```

useDeferedValue () 会将 List 组件的渲染变得更加平滑，深层次来看则是 defered value 引起的渲染则会被标记为不紧急渲染，会被 filters 引起的渲染进行抢占，进而达到用户快速输入搜索等场景下页面抖动或者卡顿问题。

##### useTransition

`const [isPending, startTransition] = useTransition()`;

多了一个通过startTransition触发的渲染，是否正在执行渲染的状态 isPending

#### 自动批处理

React 18通过默认做更多的批处理来增加开箱即用的性能改进。批处理是指 React 将多个状态更新分组到一个重新渲染中以获得更好的性能。在 React 18 之前，我们只在 React 事件处理程序内（React复合事件和生命周期钩子）进行批处理更新。默认情况下，React不会对promise、setTimeout、原生事件处理程序或任何其他事件中的更新进行批处理。

从React 18的createRoot开始，所有的更新都会被自动批处理，无论它们来自哪里。这意味着，超时、promises、本地事件处理程序或任何其他事件中的更新将以与React事件中的更新相同的方式进行批处理。

其实，也意味着从 React 18 开始，setState 事件就是一个完全的异步操作了。（这会减少渲染工作，从而提高应用程序的性能）

**注**： 如果要选择不使用批处理，可以使用 **`flushSync`**

```javascript
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```

#### Suspense



#### SSR

改造react-dom/server API，以完全支持服务器上的Suspense和流式SSR。作为这些变化的一部分，弃用旧的Node流媒体API，它不支持服务器上的增量Suspense流。

* ~~renderToNodeStream~~ 弃用
* renderToPipeableStream 新API

#### 其他

* React放弃了对IE浏览器的支持
