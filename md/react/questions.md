## 1. useReducer 实现 useState 和 useState 实现 useReducer

```javascript
// useReducer实现useState
function useState<T>(initialState: T | (() => T)) {
  const initState: T =
    // eslint-disable-next-line @typescript-eslint/ban-ts-ignore
    // @ts-ignore
    typeof initialState === 'function' ? initialState() : initialState;

  const [state, dispatch] = React.useReducer(
    // eslint-disable-next-line no-shadow
    (state: T, action: T | ((prev: T) => T)) => {
      // eslint-disable-next-line @typescript-eslint/ban-ts-ignore
      // @ts-ignore
      return typeof action === 'function' ? action(state) : action;
    },
    initState,
  );

  return [
    state,
    (value: T | ((prev: T) => T)) => {
      if (value !== state) {
        dispatch(value);
      }
    },
  ];
}

// 即

function useState(initialState) {
  const reducer = (action, state) => {
    if (typeof action === 'function') {
      return action(state);
    }
    return action;
  };
  const [state, dispatch] = useReducer(reducer, initialState);

  // setState 和 dispatch 一样引用也不变的
  const setState = useCallback(action => {
    dispatch(action);
  }, []);

  return [state, setState];
}
```

```javascript
// useState实现useReducer
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, useCallback(dispatch, [])];
}
```

## 2. 当 useCallback 和 useEffect 组合使用时，由于 useCallback 的依赖项变化也会导致 useEffect 执行，这种隐式依赖会带来BUG或隐患。

```javascript
function useRefCallback<T extends (...args: any[]) => any>(callback: T) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  return useCallback((...args: any[]) => callbackRef.current(...args), []) as T;
}
```

## 3. HTML 和 React 的事件处理有什么不同？

* 在 HTML 中，事件名应该是全小写的，然而在 React 中事件名遵循小驼峰 格式。
  ```javascript
  // html
  <button onclick="activateLasers()"></button>
  // react
  <button onClick={activateLasers}>
  ```
* 在 HTML 中，你应该返回 false 来阻止默认行为，然后在 React 中你必须明确地调用 preventDefault()。
* 在 HTML 中，你调用函数时需要加上 ()，然后在 React 中你不应该在函数名后带上 ()。（比如前面示例中的 activateLasers 函数）

## 4. PureComponent 可能造成的问题

React.PureComponent 只进行浅比较，所以当 props 或者 state 某种程度是可变的话，浅比较会有遗漏，那你就不能使用它了。（例如数组）

应对数组这种情况，可以使用concat 或 扩展运算符等手段，来避免可变对象的产生。

应对深层次嵌套对象的话，可以使用Object.assign({}, target, ...source) 这种方式产生一个新对象，而不是修改老对象。

（参考React文档中，不可变数据的力量一章）

使用 render prop(箭头函数) 会抵消使用 React.PureComponent 带来的优势。因为浅比较 props 的时候总会得到 false，并且在这种情况下每一个 render 对于 render prop 将会生成一个新的值。

## 5.  Portal 事件冒泡

尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 React 树， 且与 DOM 树 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。

这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 React 树的祖先，即便这些元素并不是 DOM 树 中的祖先。