在用hooks写复杂页面时，很容易 因为声明hooks导致一个页面大半是hooks代码，看上去比较凌乱，应该给hooks分分类，再依次注入到页面中才比较优雅。

## How

### 我们希望输入的代码

![athooks2.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371609125-4a5eabaa-dc43-4248-b5d7-fde76f36da03.png#align=left&display=inline&height=712&originHeight=712&originWidth=1349&size=72803&status=done&style=none&width=1349)

![athooks3.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371617183-0b0d18b4-4325-4f2d-8866-52b7d0c54e94.png#align=left&display=inline&height=656&originHeight=656&originWidth=1429&size=55127&status=done&style=none&width=1429)

![athooks1.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371624749-b900c42f-3867-49d9-bdd4-7e424f432a3c.png#align=left&display=inline&height=724&originHeight=724&originWidth=1544&size=97593&status=done&style=none&width=1544)

想达成的效果：在函数组件中写入`//@state` `//@effect`标记  然后将该目录下的`effect.js` ,`state.js`中的代码注入到该位置

### 用loader来进行实现

1. 我们先在`webpack.config.js`中声明一下loader 配置 以及作用范围

![wpconfig.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371704493-714c300e-7ce5-4ae7-96bd-8d8542494bee.png#align=left&display=inline&height=1560&originHeight=1560&originWidth=1564&size=241328&status=done&style=none&width=1564)

2. 编写loader

![athooks4.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371719877-7495b3e3-bde9-4964-95e7-5cb2053ffe1f.png#align=left&display=inline&height=1200&originHeight=1200&originWidth=1430&size=268955&status=done&style=none&width=1430)

 ![athooks5.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371738581-af403cfe-9210-4d53-bdea-8533d06c2c12.png#align=left&display=inline&height=1344&originHeight=1344&originWidth=1530&size=298868&status=done&style=none&width=1530)

### 最终打包输出

1. 代码

![athooks6.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371766979-cba91624-4b90-478e-985e-b701a953d5b3.png#align=left&display=inline&height=1452&originHeight=1452&originWidth=1312&size=220183&status=done&style=none&width=1312)

2. 页面

![athooks8.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1597371783665-667aae05-af5a-4ee5-9c32-045c45e08fdb.png#align=left&display=inline&height=471&originHeight=471&originWidth=1095&size=35923&status=done&style=none&width=1095)

## 结语

其实原理很简单，就是字符串的替换，但是借助`webpack`和`node`，真的就可以为所欲为。
