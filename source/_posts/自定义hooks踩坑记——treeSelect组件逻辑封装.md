浏览最近写的项目代码，发现涉及树形结构下拉组件，从请求获取数据——>数据渲染节点的过程是分布在页面代码里的，这样页面多了，相同的逻辑都要拷贝一遍，开发体验不是很好，也不利维护。

原来的代码如下
1 .请求数据

```javascript

 useEffect(() => {
 fetchDictionary(dispatch, ['getAllOrgan']) 
 },[])
```

2 . 渲染节点

![result.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943486386-df3df154-af10-4b22-a5ad-7560ec4fc83f.png#align=left&display=inline&height=3726&originHeight=3726&originWidth=2048&size=1029648&status=done&style=none&width=2048)

3 . 组件调用
![treenode-949x1024.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943495607-2071d055-8c20-4ca5-bfd7-f4feab24e530.png#align=left&display=inline&height=1024&originHeight=1024&originWidth=949&size=689404&status=done&style=none&width=949)

现在，我要写一个自定义hooks来封装请求数据和渲染节点的逻辑，显然要支持

- 异步数据获取
- 联动获取数据
- 支持根据数据改变节点特性

ok 开搞

![use-1024x534.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943504031-ff89e842-674c-4b7c-a237-01659b4ef7ec.png#align=left&display=inline&height=534&originHeight=534&originWidth=1024&size=222684&status=done&style=none&width=1024)

注意点有三

- 一是 需要联动的下拉 第一次是不需要去请求的 所以用`validateDepency`方法去鉴别一下
- 二是 对联动参数的`useMemo`缓存 因为传入的`depency` 是个对象 每次渲染都会new一遍（未验证） 所有用`Object.values`将值转成数组，作为依赖项（这里只考虑了一层级）  防止多次请求
- 三是 useEffect 对于需要联动的数据 因为请求的数据在页面渲染之后才获取 所以给`ref`对象上赋值 滞后了 而且`ref`的改变不会触发重新渲染

完成后的调用方式

![useStyle-596x1024.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943519196-cf6b1ef5-8a9e-44b0-841f-28ae14cd784f.png#align=left&display=inline&height=1024&originHeight=1024&originWidth=596&size=491393&status=done&style=none&width=596)

### 然而事情并不是如此简单？！！

这样写，还是有问题,`useLayoutEffect`不能从根本上解决数据获取完毕之前，已经绘制结束的问题

所以，终极解决方案就是 **写成一个hooks组件**！！！
![useTreeSelect-rewrite-agian.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943529442-96f093fd-928c-4a3b-96f1-2420b194d813.png#align=left&display=inline&height=4226&originHeight=4226&originWidth=1952&size=978537&status=done&style=none&width=1952)
这样能够用setState来触发重新绘制，也就不会存在那个问题了。
