### 什么是dva-loading

`dva-loading`是dva中的一个插件，由dva自带，主要是用状态表示`effects`里的某个`generator`是否在执行,探究一下`dva-loading`的实现，和插件化的加载方式。
`generator`就是我们常写的 *开头的函数

### 代码入口

![](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591944772720-4dad1ffe-a0cd-4fa2-8ea8-dc9fc3e4593b.png#align=left&display=inline&height=552&originHeight=552&originWidth=237&size=0&status=done&style=none&width=237)

1.方法体
```javascript
function createLoading(opts = {}) {

.............................................

}
export default createLoading;
```

2.方法体内的变量定义 和前置判断 
```javascript


  
const SHOW = '@@DVA_LOADING/SHOW';
const HIDE = '@@DVA_LOADING/HIDE';
const NAMESPACE = 'loading';
const namespace = opts.namespace || NAMESPACE;

  const { only = [], except = [] } = opts;
  if (only.length > 0 && except.length > 0) {
    throw Error('It is ambiguous to configurate `only` and `except` items at the same time.');
  }

  const initialState = {
    global: false,
    models: {},
    effects: {},
  };
```

- `only`是允许的`action` `except`是排除的`action` 两者只需用一个就行了
- `SHOW`和`HIDE`分别是两个`type`来控制`loading`的状态
- `initialState`里的`models`用key value形式来存命名空间和是否正在执行
- `initialState`里的`effects`用key value形式来存`dispatch`的`type`名和是否正在执行
- `initialState`里的`global`表示所有模块有没有正在执行的`generator`

3.reducer

```javascript
// `actionType`比如 ‘count/add’
// 第一个`namespace`默认是`loading`
// 第二个`namespace`是上面的`count`
  const extraReducers = {
    [namespace](state = initialState, { type, payload }) {
      const { namespace, actionType } = payload || {};
      let ret;
      switch (type) {
        case SHOW:
          ret = {
            ...state,
            global: true,
            models: { ...state.models, [namespace]: true },
            effects: { ...state.effects, [actionType]: true },
          };
          break;
        case HIDE: {
          const effects = { ...state.effects, [actionType]: false };
          const models = {
            ...state.models,
						// 这个命名空间里有执行的generator 这个命名空间才为true
            [namespace]: Object.keys(effects).some(actionType => {
              const _namespace = actionType.split('/')[0];
							// 如果不是该命名空间下的actionType
              if (_namespace !== namespace) return false;
              return effects[actionType];
            }),
          };
					// 有一个namespace是true gloal就是true
          const global = Object.keys(models).some(namespace => {
            return models[namespace];
          });
          ret = {
            ...state,
            global,
            models,
            effects,
          };
          break;
        }
        default:
          ret = state;
          break;
      }
      return ret;
    },
  };
```

4.generator执行的监听

```javascript
 function onEffect(effect, { put }, model, actionType) {
    const { namespace } = model;
		// 判断1.如果没有特殊的过滤 2.有白名单且包含 3.有黑名单且不包含
    if (
      (only.length === 0 && except.length === 0) ||
      (only.length > 0 && only.indexOf(actionType) !== -1) ||
      (except.length > 0 && except.indexOf(actionType) === -1)
    ) {
		// 返回一个generator供调用
      return function*(...args) {
        yield put({ type: SHOW, payload: { namespace, actionType } });
        yield effect(...args);
        yield put({ type: HIDE, payload: { namespace, actionType } });
      };
    } else {
      return effect;
    }
  }
```

5.返回结果

```javascript
  return {
    extraReducers,
    onEffect,
  };
```

#### use是如何使用

![](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591944773251-1b25af48-7559-4484-ba9a-27b18848b1fa.png#align=left&display=inline&height=628&originHeight=628&originWidth=228&size=0&status=done&style=none&width=228)
```javascript
export default class Plugin {
constructor() {
this._handleActions = null;
// reduce 把hooks由['onError',...]转化成{onError:[],...}
this.hooks = hooks.reduce((memo, key) => {
memo[key] = [];
return memo;
}, {});
}
use(plugin) {
invariant(isPlainObject(plugin), 'plugin.use: plugin should be plain object');
const { hooks } = this;
for (const key in plugin) {
// 如果是hooks里存在的钩子
if (Object.prototype.hasOwnProperty.call(plugin, key)) {
invariant(hooks[key], plugin.use: unknown plugin property: ${key});
if (key === '_handleActions') {
this._handleActions = plugin[key];
} else if (key === 'extraEnhancers') {
hooks[key] = plugin[key];
} else {
// 把方法push到数组
hooks[key].push(plugin[key]);
}
}
}
}
............................................................
...........................................................
}
```
### 结语

钩子如何调用还需再深入探究，就放到下一篇里探讨，文章写的比较简单，如果有谬误，欢迎指正。
