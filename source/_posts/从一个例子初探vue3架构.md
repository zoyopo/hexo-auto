# 从一个例子开始(更新流程)
```vue
// option 模式
<template>
<div class="about">
  <h1>This is an about page</h1>
  <div @click="increase">{{count}}</div>
  </div>
</template>
<script>
  export default {
    data() {
      return {       
        count:0
      }
    },
    methods:{
      increase(){
        this.count++
      }
    }
  }
</script>
<style>
  
  @media (min-width: 1024px) {
    .about {
      /*min-height: 100vh;*/
      /*display: flex;*/
      /*align-items: center;*/
    }
  }
</style>

```
##  this.count (get this.count)++

```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js
const PublicInstanceProxyHandlers = {
    get({ _: instance }, key) {
      if (key[0] !== '$') {
              const n = accessCache[key];
              if (n !== undefined) {
                  switch (n) {
                      case 1 /* SETUP */:
                          return setupState[key];
                      case 2 /* DATA */:
                          return data[key];
                      case 4 /* CONTEXT */:
                          return ctx[key];
                      case 3 /* PROPS */:
                          return props[key];
                      // default: just fallthrough
                  }
              }
              else if (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) {
                  accessCache[key] = 1 /* SETUP */;
                  return setupState[key];
              }
              else if (data !== EMPTY_OBJ && hasOwn(data, key)) {
                  accessCache[key] = 2 /* DATA */;
                  return data[key];
              }
              else if (
              // only cache other properties when instance has declared (thus stable)
              // props
              (normalizedProps = instance.propsOptions[0]) &&
                  hasOwn(normalizedProps, key)) {
                  accessCache[key] = 3 /* PROPS */;
                  return props[key];
              }
              else if (ctx !== EMPTY_OBJ && hasOwn(ctx, key)) {
                  accessCache[key] = 4 /* CONTEXT */;
                  return ctx[key];
              }
              else if (!__VUE_OPTIONS_API__ || shouldCacheAccess) {
                  accessCache[key] = 0 /* OTHER */;
              }
          }
    }
}   


```
## this.count++(set this.count)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1648352923225-1736ccd8-14c3-4a7f-93b0-3689b4a7940f.png#clientId=u2e7d69c1-1d93-4&from=paste&height=160&id=u90127191&originHeight=160&originWidth=617&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12713&status=done&style=none&taskId=u9a4151d2-bc30-4989-861e-246baafa81f&title=&width=617)



```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js
const PublicInstanceProxyHandlers = {
  //...
  // instance为vue实例
  set({ _: instance }, key, value) {
    const { data, setupState, ctx } = instance;
    if (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) {
      setupState[key] = value;
      return true;
    }
    else if (data !== EMPTY_OBJ && hasOwn(data, key)) {
      data[key] = value;
      return true;
    }
    else if (hasOwn(instance.props, key)) {
      (process.env.NODE_ENV !== 'production') &&
        warn(`Attempting to mutate prop "${key}". Props are readonly.`, instance);
      return false;
    }
    if (key[0] === '$' && key.slice(1) in instance) {
      (process.env.NODE_ENV !== 'production') &&
        warn(`Attempting to mutate public property "${key}". ` +
             `Properties starting with $ are reserved and readonly.`, instance);
      return false;
    }
    else {
      if ((process.env.NODE_ENV !== 'production') && key in instance.appContext.config.globalProperties) {
        Object.defineProperty(ctx, key, {
          enumerable: true,
          configurable: true,
          value
        });
      }
      else {
        ctx[key] = value;
      }
    }
    return true;
  },
  //....
}
```
## instance
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1648353316430-d4d59b1f-2e7d-43cc-a4d5-80c0a3ecf85b.png#clientId=u2e7d69c1-1d93-4&from=paste&height=258&id=u2d03fda9&originHeight=258&originWidth=309&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14312&status=done&style=none&taskId=u192f3bcd-1a87-46c1-ba37-2f5cfebf7be&title=&width=309)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1648353400517-886591c4-b07f-4f7b-83ca-f3bc01cc120d.png#clientId=u2e7d69c1-1d93-4&from=paste&height=236&id=uea370a97&originHeight=279&originWidth=388&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23847&status=done&style=none&taskId=u770a44dd-8c1a-4576-8919-0d6c6e36d1a&title=&width=328)

## data proxy setter

```javascript
// @vue/reactivity/dist/reactivity.esm-bundler.js
function createSetter(shallow = false) {
    return function set(target, key, value, receiver) {
        let oldValue = target[key];
        if (isReadonly(oldValue) && isRef(oldValue) && !isRef(value)) {
            return false;
        }
        if (!shallow && !isReadonly(value)) {
            if (!isShallow(value)) {
                value = toRaw(value);
                oldValue = toRaw(oldValue);
            }
            if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
                oldValue.value = value;
                return true;
            }
        }
        const hadKey = isArray(target) && isIntegerKey(key)
            ? Number(key) < target.length
            : hasOwn(target, key);
        const result = Reflect.set(target, key, value, receiver);
        // don't trigger if target is something up in the prototype chain of original
        if (target === toRaw(receiver)) {
            if (!hadKey) {
                trigger(target, "add" /* ADD */, key, value);
            }
            else if (hasChanged(value, oldValue)) {
                trigger(target, "set" /* SET */, key, value, oldValue);
            }
        }
        return result;
    };
}
```
## trigger
```javascript
// @vue/reactivity/dist/reactivity.esm-bundler.js
function trigger(target, type, key, newValue, oldValue, oldTarget) {
    const depsMap = targetMap.get(target);
    if (!depsMap) {
        // never been tracked
        return;
    }
    let deps = [];
    if (type === "clear" /* CLEAR */) {
        // collection being cleared
        // trigger all effects for target
        deps = [...depsMap.values()];
    }
    else if (key === 'length' && isArray(target)) {
        depsMap.forEach((dep, key) => {
            if (key === 'length' || key >= newValue) {
                deps.push(dep);
            }
        });
    }
    else {
        // schedule runs for SET | ADD | DELETE
        if (key !== void 0) {
            deps.push(depsMap.get(key));
        }
        // also run for iteration key on ADD | DELETE | Map.SET
        switch (type) {
            case "add" /* ADD */:
                if (!isArray(target)) {
                    deps.push(depsMap.get(ITERATE_KEY));
                    if (isMap(target)) {
                        deps.push(depsMap.get(MAP_KEY_ITERATE_KEY));
                    }
                }
                else if (isIntegerKey(key)) {
                    // new index added to array -> length changes
                    deps.push(depsMap.get('length'));
                }
                break;
            case "delete" /* DELETE */:
                if (!isArray(target)) {
                    deps.push(depsMap.get(ITERATE_KEY));
                    if (isMap(target)) {
                        deps.push(depsMap.get(MAP_KEY_ITERATE_KEY));
                    }
                }
                break;
            case "set" /* SET */:
                if (isMap(target)) {
                    deps.push(depsMap.get(ITERATE_KEY));
                }
                break;
        }
    }
    const eventInfo = (process.env.NODE_ENV !== 'production')
        ? { target, type, key, newValue, oldValue, oldTarget }
        : undefined;
    if (deps.length === 1) {
        if (deps[0]) {
            if ((process.env.NODE_ENV !== 'production')) {
                triggerEffects(deps[0], eventInfo);
            }
            else {
                triggerEffects(deps[0]);
            }
        }
    }
    else {
        const effects = [];
        for (const dep of deps) {
            if (dep) {
                effects.push(...dep);
            }
        }
        if ((process.env.NODE_ENV !== 'production')) {
            triggerEffects(createDep(effects), eventInfo);
        }
        else {
            triggerEffects(createDep(effects));
        }
    }
}
```
## triggerEffects
```javascript
// @vue/reactivity/dist/reactivity.esm-bundler.js
function triggerEffects(dep, debuggerEventExtraInfo) {
    // spread into array for stabilization
    for (const effect of isArray(dep) ? dep : [...dep]) {
        if (effect !== activeEffect || effect.allowRecurse) {
            if ((process.env.NODE_ENV !== 'production') && effect.onTrigger) {
                effect.onTrigger(extend({ effect }, debuggerEventExtraInfo));
            }
            if (effect.scheduler) {
                effect.scheduler();
            }
            else {
                effect.run();
            }
        }
    }
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1648354235268-ab4b0337-0630-475e-b668-ab6bc63b3a65.png#clientId=u2e7d69c1-1d93-4&from=paste&height=270&id=u5824ce20&originHeight=270&originWidth=420&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25362&status=done&style=none&taskId=u25eb5cc4-87d2-4b35-9bed-e2fcf512638&title=&width=420)

## queueJob
```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js
function queueJob(job) {
    // the dedupe search uses the startIndex argument of Array.includes()
    // by default the search index includes the current job that is being run
    // so it cannot recursively trigger itself again.
    // if the job is a watch() callback, the search will start with a +1 index to
    // allow it recursively trigger itself - it is the user's responsibility to
    // ensure it doesn't end up in an infinite loop.
    if ((!queue.length ||
        !queue.includes(job, isFlushing && job.allowRecurse ? flushIndex + 1 : flushIndex)) &&
        job !== currentPreFlushParentJob) {
        if (job.id == null) {
            queue.push(job);
        }
        else {
            queue.splice(findInsertionIndex(job.id), 0, job);
        }
        queueFlush();
    }
}
```
## queueFlush
```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js
function queueFlush() {
    if (!isFlushing && !isFlushPending) {
        isFlushPending = true;
        currentFlushPromise = resolvedPromise.then(flushJobs);
    }
}
```
## flushJobs
```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js
function flushJobs(seen) {
    isFlushPending = false;
    isFlushing = true;
    if ((process.env.NODE_ENV !== 'production')) {
        seen = seen || new Map();
    }
    flushPreFlushCbs(seen);
    // Sort queue before flush.
    // This ensures that:
    // 1. Components are updated from parent to child. (because parent is always
    //    created before the child so its render effect will have smaller
    //    priority number)
    // 2. If a component is unmounted during a parent component's update,
    //    its update can be skipped.
    queue.sort((a, b) => getId(a) - getId(b));
    // conditional usage of checkRecursiveUpdate must be determined out of
    // try ... catch block since Rollup by default de-optimizes treeshaking
    // inside try-catch. This can leave all warning code unshaked. Although
    // they would get eventually shaken by a minifier like terser, some minifiers
    // would fail to do that (e.g. https://github.com/evanw/esbuild/issues/1610)
    const check = (process.env.NODE_ENV !== 'production')
    // checkRecursiveUpdates 檢查遞歸更新是否存在更新依賴于更新的情況（導致遞歸死循環）
        ? (job) => checkRecursiveUpdates(seen, job)
        : NOOP;
    try {
        for (flushIndex = 0; flushIndex < queue.length; flushIndex++) {
            const job = queue[flushIndex];
            if (job && job.active !== false) {
                if ((process.env.NODE_ENV !== 'production') && check(job)) {
                    continue;
                }
                // console.log(`running:`, job.id) execute job
                callWithErrorHandling(job, null, 14 /* SCHEDULER */);
            }
        }
    }
    finally {
        flushIndex = 0;
        queue.length = 0;
        flushPostFlushCbs(seen);
        isFlushing = false;
        currentFlushPromise = null;
        // some postFlushCb queued jobs!
        // keep flushing until it drains.
        if (queue.length ||
            pendingPreFlushCbs.length ||
            pendingPostFlushCbs.length) {
            flushJobs(seen);
        }
    }
}
```

## run Job

```javascript
// @vue/reactivity/dist/reactivity.esm-bundler.js
class ReactiveEffect {
    constructor(fn, scheduler = null, scope) {
        this.fn = fn;
        this.scheduler = scheduler;
        this.active = true;
        this.deps = [];
        this.parent = undefined;
        recordEffectScope(this, scope);
    }
    run() {
        if (!this.active) {
            return this.fn();
        }
        let parent = activeEffect;
        let lastShouldTrack = shouldTrack;
        while (parent) {
            if (parent === this) {
                return;
            }
            parent = parent.parent;
        }
        try {
            this.parent = activeEffect;
            activeEffect = this;
            shouldTrack = true;
            trackOpBit = 1 << ++effectTrackDepth;
            if (effectTrackDepth <= maxMarkerBits) {
                initDepMarkers(this);
            }
            else {
                cleanupEffect(this);
            }
            return this.fn();
        }
        finally {
            if (effectTrackDepth <= maxMarkerBits) {
                finalizeDepMarkers(this);
            }
            trackOpBit = 1 << --effectTrackDepth;
            activeEffect = this.parent;
            shouldTrack = lastShouldTrack;
            this.parent = undefined;
        }
    }
```

## setupRenderEffect中的componentUpdateFn（mouted render diff update）
```javascript
// /@vue/runtime-core/dist/runtime-core.esm-bundler.js 
const componentUpdateFn = () => {
            if (!instance.isMounted) {
                let vnodeHook;
                const { el, props } = initialVNode;
                const { bm, m, parent } = instance;
                const isAsyncWrapperVNode = isAsyncWrapper(initialVNode);
                toggleRecurse(instance, false);
                // beforeMount hook
                if (bm) {
                    invokeArrayFns(bm);
                }
                // onVnodeBeforeMount
                if (!isAsyncWrapperVNode &&
                    (vnodeHook = props && props.onVnodeBeforeMount)) {
                    invokeVNodeHook(vnodeHook, parent, initialVNode);
                }
                toggleRecurse(instance, true);
                if (el && hydrateNode) {
                    // vnode has adopted host node - perform hydration instead of mount.
                    const hydrateSubTree = () => {
                        if ((process.env.NODE_ENV !== 'production')) {
                            startMeasure(instance, `render`);
                        }
                        instance.subTree = renderComponentRoot(instance);
                        if ((process.env.NODE_ENV !== 'production')) {
                            endMeasure(instance, `render`);
                        }
                        if ((process.env.NODE_ENV !== 'production')) {
                            startMeasure(instance, `hydrate`);
                        }
                        hydrateNode(el, instance.subTree, instance, parentSuspense, null);
                        if ((process.env.NODE_ENV !== 'production')) {
                            endMeasure(instance, `hydrate`);
                        }
                    };
                    if (isAsyncWrapperVNode) {
                        initialVNode.type.__asyncLoader().then(
                        // note: we are moving the render call into an async callback,
                        // which means it won't track dependencies - but it's ok because
                        // a server-rendered async wrapper is already in resolved state
                        // and it will never need to change.
                        () => !instance.isUnmounted && hydrateSubTree());
                    }
                    else {
                        hydrateSubTree();
                    }
                }
                else {
                    if ((process.env.NODE_ENV !== 'production')) {
                        startMeasure(instance, `render`);
                    }
                    const subTree = (instance.subTree = renderComponentRoot(instance));
                    if ((process.env.NODE_ENV !== 'production')) {
                        endMeasure(instance, `render`);
                    }
                    if ((process.env.NODE_ENV !== 'production')) {
                        startMeasure(instance, `patch`);
                    }
                    patch(null, subTree, container, anchor, instance, parentSuspense, isSVG);
                    if ((process.env.NODE_ENV !== 'production')) {
                        endMeasure(instance, `patch`);
                    }
                    initialVNode.el = subTree.el;
                }
                // mounted hook
                if (m) {
                    queuePostRenderEffect(m, parentSuspense);
                }
                // onVnodeMounted
                if (!isAsyncWrapperVNode &&
                    (vnodeHook = props && props.onVnodeMounted)) {
                    const scopedInitialVNode = initialVNode;
                    queuePostRenderEffect(() => invokeVNodeHook(vnodeHook, parent, scopedInitialVNode), parentSuspense);
                }
                // activated hook for keep-alive roots.
                // #1742 activated hook must be accessed after first render
                // since the hook may be injected by a child keep-alive
                if (initialVNode.shapeFlag & 256 /* COMPONENT_SHOULD_KEEP_ALIVE */) {
                    instance.a && queuePostRenderEffect(instance.a, parentSuspense);
                }
                instance.isMounted = true;
                if ((process.env.NODE_ENV !== 'production') || __VUE_PROD_DEVTOOLS__) {
                    devtoolsComponentAdded(instance);
                }
                // #2458: deference mount-only object parameters to prevent memleaks
                initialVNode = container = anchor = null;
            }
            else {
                // updateComponent
                // This is triggered by mutation of component's own state (next: null)
                // OR parent calling processComponent (next: VNode)
                let { next, bu, u, parent, vnode } = instance;
                let originNext = next;
                let vnodeHook;
                if ((process.env.NODE_ENV !== 'production')) {
                    pushWarningContext(next || instance.vnode);
                }
              
                // Disallow component effect recursion during pre-lifecycle hooks.
                toggleRecurse(instance, false);
              // 开始调用钩子函数
                if (next) {
                    next.el = vnode.el;
                    updateComponentPreRender(instance, next, optimized);
                }
                else {
                    next = vnode;
                }
                // beforeUpdate hook
                if (bu) {
                    invokeArrayFns(bu);
                }
                // onVnodeBeforeUpdate
                if ((vnodeHook = next.props && next.props.onVnodeBeforeUpdate)) {
                    invokeVNodeHook(vnodeHook, parent, next, vnode);
                }
                // 结束调用钩子函数
                toggleRecurse(instance, true);
                // render  生成更新后的树
                if ((process.env.NODE_ENV !== 'production')) {
                    startMeasure(instance, `render`);
                }
                const nextTree = renderComponentRoot(instance);
                if ((process.env.NODE_ENV !== 'production')) {
                    endMeasure(instance, `render`);
                }
                const prevTree = instance.subTree;
                instance.subTree = nextTree;
                if ((process.env.NODE_ENV !== 'production')) {
                    startMeasure(instance, `patch`);
                }
              // 更新真实dom树
                patch(prevTree, nextTree, 
                // parent may have changed if it's in a teleport
                hostParentNode(prevTree.el), 
                // anchor may have changed if it's in a fragment
                getNextHostNode(prevTree), instance, parentSuspense, isSVG);
                if ((process.env.NODE_ENV !== 'production')) {
                    endMeasure(instance, `patch`);
                }
                next.el = nextTree.el;
                if (originNext === null) {
                    // self-triggered update. In case of HOC, update parent component
                    // vnode el. HOC is indicated by parent instance's subTree pointing
                    // to child component's vnode
                    updateHOCHostEl(instance, nextTree.el);
                }
                // updated hook
                if (u) {
                    queuePostRenderEffect(u, parentSuspense);
                }
                // onVnodeUpdated
                if ((vnodeHook = next.props && next.props.onVnodeUpdated)) {
                    queuePostRenderEffect(() => invokeVNodeHook(vnodeHook, parent, next, vnode), parentSuspense);
                }
                if ((process.env.NODE_ENV !== 'production') || __VUE_PROD_DEVTOOLS__) {
                    devtoolsComponentUpdated(instance);
                }
                if ((process.env.NODE_ENV !== 'production')) {
                    popWarningContext();
                }
            }
        };
```

## render和patch下回再探(￣▽￣)"
