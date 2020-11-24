#### 组件封装原则

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

#### 10. React组件的生命周期

* `ComponentWillMount`
* `render`
* `ComponentDidMount`
* `ComponentWillReceiveProps`
* `ShouldComponentUpdate`
* `ComponentWillUpdate` 
* `ComponentDidUpdate`
* `ComponentWillUnmount`

###### ComponentWillMount()

在渲染前调用

因为componentWillMount是在render之前执行，所以在这个方法中setState不会发生重新渲染(re-render);

在componenetWillMount中 setState，新的state 不会引起新的更新行为，但是新的 state内容会被带到 render 中体现。

这是服务端渲染(server render)中唯一调用的钩子(hook);

###### render()
严格来说，render也算是React生命周期中的一部分，一般只在两个时机调用:

1. 在componentWillMount()方法之后
2. 在ComponentWillUpdate()方法之后

###### ComponentDidMount()
在第一次渲染（render）后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

这里可以对DOM进行操作，这个函数之后ref变成实际的DOM，同时也**可以使用setState()方法触发重新渲染(re-render)**;

**ref的current属性会在ComponentDidMount触发前传入DOM元素，并在ComponentDidUpdate触发前更新。在组件卸载时传入null值**

###### componentWillReceiveProps(nextProps)

在已挂载组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。

如果需要在props发生变化(或者说新传入的props)来更新state，可能需要比较this.props和nextProps, 然后使用this.setState()方法来改变this.state;

注意：如果只是调用this.setState()而不是从外部传入props, 那么不会触发componentWillReceiveProps(nextProps)函数；这就意味着: this.setState()方法不会触发componentWillReceiveProps(), props的改变或者props没有改变才会触发这个方法;

###### ShouldComponentUpdate(nextProps, nextState)

在接收到新props或state后触发，用来确定是否发生重新渲染，默认情况返回true，表示会发生重新渲染，返回false，则会跳过render(后面的涉及更新重新渲染的生命周期都不会触发)，所以也就不会发生Virtual DOM diff。

这个方法在首次渲染时或者forceUpdate()时不会触发;

现在一些情况，PureComponent已经替代了ShouldComponentUpdate的使用场景。

###### ComponentWillUpdate(nextProps, nextState)

在props或state发生改变或者shouldComponentUpdate(nextProps, nextState)触发后, 在render()之前（首次初始化并不会触发）。

千万不要在这个函数中调用this.setState()方法;

在componentWillUpdate中setState，它的下一步本来就是 render，新的 state 内容不会被带到 render 中。如果在componentWillUpdate确实设置了新的不同的 state，则会引起循环的更新行为(会造成React调用栈溢出，渲染表现不正常)，如果只是调用了 setState，但是 state 内容并无变化，则不会引起循环的渲染更新行为。

当然如果非要在componentWillUpdate中执行setState，有限制判断条件的情况下，可以通过setTimeout函数来执行，这样就不会造成React的栈溢出了。

###### ComponentDidUpdate(prevProps, prevState)

 在组件完成更新后(componentWillUpdate(nextProps, nextState)后
)立即调用。在初始化时不会被调用。

###### ComponentWillUnmount

在组件从 DOM 中移除并销毁之前立刻被调用。

这个方法可以让你处理一些必要的清理操作，比如无效的timers、interval，或者取消网络请求，或者清理任何在componentDidMount()中创建的DOM元素(elements);


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
