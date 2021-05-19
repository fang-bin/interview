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