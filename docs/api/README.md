## 组件
### Nuomi
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| id  | '' | store id，不设置会动态创建 |
| async  | function | 异步加载props |
| state  | {} | 初始状态 |
| data  | {} | 临时数据 |
| store  | { dispatch, getState } | 派发和获取当前模块状态，内部创建，不可修改，也无需传递，dispatch可以调用effects中的方法，也可以调用其他模块的effects |
| reducers  | { _updateState } | 更新状态 |
| effects  | {} | 异步或者同步更新状态，异步处理使用async await，用于业务处理，支持对象和函数，函数必须返回对象，内部需dispatch调用reducers中的方法，方法名有$前缀时会被设置为loading状态，通过connect可以在loadings中获取到，不需要开发时手动控制loading状态 |
| render  | function | 渲染组件 |
| onInit  | function | 状态被创建后回调，可以通过this.store更新状态 |

### Ruoter
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| hashPrefix  | '' | hash前缀 |

### NuomiRoute
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| pathPrefix  | '' | 路由path前缀，支持字符串和正则，可实现类似子路由功能 |
其他参数同Nuomi组件

### Route
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| path  | '' | 路由path，支持动态参数 |
| location  | {} | 当前匹配的路由数据，内部创建，不可修改，也无需传递 |
| wrapper  | false | 是否给留有创建一个div容器，可实现缓存功能 |
| reload  | false | 匹配路由后是否重置状态 |
| onBefore  | function | 路由匹配后，在reducer被创建之前回调，返回false将无法展示内容，参数为强制展示回调，调用后可以展示内容 |
| onChange  | function | 路由匹配时回调，支持函数和对象 |
| onLeave  | function | 路由离开时回调，用于决定是否可以离开，暂未实现该功能 |
其他参数同Nuomi组件，
回调的执行顺序是 onBefore > location.data > onChange > onInit，location.data下面路由跳转时会讲到
<br><b>注意：path和wrapper不能异步加载</b>

### Redirect
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| from  | '' | 匹配当前路由path，会重定向至to设置的path |
| to  | '' | 重定向路由path |
| reload  | false | 跳转后是否重置状态 |

### Link
|  props   | value  |  介绍  |
|  ----  | ----  | ---- |
| to  | '' | 跳转后的路由path |
| reload  | false | 跳转后是否重置状态 |

### connect
用法同react-redux connect，区别是获取状态的函数，第一个参数是获取当前模块的状态，第二个参数是获取全部store里的状态

### withNuomi
使用后组件中提供nuomiProps和location2个props，nuomiProps指向Nuomi/NuomiRoute/Route组件的props，location是当前路由数据

## 对象
### router

#### router.location
路由跳转
|  参数   | value  |  介绍  |
|  ----  | ----  | ---- |
| path  | '' | 跳转的路由地址 |
| data  | {} | 值为对象时表示跳转后传递的临时数据，切换路由后该数据将不存在，跳转后的路由模块中可以通过data获取，data对象上面Nuomi组件有提到，当值是函数时，函数的参数可以获取跳转后路由的props，可以通过store更新状态，更新状态发生在onChange之前，当值为布尔时，等同reload |
| reload  | false | 跳转后是否重置状态 |
| force  | true | 当path是当前路由path时，是否强制跳转，reload为true时该值自动为true |
不传参数表示获取当前路由数据
```js
router.location() // 获取当前路由数据
router.location(path, true) // 跳转后刷新
router.location(path, {}, true) // 跳转后传递临时参数并刷新
router.location(path, ({ store }) => { // 跳转后更新状态
    store.dispatch({
        type,
        payload
    });
}))
```

#### router.listener
监听路由变化
|  参数   | value  |  介绍  |
|  ----  | ----  | ---- |
| callback  | function | 接受参数为location，可以获取当前匹配的路由数据 |
返回取消监听方法

#### router.reload
路由刷新，等同router.location(当前path, true) 

#### router.back
后退

#### router.forward
前进

#### router.matchPath
用于匹配路由
|  参数   | value  |  介绍  |
|  ----  | ----  | ---- |
| location  | {} | 路由location对象 |
| path  | '' | 路由设置的path |

### nuomi

#### nuomi.getDefaultProps
获取最新的默认props
```js
// 默认props
{
  state: {
    loadings: {},
  },
  data: {},
  reducers: {
    _replaceState: (state, { payload }) => payload,
    _updateState: (state, { payload }) => ({ ...state, ...payload }),
    _updateLoading: (state, { payload }) => ({
      ...state,
      loadings: { ...state.loadings, ...payload },
    }),
  },
}
```

#### nuomi.config
设置默认props，设置后会覆盖默认值
```js
nuomi.config({
    state: {
        dataSource: [],
    },
    reducers: {
        updateState: (state, { payload }) => ({ ...state, ...payload }),
    },
});
```

#### nuomi.extend
合并props，state、reducers、data等会被浅合并

### store
请参考 [redux store](https://cn.redux.js.org/docs/basics/Store.html)
建议只用下面三个：
#### store.getState
获取所有状态
#### store.dispatch
更新状态，只能触发reducers中的方法，如果想调用effects方法，请使用store.getStore替代
#### store.getStore
获取nuomi组件的store，参数是store id，返回对象包含getState和dispatch