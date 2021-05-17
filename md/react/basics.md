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

**适用范围**

* 半受控组件(例如:input 有传参以传参为主，没有传参以内部state为主)
* 带有中间状态的组件（一些组件需要在用户输入时有一个中间状态，当触发某个操作时再把中间结果提交给上层）
* 记忆

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

**如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。如果只想处理更改，请确保进行当前值与变更值的比较。**

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

Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。

* Hook 使你在无需修改组件结构的情况下复用状态逻辑。 (在组件之间复用状态逻辑很难)
* Hook 将组件中相互关联的部分拆分成更小的函数，比如设置订阅或请求数据;(复杂组件变得难以理解)
* Hook 使你在非 class 的情况下可以使用更多的 React 特性。(难以理解的 class)

使用规则:

* 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。（还有一个地方可以调用 Hook —— 就是自定义的 Hook 中）

#### `useState(initialValue)`

`const [state, setState] = useState(initialValue)`

useState 会返回一对值：当前状态和一个让你更新它的函数。

类似 class 组件的 this.setState，但是它不会把新的 state 和旧的 state 进行合并(更新 state 变量总是替换它而不是合并它)。

setState 函数用于更新 state。它接收一个新的 state 值并将组件的一次重新渲染加入队列。

##### 函数式更新

如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。

##### 惰性初始state

initialState 参数只会在组件的初始渲染中起作用，后续渲染时会被忽略。如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只在初始渲染时被调用

```javascript
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

##### 跳过 state 更新

调用 State Hook 的更新函数并传入当前的 state 时，React 将跳过子组件的渲染及 effect 的执行。（React 使用 Object.is 比较算法 来比较 state。）

#### `useReducer(reducer, initialArg, init)`

```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。

在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 useReducer 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 dispatch 而不是回调函数 。

例：

```javascript
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

useReducer 更适合拿来做简单场景下的数据流。useReducer 是阉割版的 redux，只缺了一个状态共享能力，用 hooks 的 useContext 刚刚好。

useReducer 的好处之一便是， dispatch 不会随着 re-render 而重新分配没存中的位置，在作为 props 传入到 child component 中时也可以不用担心沒有 useMemo 而造成 re-render 的问题。

#### `useReducer` 和 `useState` 的关系

功能性上， `useState` 算是 `useReducer` 的一个子集，在 React 的源码中， `useState` 也是通过 `useReducer` 实现的。

#### `useEffect(didUpdate)`

useEffect 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。

如果 effect 返回一个函数，React 将会在执行清除操作时调用它。

如果某些特定值在两次重渲染之间没有发生变化，你可以通知 React 跳过对 effect 的调用，只要传递数组作为 useEffect 的第二个可选参数即可。

#### `useLayoutEffect`

其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

尽可能使用标准的 useEffect 以避免阻塞视觉更新。

#### `useEffect和useLayoutEffect的区别`

useEffect 是异步执行的，并不会阻塞当次 react 的 commit，执行时机是浏览器完成渲染之后（即 react 的 commit 阶段之后），如果在 useEffect 中进行 setState，当前渲染的出来的结果可能和 state 并不一致，之后会触发重新渲染，造成闪烁。

useLayoutEffect 是同步执行的，会阻塞当次的 react 的 commit，DOM变更之后立即调用，执行时机是浏览器真正把内容渲染到界面之前，会等待其执行完成之后，再进行渲染。

使用建议：

* 优先使用 useEffect，因为它是异步的，不会阻塞渲染。
* 会影响渲染的操作可以看情况放在 useLayoutEffect 中，避免出现闪烁。
* useLayoutEffect 和 componentDidMount、componentDidUpdate、componentWillUnmount 等价(react源码)。

#### `useContext(Context)`

```javascript
const value = useContext(MyContext);
```

接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 \<MyContext.Provider\> 的 value prop 决定。

当组件上层最近的 \<MyContext.Provider\> 更新时，该 Hook 会触发重渲染，并使用最新传递给 MyContext provider 的 context value 值。即使祖先使用 React.memo 或 shouldComponentUpdate，也会在组件本身使用 useContext 时重新渲染。

#### `useCallback`

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

返回一个 memoized 回调函数。

把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

`useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`。

#### `useMemo`

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

返回一个 memoized 值。

把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

你可以把 useMemo 作为性能优化的手段，但不要把它当成语义上的保证。

#### `React.memo`

`React.memo` 为高阶组件

如果你的组件在相同 props 的情况下渲染相同的结果，那么你可以通过将其包装在 React.memo 中调用，以此通过记忆组件渲染结果的方式来提高组件的性能表现。这意味着在这种情况下，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果。

React.memo 仅检查 props 变更。如果函数组件被 React.memo 包裹，且其实现中拥有 useState，useReducer 或 useContext 的 Hook，当 context 发生变化时，它仍会重新渲染。

默认情况下其只会对复杂对象做浅层对比，如果你想要控制对比过程，那么请将自定义的比较函数通过第二个参数传入来实现。

其相当于class Component中的 shouldComponentUpdate 和 PureComponent的组合，其中`React.memo`的第二参数相当于shouldComponentUpdate，不过返回值和shouldComponentUpdate正好相反。

#### `useMemo` 和 `useCallback` 的区别

* `useMemo` 缓存值，`useCallback` 缓存函数.
* `useMemo`更多的用来优化函数自身，`useCallback`更多的用来优化子组件

#### 什么时候使用 `useMemo` 和 `useCallback`

* 引用相等
* 昂贵的计算

另插一句觉得要理解对react的性能优化要理解几个重要的概念：不可变数据、引用相等等

#### `useMemo` 和 `React.memo` 的区别

* 两者都用来优化函数组件
* `useMemo`通过更细粒度的缓存值来减少组件自身更新，`useCallback`作用于整个函数组件，相当于class Component中的 `shouldComponentUpdate` 和 `PureComponent`的组合。

#### `useRef`

```javascript
const refContainer = useRef(initialValue);
```

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

useRef() 比 ref 属性更有用。它可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。

这是因为它创建的是一个普通 Javascript 对象。而 useRef() 和自建一个 {current: ...} 对象的唯一区别是，useRef 会在每次渲染时返回同一个 ref 对象。

当 ref 对象内容发生变化时，useRef 并不会通知你。变更 .current 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref 来实现。

下面抽离出来一个自定义Hook，当ref绑定元素挂载或卸载的时候，将元素的信息同步并通知。

```javascript
function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}
```

#### `useImperativeHandle`

```javascript
useImperativeHandle(ref, createHandle, [deps]);
```

useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。useImperativeHandle 应当与 forwardRef 一起使用

```javascript
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

#### `useDebugValue`

`useDebugValue(value)`

useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签。

#### 自定义Hook

自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。 

自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。

* 自定义 Hook 必须以 “use” 开头，不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 Hook 的规则（只能在函数最外层调用Hook，除了自定义Hook）。
* 在两个组件中使用相同的 Hook 并不会共享 state ，每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。

**监听深度依赖的值变化**

```javascript
import { isEqual } from 'lodash';
function useDeepCompareEffect(fn, deps){
    const comparisons = useRef(0);
    const prevDeps = useRef(deps);
    if(!isEqual(prevDeps.current, deps)){
    	comparisons.current++;
    }
    prevDeps.current = deps;
    return useEffect(fn, [comparisons.current]);
}
```

**setState回调**

```javascript
function useCallbackState<T>(init: T): [T, (value: T, cb?: (d: T) => void) => void] {
  const [state, setState] = useState<T>(init);
  const refState = useRef<undefined|((d: T) => void)>(undefined);
  const setCallbackState = (value: T, cb?: (d: T) => void): void => {
    setState(prev => {
      refState.current = cb;
      return typeof value === 'function' ? value(prev) : value;
    });
  }
  useEffect(() => {
    if (refState.current) refState.current(state);
  }, [state]);
  return [state, setCallbackState];
}
```

**useDebounce**

```javascript
function useDebounce<T>(value: T, dealy: number): T {
  const [debounceValue, setDebounceValue] = useState<T>(value);
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebounceValue(value);
    }, dealy);
    return () => {
      clearTimeout(timer);
    };
  }, [value, dealy]);
  return debounceValue;
}
```

**useThrottle**
```javascript
function useThrottle<T>(value: T, limit: number): T {
  const [throttleValue, setThrottleValue] = useState<T>(value);
  const lastTime = useRef(Date.now());
  useEffect(() => {
    const timer = setTimeout(() => {
      setThrottleValue(value);
      lastTime.current = Date.now();
    }, limit - (Date.now() - lastTime.current));
    return () => {
      clearTimeout(timer);
    }
  }, [value, limit]);
  return throttleValue;
}
```

**useForceUpdate**
```javascript
// forceUpdate
function useForceUpdate() {
  const [, forceUpdate] = useReducer(c => c + 1, 0);
  return forceUpdate;
}
```

**useLegacyState**
```javascript
type Partial<T> = {
  [P in keyof T]?: T[P];
};
function useLegacyState <T>(defaultState: T): [T, (nextState: Partial<T>) => void] {
  let [state, setState] = useState<T>(defaultState);

  const setLegacyState = (nextState: Partial<T>): void => {
    let newState = { ...state, ...nextState };
    setState(newState);
  };

  return [state, setLegacyState];
};
```

**useReducer useContext createContext实现一个小型的redux**
```javascript
/ actionType.js
const actionType = {
  INSREMENT: 'INSREMENT',
  DECREMENT: 'DECREMENT',
  RESET: 'RESET'
}
export default actionType

// actions.js
import actionType from './actionType'
const add = (num) => ({
    type: actionType.INSREMENT,
    payload: num
})

const dec = (num) => ({
    type: actionType.DECREMENT,
    payload: num
})

const getList = (data) => ({
    type: actionType.GETLIST,
    payload: data
})
export {
    add,
    dec,
    getList
}

// reducer.js
function init(initialCount) {
  return {
    count: initialCount,
    total: 10,
    user: {},
    article: []
  }
}

function reducer(state, action) {
  switch (action.type) {
    case actionType.INSREMENT:
      return {count: state.count + action.payload};
    case actionType.DECREMENT:
      return {count: state.count - action.payload};
    case actionType.RESET:
      return init(action.payload);
    default:
      throw new Error();
  }
}

export { init, reducer }

// redux.js
import React, { useReducer, useContext, createContext } from 'react'
import { init, reducer } from './reducer'

const Context = createContext()
const Provider = (props) => {
  const [state, dispatch] = useReducer(reducer, props.initialState || 0, init);
    return (
      <Context.Provider value={{state, dispatch}}>
        { props.children }
      </Context.Provider>
    )
}

export { Context, Provider }
```


#### React中使用单链表的形式来实现Hooks，而不是数组

数组： 内存中连续存放。访问快，删除添加慢。(访问O(1), 删除添加O(n)，能顺序存取或随机存取)

链表： 内存中不是顺序存储。删除添加快，访问慢。（访问O(n)，删除添加O(1)，只能顺序存取）




