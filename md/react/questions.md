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