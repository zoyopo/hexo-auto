![deep-dive-react-fiber-internals.jpeg](https://cdn.nlark.com/yuque/0/2020/jpeg/1512483/1606209367304-d6b0a44f-dc05-4eef-bcf0-1143304d5800.jpeg#align=left&display=inline&height=486&originHeight=486&originWidth=730&size=88059&status=done&style=none&width=730)
我们知道ReactDOM在后台构建DOM树并将应用程序渲染在屏幕上。但是React实际上是如何构建DOM树的？当应用程序状态更改时，它如何更新树？

在本文中，我将首先说明React在React 15.0.0及之前是如何构建DOM树的，该模型的缺陷以及React 16.0.0的新模型如何解决这些问题。这篇文章将涵盖广泛的概念，这些概念纯粹是内部实现细节，对于使用React进行实际的前端开发并不是严格必需的。

## React15架构

React15架构可以分为两层：

- Reconciler（协调器）—— 负责找出变化的组件
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上

### Renderer（渲染器）

由于`React`支持跨平台，所以不同平台有不同的**Renderer**。我们前端最熟悉的是负责在浏览器环境渲染的**Renderer** —— [ReactDOM](https://www.npmjs.com/package/react-dom)。

除此之外，还有：

- [ReactNative](https://www.npmjs.com/package/react-native)渲染器，渲染App原生组件
- [ReactTest](https://www.npmjs.com/package/react-test-Renderer)渲染器，渲染出纯Js对象用于测试
- [ReactArt](https://www.npmjs.com/package/react-art)渲染器，渲染到Canvas, SVG 或 VML (IE8)

在每次更新发生时，**Renderer**接到**Reconciler**通知，将变化的组件渲染在当前宿主环境。

### Reconciler（协调器）

我们知道，在`React`中可以通过`this.setState`、`this.forceUpdate`、`ReactDOM.render`等API触发更新。

每当有更新发生时，**Reconciler**会做如下工作：

- 调用函数组件、或class组件的`render`方法，将返回的JSX转化为虚拟DOM
- 将虚拟DOM和上次更新时的虚拟DOM对比
- 通过对比找出本次更新中变化的虚拟DOM
- 通知**Renderer**将变化的虚拟DOM渲染到页面上

#### 例子

当React遇到一个类或函数组件时，它会问该元素根据其属性将渲染成什么元素。例如
如果 `<App/>` 组件渲染下面的内容

```jsx
<Form>
  <Button>
    Submit
  </Button>
</Form>
```

React将根据其相应的属性询问`<Form>`和`<Button>`组件它们要渲染成什么。例如，如果该`Form`组件是如下所示的函数组件：

```jsx
const Form = (props) => {
  return(
    <div className="form">
      {props.form}
    </div>
  )
}
```

React将调用 `render` 方法以知道它渲染了哪些元素，并最终将看到它渲染了 `<div>` 并带有一个子元素。React将重复此过程，直到知道页面上每个组件的基础DOM标签元素为止。

为了知道react应用组件树的基础DOM标签元素，递归遍历一个树的过程被称作 `reconciliation`

在`reconciliation`结束时，React知道了DOM树的结果，并且像react-dom或react-native这样的渲染器将应用所需的最小更改集进行更新DOM节点。

因此，这意味着当您调用 `ReactDOM.render()` 或 `setState` 时，React执行了`reconciliation` 。在 `setState` 的情况下，它将执行遍历并通过将新树与渲染的树进行比对来找出树中发生了什么变化。然后，将那些更改应用于当前树，从而更新与调用`setState`相对应的状态。

现在我们知道了什么是`reconciliation`
让我们来看看，这种模型有什么缺陷。

### 帧频

帧频是连续图像出现在显示器上的频率。我们在计算机屏幕上看到的所有内容都是由屏幕上播放的图像或帧组成，这些图像或帧的显示速率瞬间达到了眼睛。

要理解这是什么意思，可以将计算机显示屏看作一本翻页书，而将翻页书的页视为翻页时以一定速率播放的帧。换句话说，计算机显示器不过是一本自动翻页书，当屏幕上的事物发生变化时，它会一直播放。

通常，为了使视频对人眼感觉平滑且瞬间，视频需要以每秒30帧（FPS）的速率播放。高于此值将提供更好的体验。这就是为什么游戏玩家喜欢第一人称射击游戏时更高的帧频的主要原因之一，而精确度非常重要。

话虽这么说，如今大多数设备以60 FPS刷新屏幕-换句话说，就是1/60 = 16.67ms，这意味着每16ms就会显示一个新帧。这个数字非常重要，因为如果React渲染器花费16毫秒以上的时间在屏幕上渲染某些东西，浏览器将丢弃该帧。
![1.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209444112-d12bb814-d056-4111-afbd-d64f1c11857a.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=356543&status=done&style=none&width=2208)

![2.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209458121-fd50e782-607c-444d-8e39-24227fd58d2c.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=359873&status=done&style=none&width=2208)

但是，实际上，浏览器还要执行其他工作，因此您的所有工作都需要在10毫秒内完成。如果您无法满足此预算，则帧速率会下降，并且屏幕上会显示内容抖动。这通常称为卡顿，会影响用户体验。

![3.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209473520-fb86b0d0-9bdf-41f3-acae-917c84b6c4dd.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=487011&status=done&style=none&width=2208)
究其原因，JS可以操作DOM，GUI渲染线程与JS线程是互斥的。所以JS脚本执行和浏览器布局、绘制不能同时执行。

在每16.6ms时间内，需要完成如下工作：

JS脚本执行 -----  样式布局 ----- 样式绘制
当JS执行时间过长，超出了16.6ms，这次刷新就没有时间执行样式布局和样式绘制了。

假设这样一个画面
![4.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209490771-19ef2e22-5b7b-4e4f-9bcc-60224e9fc731.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1197062&status=done&style=none&width=2208)
在组件过于庞大，js线程花费大量时间reconciliation，无法在恰当时间内把控制权交给浏览器，进行gui渲染
![5.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209502972-8b8c713e-9696-40f3-bdd7-f3f53e9ca32d.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=552771&status=done&style=none&width=2208)

### 递归更新的缺点

为了理解为什么会发生这种情况，让我们举一个简单的例子，看看[调用栈中](https://developer.mozilla.org/en-US/docs/Glossary/Call_stack)发生了什么。

```javascript
function fib(n) {
  if (n < 2){
    return n
  }
  return fib(n - 1) + fib (n - 2)
}

fib(10)
```

![call-stack-diagram.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209562131-c274963c-c923-4601-a3b2-d0b5a9093cf7.png#align=left&display=inline&height=352&originHeight=352&originWidth=730&size=14976&status=done&style=none&width=730)

如我们所见，调用栈将每个对 `fib()` 调用推入栈，直到弹 `fib(1)` 出为止，这是要返回的第一个函数调用。然后，它继续推送递归调用，并在到达return语句时再次弹出。这样，它有效地使用了调用栈，直到 `fib(3)` 返回并成为从栈中弹出的最后一项。

由于递归执行，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16ms，用户交互就会卡顿。

那么React15的架构支持异步更新么？让我们看一个例子：

初始化时`state.count = 1`，每次点击按钮`state.count++`

列表中3个元素的值分别为1，2，3乘以`state.count`的结果
:::

我用红色标注了更新的步骤。
![v15.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209606065-5620fcf4-d06e-421c-ad75-228384969d1d.png#align=left&display=inline&height=804&originHeight=804&originWidth=1750&size=75793&status=done&style=none&width=1750)

我们可以看到，**Reconciler**和**Renderer**是交替工作的，当第一个`li`在页面上已经变化后，第二个`li`再进入**Reconciler**。

由于整个过程都是同步的，所以在用户看来所有DOM是同时更新的。

接下来，让我们模拟一下，如果中途中断更新会怎么样？

### 以下是我们模拟中断的情况，实际上`React15`并不会中断进行中的更新

![dist.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209680163-5c6cfb6f-01f0-42cf-addb-b6ce6b9a98a4.png#align=left&display=inline&height=732&originHeight=732&originWidth=1698&size=75691&status=done&style=none&width=1698)

当第一个`li`完成更新时中断更新，即步骤3完成后中断更新，此时后面的步骤都还未执行。

用户本来期望`123`变为`246`。实际却看见更新不完全的DOM！（即`223`）

基于这个原因，`React`决定重写整个架构。

## Fiber Reconciler 的工作原理

### fiber Reconciler 是如何解决上面的问题

答案是在浏览器每一帧的时间中，预留一些时间给JS线程，React利用这部分时间更新组件（在源码中，预留的初始时间是5ms）。

当预留的时间不够用时，React将线程控制权交还给浏览器使其有时间渲染UI，React则等待下一帧时间到来继续被中断的工作。

这种将长任务分拆到每一帧中，像蚂蚁搬家一样一次执行一小段任务的操作，被称为时间切片（time slice）

![6.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209696575-9d7765bf-3eb9-43e5-9353-1f8cddc46ac4.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1327378&status=done&style=none&width=2208)

![12.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209707020-5ffdafdf-d9b1-4a60-80bd-45e9bd4dadb3.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1292353&status=done&style=none&width=2208)

**那么如何实现这种调度的呢？**

### Scheduler（调度器）

既然我们以浏览器是否有剩余时间作为任务中断的标准，那么我们需要一种机制，当浏览器有剩余时间时通知我们。

其实部分浏览器已经实现了这个API，这就是[requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)。但是由于以下因素，React放弃使用：

- 浏览器兼容性
- 触发频率不稳定，受很多因素影响。比如当我们的浏览器切换tab后，之前tab注册的requestIdleCallback触发的频率会变得很低
基于以上原因，React实现了功能更完备的requestIdleCallbackpolyfill，这就是Scheduler。除了在空闲时触发回调的功能外，Scheduler还提供了多种调度优- 先级供任务设置。

![7.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209739708-b2ff8b92-4d28-42c2-9ebd-7da00deb0edf.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1239848&status=done&style=none&width=2208)

我们这里可以姑且用requestIdleCallback来理解

### Fiber节点

我们主要可以关注一几个属性

#### Child

代表我们在组件中调用 `render` 或者直接 `return` 的元素，比如：

```javascript
const Name = (props) => {
  return(
    <div className="name">
      {props.name}
    </div>
  )
}
```

`<Name/>` 组件的 `Child` 是 `div` ,因为它返回 `<div/>` 元素

#### Sibling

表示`render`返回元素列表的情况。

```javascript
const Name= （props）=> { return（[ < Customdiv1 />，< Customdiv2 /> ]）}
```

在上述情况下，`<Customdiv1>`和`<Customdiv2>`是 `<Name>` 的child。这两个Child形成一个单链表。

#### Return

表示返回栈帧，从逻辑上讲，它是返回到父Fiber节点的返回。因此，它代表父代。

#### Alternate

在任何时候，一个组件实例最多具有两个与其对应的 `Fiber` ：当前 `Fiber` 和工作中的 `Fiber` 。当前 `Fiber` 的alternate属性指向工作中的 `Fiber` ，而工作中的 `Fiber` alternate属性指向当前 `Fiber` 。当前的 `Fiber` 表示已经渲染的内容，而从概念上讲，工作中的 `Fiber` 是尚未返回的栈帧。

#### stateNode

Fiber对应的真实DOM节点

### render阶段

#### “递”阶段

首先从rootFiber开始向下深度优先遍历。为遍历到的每个Fiber节点调用beginWork方法 。

该方法会根据传入的Fiber节点创建子Fiber节点，并将这两个Fiber节点连接起来。

组件更新时会使用diff算法,为生成的Fiber节点带上effectTag属性，标记是否增，删，还是改

当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。

#### “归”阶段

在“归”阶段会调用completeWork处理Fiber节点。

当某个Fiber节点执行完completeWork，如果其存在兄弟Fiber节点（即fiber.sibling !== null），会进入其兄弟Fiber的“递”阶段。

如果不存在兄弟Fiber，会进入父级Fiber的“归”阶段。

同样在这个阶段，如果组件是第一次加载，那么会在`filber`结点里构建对应`dom`结点，并在归时把子结点`append`到父节点，如果是更新就处理`props`，并将`effectTag`用单链表串联成`effectList`

“递”和“归”阶段会交错执行直到“归”到rootFiber。至此，render阶段的工作就结束了。

### 例子

```javascript


const Item = (props) => {
  return <div>{props.children}</div>
}

class List extends Component {
  constructor(props) {
    super(props)
		
		this.state={
		  items:[1,2,3]
		}
  }
	onSquare =()=>{
		this.setState((state,props)=>({
		items:state.items.map(item=>item*item)
		}))
	}
  render() {
    <div>
		 <button onClick={onSquare}>^2</button>
      {items.map(item=><Item>{item}</Item>)}
    </div>
  }
}

ReactDOM.render(<List />, document.getElementById('root'))
```

![10.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209774043-efd0ebfe-2645-42da-a734-faf75d3bb854.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=393958&status=done&style=none&width=2208)
![9.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209787492-d4b900d3-3b28-4cbf-b501-897d8906c81a.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=650099&status=done&style=none&width=2208)
![11.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209800762-fafd471e-5e1a-4e37-ac64-6286bb0c924f.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=451771&status=done&style=none&width=2208)

我们可以看到，Fiber树由单链表形式相互链接的同级关系和父子关系的子节点组成。可以使用[深度优先搜索](https://en.wikipedia.org/wiki/Depth-first_search)遍历该树。

在我们点击平方按钮时

询问主线程
![7.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209828689-6889472c-8f62-43e0-992d-d01706fd62f6.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1239848&status=done&style=none&width=2208)

主线程答复
![14.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209839490-642f2684-b815-4c63-a911-5000c46da000.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=608986&status=done&style=none&width=2208)

进行workloop 到hostroot
![16.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209874357-c3d65add-1719-46e8-979e-82d73da326c1.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=518251&status=done&style=none&width=2208)
diff 未改变 就拷贝
![9385F71F84A4769E6E7492388DBFDE1A.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606209987648-1e7ac78e-3499-41a7-a6ef-a5fadca031da.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=308527&status=done&style=none&width=2208)
进行workloop 到list
![18.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210015626-df483e6d-2cba-414f-a5da-e6a8925388bf.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=515476&status=done&style=none&width=2208)

![3FC5ACF00CC52B0308849EBE911D9525.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210055945-5bacb0cb-de4a-418a-8369-6822107f3097.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=684553&status=done&style=none&width=2208)
依次类推 改变的打上标签
![51996CD95DCEFB45573766D75C1C94DB.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210074630-b4c01723-0c58-4968-8602-05a44674c387.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=619588&status=done&style=none&width=2208)

![160220D0A20BD21F3401BC29AD529E1B.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210099655-bc30d015-a7ae-4e2e-8389-5ae221d27840.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1124787&status=done&style=none&width=2208)

![DEF51426F52D138DC5DB171796C69301.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210163220-75a2cf36-d700-4d3c-9dc9-16d1e795dfe1.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1105255&status=done&style=none&width=2208)

但是这时候剩余的时间已经不够了
![BC7DA44873ACE38DB93F80CC46314A77.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210207771-7f4e581a-c56d-4902-9a0a-ebf180cfb7b1.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1244462&status=done&style=none&width=2208)
![00596075F005D8C6B47B3F6C280332F5.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210295951-9e1620df-d924-4595-ad2c-60da0b957441.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=899493&status=done&style=none&width=2208)

这时候我们已经存下了变更

![01095ED6CEF52A2AF55ABDEAD51CA75B.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210319423-cf624f14-be58-4838-a057-32f31c542812.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=380867&status=done&style=none&width=2208)

当主线程处理完了其他事情

![3B04B7DBD9A1EA771ADCC16FB9E1625B.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210360749-30a13921-9484-474f-9795-ad8df730aab8.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=856114&status=done&style=none&width=2208)

![50722078F2C45AFC344D44F8C86A6D57.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210390937-7e17cfbb-0b31-47d0-9ed4-1fd606bd441e.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1285132&status=done&style=none&width=2208)

![C5175102A41F4DE0C9343CD968B35B5F.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210408409-e886f1e6-35f4-4bfc-b65a-bdd079192e26.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1378818&status=done&style=none&width=2208)

### 上述过程我们称为Schedule/render阶段 ，此过程是可中断的，但是进入commit阶段就无法中断

### commit阶段

时间充裕进入commit阶段
![17B5220B298DA2D5443849AC24CFC50A.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210433323-6eb6d5be-c12c-49b1-a250-1aa08b83615e.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=966059&status=done&style=none&width=2208)
此时 effctList链表如图
![67A178CD3158248AB2C46AEF91A617FF.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210454286-54ab6912-54ed-4b56-9c68-9604a07312e7.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=516935&status=done&style=none&width=2208)
然后遍历链表 完成更新
![9176CE80C101745E9BDFAEF0859D7DEC.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210469963-e7643957-ebe6-4044-adfa-f8c469d1127c.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=249090&status=done&style=none&width=2208)
将current指向最新的tree
![6F7EB2C4AD1DAB2D613B8B9846C53582.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210485248-08591187-659a-4c3e-8a6c-cad362b2bed2.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1006530&status=done&style=none&width=2208)

#### commit阶段流程图

![TM20201120114514.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210551539-90e9988f-6b28-43ce-8bba-4eff5f857ce2.png#align=left&display=inline&height=772&originHeight=772&originWidth=731&size=74729&status=done&style=none&width=731)

### 优先级

![6D107B41F80EC2488860F93504FF6D26.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210569835-1dc57532-52ca-4fae-b41b-aac22b959a9f.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=1107988&status=done&style=none&width=2208)

假设在我们 上述第一个Schedule/render阶段之后 ，点击了紧急按钮会发生什么事

![4D8E630216AD46D13EE099C622E00423.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210584550-3c88d04b-78bb-41b5-8c61-88d765157a8b.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=657106&status=done&style=none&width=2208)

按照优先级顺序，这个紧急事件会进行插队

![ACCD015C071FC3CD33CCAB3E2DCDC8E4.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1606210602943-f6636928-addc-48c3-bbf2-22c638d7ac5d.png#align=left&display=inline&height=1242&originHeight=1242&originWidth=2208&size=980176&status=done&style=none&width=2208)

直到这个处理完了，才会去处理剩下的`<Item/>`

#### 如果有很多优先级高的情况呢

我们可以考虑一下，当这种情况出现时，势必会导致低优先级的任务一直被阻塞，无法执行，我们称作`低优先级饿死`

react官方通过expirationTime属性来解决这个问题

> 每一个fiber都分配一个expirationTime属性（其实有多种expirationTime属性），它是大于当时的毫秒数。但调度器执行时，就计算出当前的毫秒数now, 然后now - fiber.expirationTime >= 0，那么这fiber就可以更新了，其priorityLevel会改成ImmediatePriority——司徒正美


#### react17 中的启发式更新算法

expirationTimes模型可以轻松胜任cpu操作，但是不能满足IO操作，会导致高优先级的IO操作会阻断低优先级的cpu操作，即使是某一个组件里的io操作都会阻断整个fiber树的其他cpu操作

所以使用lanes模型来区分IO和cpu操作

具体可见
[React17新特性解读：启发式更新算法](https://my.oschina.net/u/3623645/blog/4541413)

## 参考资料

[A cartoon into Fiber](https://www.youtube.com/embed/ZCuYPiUIONs?version=3&rel=1&fs=1&autohide=2&showsearch=0&showinfo=1&iv_load_policy=1&wmode=transparent)

[React技术揭秘](https://react.iamkasong.com/)

[deep-dive-into-react-fiber-internals](https://blog.logrocket.com/deep-dive-into-react-fiber-internals/)

[React Fiber的优先级调度机制与事件系统](https://zhuanlan.zhihu.com/p/95443185)
