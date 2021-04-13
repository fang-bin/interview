#### 1. render内使用箭头函数或者bind函数，可能会造成子组件额外的重新渲染。

```javascript
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={() => this.handleClick()}>
      // onClick={this.handleClick.bind(this)}实际上也会造成性能浪费
        Click me
      </button>
    );
  }
}
```

每次渲染 LoggingButton 时都会创建不同的回调函数。在大多数情况下，这没什么问题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。我们通常建议在构造器中绑定或使用 class fields 语法来避免这类性能问题。

#### 2. React 渲染

##### 为什么有必要设置key? 要讲一下React的Diffing算法

React 的 render() 方法，会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 render() 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何有效率的更新 UI 以保证当前 UI 与最新的树保持同步。

React 在以下两个假设的基础之上提出了一套 O(n) 的启发式算法来生成将一棵树转换成另一棵树的最小操作数。
1. 两个不同类型的元素会产生出不同的树；
2. 开发者可以通过 key prop 来暗示哪些子元素在不同的渲染下能保持稳定；

**Diffing 算法**
当对比两颗树时，React 首先比较两棵树的根节点。不同类型的根节点元素会有不同的形态。

1. 对比不同类型的元素
  当根节点为不同类型的元素时，React 会拆卸原有的树并且建立起新的树。
  当拆卸一棵树时，对应的 DOM 节点也会被销毁。组件实例将执行 componentWillUnmount() 方法。当建立一棵新的树时，对应的 DOM 节点会被创建以及插入到 DOM 中。组件实例将执行 componentWillMount() 方法，紧接着 componentDidMount() 方法。所有跟之前的树所关联的 state 也会被销毁。
  在根节点以下的组件也会被卸载，它们的状态会被销毁。
2. 对比同一类型的元素
  当对比两个相同类型的 React 元素时，React 会保留 DOM 节点，仅比对及更新有改变的属性。
  在处理完当前节点之后，React 继续对子节点进行递归。
3. 对比同类型的组件元素
  当一个组件更新时，组件实例保持不变，这样 state 在跨越不同的渲染时保持一致。React 将更新该组件实例的 props 以跟最新的元素保持一致，并且调用该实例的 componentWillReceiveProps() 和 componentWillUpdate() 方法。
4. 对子节点进行递归
  默认条件下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。
  在子元素列表末尾新增元素时，更新开销比较小。
  如果只是简单的将新增元素插入到表头，那么更新开销会比较大。React会重建每一个子元素 。这种情况会带来性能问题。
  **为了解决以上问题，React 支持 key 属性。当子元素拥有 key(这个 key 不需要全局唯一，但在列表中需要保持唯一。) 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。**

如果没有对渲染项设置key,会默认设置其index为key,不过如果列表项目的顺序可能会变化，不建议使用索引来用作 key 值。因为这样做会导致性能变差（**key 来决定是否更新以及复用**），还可能引起组件状态的问题。原因如下：

Key是React用来识别Dom元素的唯一东西，如果变更元素后的key和之前某元素一样，则React会认为该元素就是之前的元素，这种情况下，它导致非受控组件的 state（比如输入框）可能相互篡改导致无法预期的变动。

React 可以在每个 action 之后对整个应用进行重新渲染，得到的最终结果也会是一样的。在此情境下，重新渲染表示在所有组件内调用 render 方法，这不代表 React 会卸载或装载它们。React 只会基于以上提到的规则来决定如何进行差异的合并。

 React 依赖探索的算法，因此当以下假设没有得到满足，性能会有所损耗。

1. 该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。

2. Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 Math.random() 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。

#### 3. 受控组件和非受控组件

在React中，所谓受控组件和非受控组件都是针对表单而言的。

表单元素依赖于状态，表单元素需要默认值实时映射到状态的时候，就是受控组件（和双向绑定类似），非受控组件即不受状态的控制，获取数据就是相当于操作DOM。

受控组件的优点是，我们可以实时对输入的内容进行校验，表单的修改也可以实时映射到状态值上。 缺点是我们要为受控组件的每个状态更新都编写数据处理函数，会增加代码量和开发难度。

非受控组件的优点是很容易和第三方组件结合，真实数据都存储在DOM节点中，可以同时集成React和非React代码，快速编写，减少代码量。

同时，也有一些始终是非受控组件的表单元素，比如`<input type="file"/>`

**获取非受控组件的状态，可以使用ref**

#### 4. Context

Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据。请谨慎使用，因为这会使得组件的复用性变差。

**如果你只是想避免层层传递一些属性，组件组合（component composition）有时候是一个比 context 更好的解决方案。**

Context 能让你将这些数据向组件树下所有的组件进行“广播”，所有的组件都能访问到这些数据，也能访问到后续的数据更新。

##### 5. 错误边界

过去，组件内的 JavaScript 错误会导致 React 的内部状态被破坏，并且在下一次渲染时 产生 可能无法追踪的 错误。

错误边界是一种 React 组件，这种组件可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI，而不是渲染那些崩溃了的子组件树。错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

错误边界无法捕获以下场景中产生的错误：
1. 事件处理
2. 异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
3. 服务端渲染
4. 它自身抛出来的错误（并非它的子组件）

**当抛出错误后，请使用 static getDerivedStateFromError() 渲染备用 UI ，使用 componentDidCatch() 打印错误信息。**

错误边界仅可以捕获其子组件的错误，它无法捕获其自身的错误。

#### 6. 高阶组件（HOC）

在业务开发的过程中，如果完全不同的组件有相似的功能，这就会产生横切关注点（cross-cutting concerns）问题。其根本还是在不同的组件当中复用一些相同的组件逻辑。

高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。

**高阶组件是参数为组件，返回值为新组件的函数。**

HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将组件包装在容器组件中来组成新组件。HOC 是纯函数，没有副作用。

###### 为什么不推荐使用mixins

mixin函数，其实质就是将mixins中的方法遍历赋值给newObj.prototype，从而实现mixin返回的函数创建的对象都有mixins中的方法，也就是把额外的功能都混入进去。

React 在反对 mixins 的这方面已经说得很清楚了，里面有句中肯话意思是 mixins 本身是没有问题的。

Mixins 的问题在于他太过于灵活，太依赖使用者，mixin这种引入大量不可控因素的方式，导致新人上手代码时对其如何工作越发不理解(incomprehensible)。

其次，ES6 本身是不包含任何 mixin 支持。因此，当你在 React 中使用 ES6 class 时，将不支持 mixins 。

高阶组件的约定:

* 将不相关的 props 传递给被包裹的组件
* 最大化可组合性
* 包装显示名称以便轻松调试
  比如高阶组件名为 withSubscription，并且被包装组件的显示名称为 CommentList，显示名称应该为 WithSubscription(CommentList)

注意:
* 不要在 render 方法中使用 HOC
  不应在组件的 render 方法中对一个组件应用 HOC
  React 的 diff 算法（称为协调）使用组件标识来确定它是应该更新现有子树还是将其丢弃并挂载新子树。 如果从 render 返回的组件与前一个渲染中的组件相同（===），则 React 通过将子树与新子树进行区分来递归更新子树。 如果它们不相等，则完全卸载前一个子树。
  这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  这不仅仅是性能问题 - 重新挂载组件会导致该组件及其所有子组件的状态丢失。
* 务必复制静态方法
* Refs 不会被传递
  虽然高阶组件的约定是将所有 props 传递给被包装组件，但这对于 refs 并不适用。那是因为 ref 实际上并不是一个 prop - 就像 key 一样，它是由 React 专门处理的。如果将 ref 添加到 HOC 的返回组件中，则 ref 引用指向容器组件，而不是被包装组件。
  可通过使用 React.forwardRef API来解决

#### 7. 深入理解JSX

JSX 仅仅只是 React.createElement(component, props, ...children) 函数的语法糖。

```javascript
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>

// 会编译为

React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

#### 8. ShouldComponentUpdate && PureComponent

React的渲染是根据触发render之后，比对Virtual DOM和之前是否完全一样来触发的渲染。

virtual DOM diff 也是需要执行时间的。虽然说速度很快，但再快也比不上完全不呼叫来的快。

一些组件明确不该有任何变化的情况，可以在 shouldComponentUpdate 里直接 return false，或者加上一个比较（深或浅），倾向于 shallowEqual ，因为 deepEqual 在层级深的时候也是很吃资源的。

只有当组件 props.color 或者 state.count 的值改变才需要更新时，你可以使用 shouldComponentUpdate 采用类似“浅比较”的模式来检查 props 和 state 中所有的字段，以此来决定是否组件需要更新。

PureComponent 也就是改写了 shouldComponentUpdate ，加上了浅比较，可以减少不必要的 render 操作的次数，从而提高性能，当组件更新时，如果组件的 props 和 state 都没发生改变，render 方法就不会触发，省去 Virtual DOM 的生成和比对过程，达到提升性能的目的。

```javascript
// PureComponent
if (type.prototype && type.prototype.isPureReactComponent) {
 return (
   !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
 );
}

const hasOwn = Object.prototype.hasOwnProperty;
function is(x, y) {
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y;
  } else {
    return x !== x && y !== y;
  }
}

function shallowEqual(objA, objB) {
  if (is(objA, objB)) return true

  if (typeof objA !== 'object' || objA === null ||
      typeof objB !== 'object' || objB === null) {
    return false
  }

  const keysA = Object.keys(objA)
  const keysB = Object.keys(objB)

  if (keysA.length !== keysB.length) return false

  for (let i = 0; i < keysA.length; i++) {
    if (!hasOwn.call(objB, keysA[i]) ||
        !is(objA[keysA[i]], objB[keysA[i]])) {
      return false
    }
  }

  return true
}
```

is 方法（Object.is） 比 === 修复了 NaN 和 +-0 的情况，可以看出也仅仅是做了一层基本数据类型的比较。所以一些不合理的写法会导致 shadowEqual 结果不正确，比如：

```javascript
const newObj = this.state.obj
newObj.id = 2;
this.setState({
  obj: newObj
});
```

由于 newObj 与 obj 地址相同，所以 shadowEqual 比较结果是 true。可以使用clone（Object.assign、lodash.merge、JSON.parse/stringify）:

```javascript
const newObj = Object.assign(this.state.obj)
newObj.id = 2;
this.setState({
 obj: newObj
});
```

如果 PureComponent 里有 shouldComponentUpdate 函数的话，直接使用 shouldComponentUpdate 的结果作为是否更新的依据，没有shouldComponentUpdate 函数的话，才会去判断是不是 PureComponent ，是的话再去做 shallowEqual浅比较。

**注意**

如果state或者props每次都会变，那么PureComponent效率还不如Component，因为毕竟shadowEqual也是需要执行时间的。


#### 9. Portals

Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

`ReactDOM.createPortal(child, container)`

## 10. React组件的生命周期

最新的react生命周期

![react生命周期](https://github.com/fang-bin/interview/blob/master/image/react-lifecycle.jpg)

### 挂载

1. `constructor(props)`
2. `static getDerivedStateFromProps(props, state)`
3. `UNSAFE_componentWillMount()`
4. `render()`
5. `componentDidMount()`

#### `constructor(props)`

如果不初始化 state 或不进行方法绑定，则不需要为 React 组件实现构造函数。

在 React 中，构造函数仅用于以下两种情况：

* 通过给 this.state 赋值对象来初始化内部 state。
* 为事件处理函数绑定实例

#### `static getDerivedStateFromProps(props, state)`

getDerivedStateFromProps 会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。

它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。

此方法无权访问组件实例。

不管原因是什么，都会在每次渲染前触发此方法。

#### `UNSAFE_componentWillMount()`

在挂载之前被调用。它在 render() 之前调用，因此在此方法中同步调用 setState() 不会触发额外渲染。

此方法是服务端渲染唯一会调用的生命周期函数。

#### `render()`

render() 方法是 class 组件中唯一必须实现的方法。

当 render 被调用时，它会检查 this.props 和 this.state 的变化并返回以下类型之一：

* React 元素 (DOM节点或自定义组件，均为React元素)
* 数组或 fragments (返回多个元素)
* Portals (渲染子节点到不同的 DOM 子树中)
* 字符串或数值类型 (在 DOM 中会被渲染为文本节点)
* 布尔类型或 null (什么都不渲染)

render() 函数应该为纯函数，这意味着在不修改组件 state 的情况下，每次调用时都返回相同的结果，并且它不会直接与浏览器交互。

**如果 shouldComponentUpdate() 返回 false，则不会调用 render()**

#### `componentDidMount()`

会在组件挂载后（插入 DOM 树中）立即调用。

依赖于 DOM 节点的初始化应该放在这里。如需通过网络请求获取数据，此处是实例化请求的好地方。

可以在 componentDidMount() 里直接调用 setState()。它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前。如此保证了即使在 render() 两次调用的情况下，用户也不会看到中间状态。**请谨慎使用该模式**，因为它会导致性能问题。通常，你应该在 constructor() 中初始化 state。如果你的渲染依赖于 DOM 节点的大小或位置，比如实现 modals 和 tooltips 等情况下，你可以使用此方式处理

### 更新

1. `static getDerivedStateFromProps(props, state)`
2. `UNSAFE_componentWillReceiveProps(nextProps)`
3. `shouldComponentUpdate(nextProps, nextState)`
4. `UNSAFE_componentWillUpdate(nextProps, nextState)`
5. `render()`
6. `getSnapshotBeforeUpdate(preProps, preState)`
7. `componentDidUpdate(preProps, preState, snapshot)`

#### `UNSAFE_componentWillReceiveProps(nextProps)`

UNSAFE_componentWillReceiveProps() 会在已挂载的组件接收新的 props 之前被调用。

如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。如果只想处理更改，请确保进行当前值与变更值的比较。

调用 this.setState() 通常不会触发 UNSAFE_componentWillReceiveProps()。

#### `shouldComponentUpdate(nextProps, nextState)`

当 props 或 state 发生变化时，shouldComponentUpdate() 会在渲染执行之前被调用。返回值默认为 true。首次渲染或使用 forceUpdate() 时不会调用该方法。

此方法仅作为性能优化的方式而存在。可以考虑使用内置的 PureComponent 组件。PureComponent 会对 props 和 state 进行浅层比较，并减少了跳过必要更新的可能性。

不建议在 shouldComponentUpdate() 中进行深层比较或使用 JSON.stringify()。这样非常影响效率，且会损害性能。

目前，如果 shouldComponentUpdate() 返回 false，则不会调用 UNSAFE_componentWillUpdate()，render() 和 componentDidUpdate()。后续版本，React 可能会将 shouldComponentUpdate 视为提示而不是严格的指令，并且，当返回 false 时，仍可能导致组件重新渲染。

#### `UNSAFE_componentWillUpdate(nextProps, nextState)`

当组件收到新的 props 或 state 时，会在渲染之前调用 UNSAFE_componentWillUpdate()。使用此作为在更新发生之前执行准备更新的机会。初始渲染不会调用此方法。

**不能此方法中调用 this.setState()**，在 UNSAFE_componentWillUpdate() 返回之前，你也不应该执行任何其他操作（例如，dispatch Redux 的 action）触发对 React 组件的更新。

此方法可以替换为 componentDidUpdate()。如果你在此方法中读取 DOM 信息（例如，为了保存滚动位置），则可以将此逻辑移至 getSnapshotBeforeUpdate() 中。

#### `getSnapshotBeforeUpdate(preProps, preState)`

getSnapshotBeforeUpdate() 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 componentDidUpdate()。

#### `componentDidUpdate(preProps, preState, snapshot)`

componentDidUpdate() 会在更新后会被立即调用。首次渲染不会执行此方法。

可以在 componentDidUpdate() 中直接调用 setState()，但请注意它必须被包裹在一个条件语句里，正如上述的例子那样进行处理，否则会导致死循环。

### 卸载

`componentWillUnmount()`

#### `componentWillUnmount()`

componentWillUnmount() 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 componentDidMount() 中创建的订阅等。

componentWillUnmount() 中不应调用 setState()，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。

### 错误处理

1. `static getDerivedStateFromError(error)`
2. `componentDidCatch(error, info)`

#### `static getDerviedStateFromError(error)`

此生命周期会在后代组件抛出错误后被调用。 它将抛出的错误作为参数，并返回一个值以更新 state，并执行渲染。（它是在渲染阶段调用的，所以不允许出现副作用。）

#### `componentDidCatch(error, info)`

此生命周期在后代组件抛出错误后被调用。 它接收两个参数：

* error —— 抛出的错误。
* info —— 带有 componentStack key 的对象，其中包含有关组件引发错误的栈信息。

componentDidCatch() 会在“提交”阶段被调用，因此允许执行副作用。 它应该用于记录错误之类的情况

### 触发更新

* new props
* `setState(updater[, callback])`
* `forceUpdate(callback)`

#### `setState(updater[, callback])`

setState() 将对组件 state 的更改排入队列，并通知 React 需要使用更新后的 state 重新渲染此组件及其子组件。这是用于更新用户界面以响应事件处理器和处理服务器数据的主要方式。

将 setState() 视为请求而不是立即更新组件的命令。为了更好的感知性能，React 会延迟调用它，然后通过一次传递更新多个组件。React 并不会保证 state 的变更会立即生效。

setState() 并不总是立即更新组件。它会批量推迟更新。这使得在调用 setState() 后立即读取 this.state 成为了隐患。为了消除隐患，请使用 componentDidUpdate 或者 setState 的回调函数（setState(updater, callback)），这两种方式都可以保证在应用更新后触发。如需基于之前的 state 来设置当前的 state，请阅读下述关于参数 updater 的内容。

除非 shouldComponentUpdate() 返回 false，否则 setState() 将始终执行重新渲染操作。如果可变对象被使用，且无法在 shouldComponentUpdate() 中实现条件渲染，那么仅在新旧状态不一时调用 setState()可以避免不必要的重新渲染。

参数一为带有形式参数的 updater 函数： `(state, props) => stateChange`

#### `forceUpdate(callback)`

默认情况下，当组件的 state 或 props 发生变化时，组件将重新渲染。如果 render() 方法依赖于其他数据，则可以调用 forceUpdate() 强制让组件重新渲染。

调用 forceUpdate() 将致使组件调用 render() 方法，此操作会跳过该组件的 shouldComponentUpdate()。但其子组件会触发正常的生命周期方法，包括 shouldComponentUpdate() 方法。如果标记发生变化，React 仍将只更新 DOM。


#### 11. Ref

ref的current属性会在ComponentDidMount触发前传入DOM元素，并在ComponentDidUpdate触发前更新。在组件卸载时传入null值

不能在函数组件上使用 ref 属性，因为它们没有实例

如果要在函数组件中使用 ref，你可以使用 forwardRef（可与 useImperativeHandle 结合使用），或者可以将该组件转化为 class 组件。

```javascript
function CustomTextInput(props) {
  // 这里必须声明 textInput，这样 ref 才可以引用它
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

#### 12 Render Props

具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。

```javascript
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
使用 render prop 的库有 React Router、Downshift 以及 Formik。

render prop 是一个用于告知组件需要渲染什么内容的函数 prop

**注意**:

如果你在 render 方法里创建函数，那么使用 render prop 会抵消使用 React.PureComponent 带来的优势。因为浅比较 props 的时候总会得到 false，并且在这种情况下每一个 render 对于 render prop 将会生成一个新的值。

## React Hooks
