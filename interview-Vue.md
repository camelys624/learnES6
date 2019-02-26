---
  ebook:
    title: interview-Vue
    authors: ykk
---

# Vue

## NextTick 原理分析

`nextTick` 可以让我们在下次 DOM 更新循环结束之后执行回调，用于获得更新后的 DOM。

在 Vue2.4 之前都是使用的 microtask，但是 microtask 的优先级过高，在某些情况下可能会出现比时间冒泡更快的情况，但是如果都使用 macrotask 有可能会出现渲染的性能问题。所以在新版本中，会默认使用 microtask，但是在特殊情况下会使用 macrotasks,比如 v-on。

对于实现 macrotasks，会先判断是否能使用 `setImmediate`，不能的话降级为 `MessageChannel`，以上都不行的话就使用 `setTimeout`

```js
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimeFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (
  typeof MessageChannel !== 'undefined' &&
  (isNative(MessageChannel) ||
    // PhotomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]')
){
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimeFunc = () => {
    port.postMessage(1)
  }
} else {
  // istanbul ignore next
  macroTimeFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

`nextTick` 同时也支持 Promise 的使用，会判断是否实现了 Promise

```js
export function nextTick(cb?: Function, ctx?: Object) {
  let _resolve
  // 将回调函数整合进一个数组中
  callbacks.push(() => {
    if(cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if(_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimeFunc()
    } else {
      microTimeFunc()
    }
  }
  // 判断是否可以使用 Promise
  // 可以的话给 _resolve 赋值
  // 这样回调函数就能以 promise 的方式调用
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(reslove => {
      _resolve = resolve
    })
  }
}
```

## 生命周期分析

声明周期函数就是组件在初始化或者数据更新时会触发的钩子函数。

![Vue生命周期](./static/img/live.png)

在初始化时，会调用以下代码，声明周期就是通过 `callHook` 调用的

```js
Vue.prototype._init = function(options) {
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm, 'beforeCreate')  // 拿不到 props data
  initInjections(vm)
  initState(vm)
  initProvide(vm)
  callHook(vm, 'created')
}
```

可以发现在以上代码中， `beforeCreate` 调用的时候，是获取不到 props 或者 data 中的数据的，因为这些数据的初始化都在 `initState` 中。

接下来会执行挂载函数

```js
export function mountComponent {
  callHook(vm, 'beforeMount')
  // ...
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
}
```

`beforeMount` 就是挂载前执行的，然后开始创建 VDOM 并替换成真实 DOM，最后执行 `mounted` 钩子。这里会有一个判断逻辑，如果是外部 `new Vue({})` 的话，不会存在 `$vnode`,所以执行 `mounted` 钩子了。如果有子组件的话，会递归挂载子组件，只有当所有子组件全部挂载完毕，才会执行根组件的挂载钩子。

接下来是数据更新时才会调用的钩子函数

```js
function flushSchedulerQueue() {
  // ...
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    if (watcher.before) {
      watcher.before()  // 调用 beforeUpdate
    }
    id = watcher.id
    has[id] = null
    watcher.run()
    // in dev build, check and stop circular updates.
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'you may have an infinite update loop' +
          (watcher.user ? `in watcher with expression "${watcher.expression}"`: `in a component render function.`)，
          watcher.vm
        )
        break
      }
    }
  }
  callUpdateHooks(updatedQueue)
}

function callUpdateHooks(queue) {
  let i = queue.length
  while (i--) {
    const watcher = queue[i]
    const vm = watcher.vm
    if (vm._watcher === watcher && vm._isMounted) {
      callHook(vm, 'updated')
    }
  }
}
```

上图还有两个生命周期没有说，分别为 `activated` 和 `deactived`，这两个钩子函数是 `keep-alive` 组件独有的。用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactived` 钩子函数，命中缓存选然后会执行 `actived` 钩子函数。

最后就是销毁组件的钩子函数了。

```js
Vue.prototype.$destroy = function() {
  // ...
  callHook(vm, 'beforeDestroy')
  vm._isBeingDestroyed = true
  // remove self from parent
  const parent = vm.$parent
  if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
    remove(parent.$children, vm)
  }
  // teardown watchers
  if (vm._watcher) {
    vm._watcher.teardown()
  }
  let i = vm._watcher.length
  while (i--) {
    vm._watchers[i].teardown()
  }
  // remove reference from data ob
  // frozen object may not have observer
  if (vm._data.__ob__) {
    vm._data.__ob__.vmCount--
  }
  // call the last hook...
  vm._isDestroyed = true
  // invoke destroy hooks on current rendered tree
  vm.__path__(vm._node, null)
  // fire destroyed hook
  callHook(vm, 'destroyed')
  // turn off all instance listeners.
  vm.$off()
  // remove __vue__ reference
  if (vm.$el) {
    vm.$el.__vue__ = null
  }
  // release circular reference (##6759)
  if (vm.$vnode) {
    vm.$vnode.parent = null
  }
}
```

在执行销毁操作前会调用 `beforeDestroy` 钩子函数，然后进行一系列的销毁操作，如果有子组件的话，也会递归销毁子组件，所有子组件都会销毁完毕才会执行根组件的 `destroyed` 钩子函数。

## VueRouter 源码解析

### 重要函数思维导图

以下思维导图罗列了源码中重要的一些函数

![思维导图](./static/img/thought.png)

接下来看下 `install` 函数的部分实现

```js
export function install(Vue) {
  // 确保 install 调用一次
  if (install.installed && _Vue === vue) return
  install.installed = true
  // 把 Vue 赋值给全局变量
  _Vue = vue
  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if(
      isDef(i) &&
      isDef((i = i.data)) &&
      isDef((i = i.registerRouteInstance))
    ){
      i(vm, callVal)
    }
  }
  // 给每个子组件的钩子函数混入实现
  // 可以发现在 'beforeCreate' 钩子执行时
  // 会初始化路由
  Vue.mixin({
    beforeCreate() {
      // 判断组件是否存在 router 对象，该对象只在根组件上有
      if (isDef(this.$options.router)) {
        // 根路由设置为自己
        this._routerRoot = this
        this._router = this.$options.router
        // 初始化路由
        this._router.init(this)
        // 很重要，为 _route 属性实现双向绑定
        // 触发组件渲染
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        // 用于 router-view 层级判断
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed() {
      registerInstance(this)
    }
  })
  // 全局注册组件 router-link 和 router-view
  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)
}
```

对于路由注册来说，核心就是调用 `Vue.use(VueRouter)`，使得 VueRouter 可以使用 Vue。然后通过 Vue 来调用 VueRouter 的 `install` 函数。在该函数中，核心就是给组件混入钩子函数和全局注册两个路由组件。

### VueRouter 实例化

在安装插件后，对 VueRouter 进行实例化。

```js
const Home = { template: '<div>home</div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>'}

// 3. Create the router
const router = new VueRouter({
  mode: 'hash',
  base: __dirname,
  routes: [
    { path: '/', component: Home }, // all paths are defined without the hash.
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar }
  ]
})
```

来看一下 VueRouter 的构造函数

```js
constructor(optios: RouterOptios = {}) {
  // ....
  // 路由匹配对象
  this.matcher = createMatcher(optios.routers || [], this)

  // 根据 mode 采取不同的路由方式
  let mode = options.mode || 'hash'
  this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
  if (this.fallback) {
    mode = 'hash'
  }
  if (!inBrowser) {
    mode = 'abstract'
  }
  this.mode = mode

  switch (mode) {
    case 'history':
      this.history = new HTML5history(this, options.base)
      break
    case 'hash':
      this.history = new HTML5history(this, options.base, this.fallback)
      break
    case 'abstract':
      this.history = new AbstractHistory(this, options.base)
      break
    default:
      if (process.env.NODE_ENV !== 'production') {
        assert(false, `invalid mode: ${mode}`)
      }
  }
}
```

在实例化 VueRouter 的过程中，核心是创建一个路由匹配对象，并且根据 mode 来采取不同的路由方式。

### 创建路由匹配对象

```js
export function createMatcher(
  routes: Array<RouteConfig>,
  router: VueROuter
): Matcher {
  // 创建路由映射表
  const { pathList, pathMap, nameMap } = createRouteMap(routes)

  function addRoutes(routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
  }
  // 路由匹配
  function match(
    raw: RawLocation,
    currentRoute?: Route,
    redirecteFrom?: Location
  ): Route {
    // ...
  }

  return {
    match,
    addRoutes
  }
}
```

`createMatcher` 函数的作用就是创建路由映射表，然后通过闭包的方式让 `addRoutes` 和 `match` 函数能够使用路由映射表的几个对象，最后返回一个 `Matcher` 对象。

接下来看 `createMatcher` 函数是如何创建映射表的

```js
export function createRouteMap(
  routes: Array<RouteConfig>,
  oldPathList?: Array<string>,
  oldPathMap?: Dictionary<RouteRecord>,
  oldNameMap?: Dictionary<RouteRecord>
): {
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  namePath: Dictionary<RouteRecord>
} {
  // 创建映射表
  const pathList: Array<string> = oldPathList || []
  const pathMap: Dictionary<RouteRecord> = oldPathMap || Object.create(null)
  const nameMap: Dictionary<RouteRecord> = oldNameMap || Object.create(null)
  // 遍历路由配置，为每个配置添加路由记录
  routes.forEach(route => {
    addRouteRecord(pathList, pathMap, nameMap, route)
  })
  // 确保通配符在最后
  for (let i = 0, l = pathList.length; i < 1; i++) {
    if (pathList[i] === '*') {
      pathList.push(pathList.splice(i, 1)[0])
      l--
      i--
    }
  }
  return {
    pathList,
    pathMap,
    nameMap
  }
}
// 添加路由记录
function addRouteRecord(
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>,
  route: RouteConfig,
  parent?:RouteRecord,
  matchAs?: string
) {
  // 获得路由配置下的属性
  const { path, name } = route
  const pathToRegexpOptions: pathToRegexpOptions = route.pathToRegexpOptions || {}
  // 格式化 url，替换 /
  const normalizedPath = normalizedPath(path, parent, pathToRegexpOptions.strict)
  // 生成记录对象
  const record: RouteRecord = {
    path: normalizedPath,
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
    components: route.components || { default: route.component },
    instances: {},
    name,
    parent,
    matchAs,
    redirect: route.redirect,
    beforeEnter: route.beforeEnter,
    meta: route.meta || {},
    props:
      route.props == null
        ? {}
        : route.components
        ? route.props
        : { default: route.props }
  }
  if (route.children) {
    // 递归路由配置的 children 属性，添加路由记录
    route.children.forEach(child => {
      const childMatchAs = matchAs
        ? cleanPath(`${matchAs}/${child.path}`)
        : undefined
      addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
    })
  }
  // 如果路由有别名的话
  // 给别名也添加路由记录
  if (route.alises !== undefined) {
    const aliases = Array.isArray(route.alias) ? route.alias : [route.alias]

    aliases.forEach(alias => {
      const aliasRoute = {
        path: alias,
        children: route.children
      }
      addRouteRecord(
        pathList,
        pathMap,
        nameMap,
        aliasRoute,
        parent,
        record.path || '/'  // matchAs
      )
    })
  }
  // 更新映射表
  if (!pathMap[record.path]) {
    pathList.push(record.path)
    pathMap[record.path] = record
  }
  // 命名路由添加记录
  if (name) {
    if (!nameMap[name]) {
      nameMap[name] = record
    } else if (process.env.NODE_ENV !== 'production' && !matchAs) {
      warn(
        false,
        `Duplicate named routes definition: ` +
        `{ name: "${name}", path: "${record.path}" }`
      )
    }
  }
}
```

以上就是创建路由匹配对象的全过程,通过用户配置的路由规则来创建对应的路由映射表.

### 路由初始化

当根组件调用 `beforeCreate` 钩子函数时,会执行以下代码

```js
beforeCreate() {
  // 只有根组件有 router 属性,所以根组件初始化时会初始化路由
  if (isDef(this.$options.router)) {
    this._routerRoot = this
    this._router = this.$options.router
    this._router.init(this)
    Vue.util.defineReactive(this, '_route', this._router.history.current)
  } else {
    this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
  }
  registerInstance(this, this)
}
```

接下来看路由初始化会做些什么

```js
init(app: any /* Vue component instance */) {
  // 保存组件实例
  this.apps.push(app)
  // 如果根组件已经有了就返回
  if (this.app) {
    return
  }
  this.app = app
  // 赋值路由模式
  const history = this.history
  // 判断路由模式,以哈希模式为例
  if (history instanceof HTML5History) {
    history.transitionTo(history.getCurrentLocation())
  } else if (history instanceof HashHistory) {
    // 添加 hashchange 监听
    const setHashListener = () => {
      history.setupListeners()
    }
    // 路由跳转
    history.transitionTo(
      history.getCurrentLocation(),
      setupHashListener,
      setupHashListener
    )
  }
  // 该回调会在 transitionTo 中调用
  // 对组件的 _route 属性进行赋值,触发组件渲染
  history.listen(route => {
    this.app.forEach(app => {
      app._route = route
    })
  })
}
```

在路由初始化时,核心就是进行路由的跳转,改变 URL 然后渲染对应的组件.接下来来看以下路由是如何进行跳转的.

### 路由跳转

```js
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  // 获取匹配的路由信息
  const route = this.router.match(location, this.current)
  // 确认切换路由
  this.confirmTransition(route, () => {
    // 以下为切换路由成功或失败的回调
    // 更新路由信息,对组件 _route 属性进行赋值,触发组件渲染
    // 调用 afterHooks 中的钩子函数
    this.updateRoute(route)
    // 添加 hashchange 监听
    onComplete && onComplete(route)
    // 更新 URL
    this.ensureURL()
    // 只执行一次 ready 回调
    if (!this.ready) {
      this.ready = true
      this.readyCbs.forEach(cb => { cb(route) })
    }
  }, err => {
    // 错误处理
    if (onAbort) {
      onAbort(err)
    }
    if (err && !this.ready) {
      this.ready = true
      this.readyErrorCbs.forEach(cb => { cb(err) })
    }
  })
}
```

在路由跳转中,需要先获取匹配的路由信息,所以先来看下如何获取匹配的路由信息

```js
function match {
  raw: RawLocation,
  currentRoute?: Route,
  redirecteFrom?: Location
}: Route {
  // 序列化 url
  // 比如对于该 url 来说 /abc?foo=bar&baz=qux##hello
  // 会序列化路径为 /abc
  // 哈希为 ##hello
  // 参数为 foo: 'bar',baz: 'qux'
  const location = normalizeLocation(raw, currentRoute, false, router)
  const { name } = location
  // 如果是命名路由,就判端記錄中是否有該命名路由配置
  if (name) {
    const record = nameMap[name]
    // 没找到表示没有匹配的路由
    if (!record) return _createRoute(null, location)
    const paramNames = record.regex.keys.filter(key => !key.optional)
      .map(key => key.name)
    // 参数处理
    if (typeof location.param !== 'object') {
      location.params = {}
    }
    if (currentRoute && typeof currentRouteparams === 'object') {
      for (const key in currentRoute.params) {
        if (!(key in location.params) &&
        paramNames.indexOf(key) > -1) {
          location.params[key] = currentRoute.params[key]
        }
      }
    }
    if (record) {
      location.path = fillParams(
        record.path,
        location.params,
        `named route "${name}"`
      )
      return _createRoute(record, location, redirecteFrom)
    }
  } else if (location.path) {
    // 非命名路由处理
    location.params = {}
    for (let i = 0; i < pathList.length; i++) {
      // 查找记录
      const path = pathList[i]
      const record = pathMap[path]
      // 如果匹配路由，则创建路由
      if (matchRoute(record.regex, location.path, location.params)) {
        return _createRoute(record, location, redirecteFrom)
      }
    }
  }
  // 没有匹配的路由
  return _createRoute(null, location)
}
```

接下来看看如何创建路由

```js
// 根据条件创建不同的路由
function _createRoute(
  record: ?RouteRecord,
  location: Location,
  redirecteFrom?: Location
): Route {
  if (record && record.redirect) {
    return redirect(record, redirecteFrom || location)
  }
  if (record && record.matchAs) {
    return alias(record, location, record.matchAs)
  }
  return createRoute(record, location, redirecteFrom, router)
}
export function createRoute(
  record: ?RouteRecord,
  location: Location,
  redirecteFrom?: ?Location,
  router?: VueRouter
): Route {
  const stringifyQuery = router && router.options.stringifyQuery
  // 克隆参数
  let query: any = location.query || {}
  try {
    query = clone(query)
  } catch (e) {}
  // 创建路由对象
  const route: Route = {
    name: location.name || (record && record.name),
    meta: (record && record.meta) || {},
    path: location.path || '/',
    hash: location.hash || '',
    query,
    params: location.params || {},
    fullPath: getFullPath(location, stringifyQuery),
    matched: record ? formatMatch(record) : []
  }
  if (redirecteFrom) {
    route.redirecteFrom = getFullPath(redirecteFrom, stringifyQuery)
  }
  // 让路由对象不可改变
  return Object.freeze(route)
}
// 获得包含当前路由的所有潜逃路径片段的路由记录
// 包含从根路由到当前路由的匹配记录，从上至下
function formatMatch(record: ?RouteRecord): Array<RouteRecord> {
  const res = []
  while (record) {
    res.unshift(record)
    record = record.parent
  }
  return res
}
```

至此匹配路由已经完成，我们回到 `transitionTo` 函数中，接下来执行 `confirmTransition`

```js
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  // 确认切换路由
  this.confirmTransition(route, () => {})
}
confirmTransition(route: Route, onComplete: Function, onAbort?: Function) {
  const current = this.current
  // 中断跳转路由函数
  const abort = err => {
    if (isError(err)) {
      if (this.errorCbs.length) {
        this.errorCbs.forEach(cb => {
          cb(err)
        })
      } else {
        warn(false, 'uncaught error during route navigation:');
        console.error(err);
      }
    }
    onAbort && onAbort(err)
  }
  // 如果是相同的路由就不跳转
  if {
    isSameRoute(route, current) &&
    route.martched.length === current.matched.length
  } {
    this.ensureURL()
    return abort()
  }
  // 通过对比路由解析出可复用的组件，需要渲染的组件，失活的组件
  const { updated, deactivated, activated } = resolveQueue(
    this.current.matched,
    route.matched
  )

  function resolveQueue(
    current: Array<RouteRecord>,
    next: Array<RouteRecord>
  ): {
    update: Array<RouteRecord>,
    activated: Array<RouteRecord>,
    deactivated: Array<RouteRecord>
  } {
    let i
    const max = Math.max(current.length, next.length)
    for (i = 0;i < max; i++) {
      // 当前路由路径和跳转路由路径不同时跳出遍历
      if (current[i] !== next[i]) {
        break
      }
    }
    return {
      // 可复用的组件对应路由
      updated: next.slice(0, i),
      // 需要渲染的组件对应路由
      activated: next.slice(i),
      // 失活的组件对应路由
      deactivated: current.slice(i)
    }
  }
  // 导航守卫数组
  const queue: Array<?navigationGuard> = [].concat(
    // 失活的组件钩子
    extractLeaveGuards(deactivated),
    // 全局 beforeEach 钩子
    this.router.beforeHooks,
    // 在当前路由改变，但是该组件被复用时调用
    extractUpdateHooks(update),
    // 需要渲染组件 enter 守卫钩子
    activated.map(m => m.beforeEnter),
    // 解析异步路由组件
    resolveAsyncComponents(activated)
  )
  // 保存路由
  this.pending = route
  // 迭代器，用于执行 queue 中的导航守卫钩子
  const iterator = (hook: navigationGuard, next) => {
    // 路由不相等就不跳转路由
    if (this.pending !== route) {
      return abort()
    }
    try {
      // 执行钩子
      hook(route, current, (to: any) => {
        // 只有执行了钩子函数中的 next, 才会继续执行下一钩子函数
        // 否则会暂停跳转
        // 以下逻辑是在判断 next() 中的传参
        if (to === false || isError(to)) {
          // next(false)
          this.ensureURL(true)
          abort(to)
        } else if (
          typeof to === 'string' ||
          (typeof to === 'object' &&
          (typeof to.path === 'string' || typeof to.name === 'string'))
        ) {
          // next('/') 或者 next({ path: '/' }) -> 重定向
          abort()
          if (typeof to === 'object' && to.replace) {
            this.replace(to)
          } else {
            this.push(to)
          }
        } else {
          // 这里执行 next
          // 也就是执行下面函数 runQueue 中的 step(index + 1)
          next(to)
        }
      })
    } catch (e) {
      abort(e)
    }
  }
  // 经典的同步执行异步函数
  runQueue(queue, interator, () => {
    const postEnterCbs = []
    const isValid = () => this.current === route
    // 当所有异步组件加载完成后，会执行这里的回调，也就是 runQueue 中的 cb()
    // 接下来执行 需要渲染组件的导航守卫钩子
    const enterGuards = extractEnterGuards(activated, postEnterCbs, isValid)
    const queue = enterGuards.concat(this.router.resolveHooks)
    runQueue(queue, iterator, () => {
      // 跳转完成
      if (this.pending !== route) {
        return abort()
      }
      this.pending = null
      onComplete(route)
      if (this.router.app) {
        this.router.app.$nextTick(() => {
          postEnterCbs.forEach(cb => {
            cb()
          })
        })
      }
    })
  })
}
export function runQueue (queue: Array<?navigationGuard>, fn: Function, cb: Function) {
  const step = index => {
    // 队列中的函数都执行完毕，就执行回调函数
    if (index >= queue.length) {
      cb()
    } else {
      if (queue[index]) {
        // 执行迭代器，用户在钩子函数中执行 next() 回调
        // 回调中判断传参，没有问题就执行 next()，也就是 fn 函数中的第二个参数
        fn(queue[index], () => {
          step(index + 1)
        })
      } else {
        step(index + 1)
      }
    }
  }
  // 取出队列中的第一个钩子函数
  step(0)
}
```

接下来介绍导航守卫

```js
const queue: Array<?navigationGuard> = [].concat(
  // 失活的组件钩子
  extractLeaveGuards(deactivated),
  // 全局 beforeEach 钩子
  this.router.beforeHooks,
  // 在当前路由改变，但是该组件被复用时调用
  extractUpdateHooks(updated),
  // 需要渲染组件 enter 守卫钩子
  activated.map(m => m.beforeEnter),
  // 解析异步路由组件
  resolveAsyncComponents(activated)
)
```
