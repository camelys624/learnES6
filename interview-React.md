---
ebook:
  title: interview-React
  authors: yck
  editors: camel_y
---

# React

## React 生命周期分析

在 V16 版本中引入了 Fiber 机制。这个机制一定程度上的影响了部分生命周期的调用，并且也引入了新的 2 个 API 来解决问题。

在之前的版本中，如果你拥有一个很复杂的复合组件，然后改动了最上层组件的 `state`，那么调用栈可能会很长

![调用栈时间](./static/img/react1.png)

调用栈过长，再加上中间进行了复杂的操作，就可能导致长时间阻塞主线程，带来不好的用户体验。Fiber 就是为了解决该问题而生。

Fiber 本质上是一个虚拟的堆栈帧，新的调度器会按照优先级自由调度这些帧，从而将之前的同步渲染改成了异步渲染，在不影响体验的情况下分段计算更新。

![堆栈帧](./static/img/react2.png)

对于如何区别优先级，React 由自己的一套逻辑。对于动画这种实时性很高的东西，也就是 16ms 必须渲染一次保证不卡顿的情况下， React 会每 16ms (以内) 暂停以下更新，返回来继续渲染动画。

对于异步渲染，现在渲染有两个阶段： `reconciliation` 和 `commit`。前者过程是可以打断的，后者不能暂停，会一直更新界面直到完成。

#### Reconciliation 阶段

- `componentWillMount`
- `componentWillReceiveProps`
- `shouldComponentUpdate`
- `componentWillUpdate`

#### Commit 阶段

- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

因为 `reconciliation` 阶段是可以被打断的，所以 `reconciliation` 阶段会执行的生命周期函数就可能会出现调用多次的情况，从而引起 BUG。所以对于 `reconciliation` 阶段调用的几个函数，除了 `shouldComponentUpdate` 以外，其他都应该避免去使用，并且 V16 中也引入了新的 API 来解决这个问题。

`getDerivedStateFromProps` 用于替换 `componentWillReceiveProps` ,该函数会在初始化和 `update` 时被调用

```js
class ExampleComponent extends React.Component {
  // Initialize state in constructor,
  // Or with a property initializer.
  state = {}

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.someMirroredValue !== nextProps.someValue) {
      return {
        derivedData: computDerivedState(nextProps),
        someMirroredValue: nextProps.someValue
      }
    }

    // Return null to indicate no change to state
    return null
  }
}
```

`getSnapshotBeforeUpdate` 用于替换 `componentWillUpdate` ,该函数会在 `update` 后 DOM 更新前被调用，用于读取更新的 DOM 数据。

### V16 生命周期函数用法建议

```js
class ExampleComponent extends React.Component {
  // 用于初始化 state
  constructor() {}
  // 用于替换 'componentWillReceiveProps',该函数会在初始化和 'update' 时被调用
  // 因为该函数是静态函数，所以娶不到 'this'
  // 如果需要对比 'prevProps' 需要单独在 'state' 中维护
  static getDerivedStateFromProps(nextProps, prevState) {}
  // 判断是否需要更新组件，多用于组件性能优化
  shouldComponentUpdate(nextProps, nextState) {}
  // 组件挂载后调用
  // 可以在该函数中进行请求或者订阅
  componentDidMount() {}
  // 用于获取最新的 DOM 数据
  getSnapshotBeforeUpdate() {}
  // 组件即将销毁
  // 可以在此移除订阅，定时器等等
  componentWillUnmount() {}
  // 组件销毁厚调用
  componentDidUnMount() {}
  // 组件更新后调用
  componentDidUpdate() {}
  // 渲染组件函数
  render() {}
  // 以下函数不建议使用
  UNSAFE_componentWillMount() {}
  UNSAFE_componentWillUpdate(nextProps, nextState) {}
  UNSAFE_componentWillReceiveProps(nextProps) {}
}
```

## setState

`setState` 在 React 中是经常使用的一个 API，但是它存在一些问题，可能会导致犯错，核心原因就是因为这个 API 是异步的。

首先 `setState` 的调用并不会马上引起 `state` 的改变，并且如果你一次调用了多个 `setState` ,那么结果可能并不如你期待的一样。

```js
handle() {
  // 初始化 'count' 为 0
  console.log(this.state.count);  // -> 0
  this.setState({ count: this.state.count + 1 })
  this.setState({ count: this.state.count + 1 })
  this.setState({ count: this.state.count + 1 })
  console.log(this.state.count);  // -> 0
}
```

第一，两次的打印都为 0，因为 `setState` 是个异步 API，只有同步代码运行完毕才会执行。 `setState` 异步的原因我认为在于， `setState` 可能会导致 DOM 的重绘，如果调用一次就马上去进行重绘，那么调用多次就会造成不必要的性能损失，设计成异步的话，就可以将多次调用放入一个队列中，在恰当的时候统一进行更新过程。

第二，虽然调用了三次 `setState`,但是 `count` 的值还是为 1.因为多次调用会合并为一次，只有当更新结束后 `state` 才会改变，三次调用等同于如下代码

```js
Object.assign(
  {},
  { count: this.sate.count + 1 },
  { count: this.sate.count + 1 },
  { count: this.sate.count + 1 }
)
```

当然你也可以通过以下方式来实现调用三次 `setState` 使得 `count` 为 3

```js
handle() {
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
}
```

如果你想在每次调用 `setState` 后获得正确的 `state`，可以通过如下代码实现

```js
handle() {
  this.setState((prevState) => ({ count: prevState.count + 1 }), () => {
    console.log(this.state);
  })
}
```

## Redux 源码分析

首先让我们来看下 `combineReducers` 函数

```js
// 传入一个 object
export default function combineReducers(reducers) {
  // 获取该 Object 的 key 值
  const reducerKeys = Object.keys(reducers)
  // 过滤后的 reducers
  const finalReducers = {}
  // 获取每一个 key 对应的 value
  // 在开发环境下判断值是否为 undefined
  // 然后将值类型是函数的值放入 finalReducers
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provied for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  // 拿到过滤后的 reducers 的 key 值
  const finalReducerKeys = Object.keys(finalReducers)
  // 在开发环境下判断，保存不期望 key 的缓存用以下面做警告
  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }
  let shapeAssertionError
  try {
    // 该函数解析在下面
    asserReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }
  // combineReducers 函数返回一个函数，也就是合并后的 reducer 函数
  // 该函数返回总的 state
  // 并且你也可以发现这里使用了闭包，函数里面使用到了外面的一些属性
  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }
    // 该函数解析在下面
    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }
    // state 是否改变
    let hasChanged = false
    // 改变后的 state
    const nextState = {}
    for (let i = 0;i < finalReducerKeys.length; i++) {
      // 拿到相应的 key
      const key = finalReducerKeys[i]
      // 获得 key 对应的 reducer 函数
      const reducer = finalReducerKeys[key]
      // state 树下的 key 是与 finalReducers 下的 key 相同的
      // 所以你在 combineReducers 中传入的参数的 key 即代表了各个 reduce 也代表了各个 state
      const previousStateForkey = state[key]
      // 然后执行 reducer 函数获得该 key 值对应的 state
      const nextStateForkey = reducer(previousStateForkey, action)
      // 判断 state 的值， undefined 的话就报错
      if (typeof nextStateForkey === 'undefined') {
        const errorMessage = getUnexpectedStateShapeWarningMessage(key, action)
        throw new Error(errorMessage)
      }
      // 然后将 value 塞进去
      nextState[key] = nextStateForkey
      // 如果 state 改变
      hasChanged = hasChanged || nextStateForkey !== previousStateForkey
    }
    // state 只要改变，就返回新的 state
    return hasChanged ? nextState : state
  }
}
```

`combineReducers` 函数总的来说很简单，总结来说就是接收一个对象，将参数过滤后返回一个函数。该函数里有一个过滤参数后的对象 finalReducers,遍历该对象，然后执行对象中的每一个 reducer 函数，最后将新的 state 返回。

接下来让我们来看看 combineReducers 中用到的两个函数

```js
// 这是执行的第一个用于抛错的函数
function asserReducerShape(reducers) {
  // 将 combineReducers 中的参数遍历
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
    // 给他传一个 action
    const initialState =reducer(undefined, { type: ActionTypes.INIT })
    // 如果得到的 state 为 undefined 就抛错
    if (typeof initState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during Initialization.` +
        `If the state passed to the reducer is undefined,you must` +
        `explicity return the initial state.the initial state may` +
        `not be undefined.If you don't want to set a value for this reducer,` +
        `you can use null instead of undefined.`
      )
    }
    // 再过滤一次，考虑到万一你在 reducer 中给 ActionTypes.INIT 返回了值
    // 传入一个随机的 action 判断值是否为 undefined
    const type =
    '@@redux/PROBE_UNKNOWN_ACTION_' +
    Math.random()
  })
}
```
