# 什么是[react-swipeable-views](https://github.com/oliviertassinari/react-swipeable-views) 
 `[react-swipeable-views](https://github.com/oliviertassinari/react-swipeable-views) `是一个触屏端手势滑动进行整屏切换的react组件。


# 使用痛点
## api无法满足业务需求
  有一个场景，当用户在页面中完成某些操作后，才可以上划
但是此组件提供的`disabled`属性，会将上划，下划都禁用


## 页面底部的区域元素会被操作栏遮挡
![](https://cdn.nlark.com/yuque/0/2022/jpeg/1512483/1645684379019-4227da6b-5ac9-41f1-a5ef-1fddc212e38f.jpeg)

因为设计和布局都是按照整个浏览器高度，但是在地址栏和操作栏的影响下 一些定位在底部元素就无法看到了



## 容器高度无法自动适配浏览器视窗高度 

![](https://cdn.nlark.com/yuque/0/2022/jpeg/1512483/1645683107392-1262589e-b28c-4ee3-b604-0f9305d5abd3.jpeg)


   某些浏览器行为（如上图），比如上划到底部后，继续上划，浏览器地址栏会消失，此时视窗大小会发生变化
 如果不进行高度重新适配，`transform:translateY的位置`还是依据之前的视窗高度，造成显示错位

## 原生局部滚动会和组件干架

![1ec411d0488d50c487bf33676fd73e25 (2).gif](https://cdn.nlark.com/yuque/0/2022/gif/1512483/1645685516395-f12a4eff-50ac-4af6-bf61-b9e95046b7a8.gif#clientId=u273298dd-9517-4&from=drop&id=u501a5af0&originHeight=690&originWidth=388&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9737103&status=done&style=none&taskId=u25f1a83d-ae91-4eff-a4b9-e49952899a7&title=)


## swipe时会导致内容区域元素层级变化
![](https://cdn.nlark.com/yuque/0/2022/jpeg/1512483/1646098125926-2b4936db-8dd4-48fe-8b62-dc09e3523647.jpeg)
##  


# 如何解决
## api无法满足业务需求
基本思路：克隆一份源码到本地，对源码进行扩展
### 扩展属性
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1645685854385-0a950859-5a09-4792-922b-5c757e757cd3.png#clientId=u273298dd-9517-4&from=paste&height=323&id=u817057c5&originHeight=323&originWidth=941&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39250&status=done&style=none&taskId=u58e88957-f20f-40be-82ee-ae915f87f7c&title=&width=941)

### 在handleSwipeMove中处理
```javascript
  handleSwipeMove = event => {
  ...............................
  ...............................
  ...............................
  ...............................
    const { axis, 
            children, 
            ignoreNativeScroll,
            onSwitching, 
            resistance,
            disableTouchUp,
            disableTouchDown
          } = this.props;
    const touch = applyRotationMatrix(event.touches[0], axis);
    const isTouchUp = touch.pageX - this.startX < 0
    const isTouchDown = touch.pageX - this.startX > 0
    if (disableTouchUp && isTouchUp) {
      console.log('isTouchUp',isTouchUp)
      return;
    }
    if (disableTouchDown && isTouchDown) {
      return;
    }
  ...............................
  ...............................
  ...............................
  ...............................
 }
```
###  
## 页面底部的区域元素会被操作栏遮挡

### 获取正确height

```javascript
const [rightHeight, setRightHeight] = useState(0)   
useEffect(() => {
     setRightHeight(window.innerHeight)                   
   },[])
```
### 给swipeView中的组件设置高度
```jsx
const getRightHeight = () =>{
	return {height:rightHeight}
}
<SwipeableViews
  resistance
  axis="y"
  animateHeight={true}
  >
  <APage style={getRightHeight()}/>
  <BPage style={getRightHeight()}/>
  <CPage style={getRightHeight()}/> 
</SwipeableViews>
```
```jsx
const A|B|CPage =({style}) => {

  return <div style={style}>
    ...........
    ...........
    ...........
    ...........
  </div>
}
```
## 容器高度无法自动适配浏览器视窗高度 
### 方法一
```javascript
const [animateHeight, setAnimateHeight] = useState(true)
useEffect(() => {
    window.onresize = function () {
      // 重新调整swipeview内元素高度并通知组件重新适配高度
      setRightHeight(window.innerHeight)
      setAnimateHeight(false)
      setTimeout(() => {
        setAnimateHeight(true)
      }, 60)
    }
},[])

```
```jsx
const getRightHeight = () =>{
	return {height:rightHeight}
}
<SwipeableViews
  resistance
  axis="y"
  animateHeight={animateHeight}
  >
  <APage style={getRightHeight()}/>
  <BPage style={getRightHeight()}/>
  <CPage style={getRightHeight()}/> 
</SwipeableViews>
```
这是通过对`animateHeight`的更改触屏对`containerStyle`的高度的修复
见源码
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1645687005926-c47000b9-3a87-41b4-9afb-2e27dcd50aed.png#clientId=u273298dd-9517-4&from=paste&height=129&id=ua4798fd5&originHeight=116&originWidth=1124&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19649&status=done&style=none&taskId=ub10c6cd5-8d18-4c9d-8369-748b2484b7a&title=&width=1248.8889219731468)
这种方法会导致高度瞬间消失，使得页面会瞬间白屏下（如下图）
![d8e8940fb2d62a28d82d553fb257d969 (1).gif](https://cdn.nlark.com/yuque/0/2022/gif/1512483/1645692852874-05ae28c0-a24b-4a6b-b034-91c675a75de0.gif#clientId=ud0947295-46ee-4&from=drop&id=uc69cd585&originHeight=690&originWidth=388&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4656200&status=done&style=none&taskId=u457d5ce3-51cc-4013-ac9c-df68d4c207a&title=)
所以，看到这，笔者开始思考，会不会直接修改`containerStyle`来的更简单呢

### 方法二
```jsx
const getRightHeight = () =>{
	return {height:rightHeight}
}

<SwipeableViews
  resistance
  axis="y"
  containerStyle={getRightHeight()}
  >
  <APage style={getRightHeight()}/>
  <BPage style={getRightHeight()}/>
  <CPage style={getRightHeight()}/> 
</SwipeableViews>
```
事实证明确，确实不会闪屏，但还会存在视窗resize时，下一页内容出现的问题
究其根本原因，因为该组件是使用的`transform:translateY`对一屏位置进行确定，所以屏与屏之间是连续的
当`resize`事件发生时，我们绑定的回调函函数执行，state改变到页面重新绘制是个异步行为，所以页面会先保持未重新适配前的状态，再变为适配后的状态。
## 原生局部滚动会和组件干架
思路：如果放在容器内部会互相感扰，那就放到容器外部
### 将最后一页从`SwipeableViews`容器中拷贝一份到外面
```jsx

<>
<SwipeableViews
  resistance
  axis="y"
  containerStyle={getRightHeight()} 
  >
  <APage style={getRightHeight()}/>
  <BPage style={getRightHeight()}/>
  <CPage style={getRightHeight()}/> 
</SwipeableViews>
<CPage style={getRightHeight()}/> 
</>
```
### 到第三页隐藏swipeView 显示外部第三页
```jsx
const [currentIndex, setCurrentIndex] = useState(0)  

const onSwitching = (index) => {
  setCurrentIndex(index)
}

<>
<SwipeableViews
   style={{display: currentIndex < 2 ? 'block' : 'none'}}
  resistance
  axis="y"
  containerStyle={getRightHeight()}
  onSwitching={onSwitching}
  >
  <APage style={getRightHeight()}/>
  <BPage style={getRightHeight()}/>
  <CPage style={getRightHeight()}/> 
</SwipeableViews>
<CPage style={{...getRightHeight(),display: currentIndex >= 2 ? 'block' : 'none'}}/> 
</>

```

1. 为什么不能使用`currentIndex < 2?<SwipeableViews/>:<CPage>`这种形式？

答：这种方式会导致组件的重新渲染，滑动位置的状态会**丢失**

2. 为什么还要保留`SwipeableViews`内部的`CPage`？

答：保证流畅的过渡，内部`CPage`的存在会使得切换到第三页的临界状态依然会存在过渡动画

### 给第三页加上一个touch事件，保证可从第三页回到第二页 

```jsx
// Parent 回到第二页
 const onTouchDownToLastPage = () => {
    setCurrentIndex(1)
  }

// Child
const CPage =({style,onTouchDownToLastPage}) => {
    const onTouchStart = (e) => {
    if(touchStartY.current === null) {
      setHasTouched(true)
    }
    touchStartY.current = e.changedTouches[0].pageY
  }
    const onTouchMove = (e) => {

    let delta = e.changedTouches[0].pageY - touchStartY.current
    const dom = comicImgRef.current
    const scrollTop = comicWrap.current.scrollTop
    // (delta > 0 保证下滑 Math.abs(scrollTop) < 50 保证滚动调离顶部阈值范围
    if (delta > 0 && Math.abs(scrollTop) < 50) {
      onTouchDownToLastPage && onTouchDownToLastPage()
    }
  }

  return <div style={style} onTouchStart={onTouchStart} onTouchMove={onTouchMove}>
    ...........
    ...........
    <div className={styles.comic} ref={comicImgRef}></div>
    ...........
    ...........
  </div>
}
```

## swipe时会导致内容区域元素层级变化
### 解决方法 

![](https://cdn.nlark.com/yuque/0/2022/jpeg/1512483/1646098126897-4ed8c643-b468-44f7-a78b-a5de8d4d13a3.jpeg)

# 其他开发问题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1645690259850-b1c0ca17-db72-4c33-9acf-35aa9e84cdc3.png#clientId=u273298dd-9517-4&from=paste&height=472&id=u58f6d7b3&originHeight=378&originWidth=253&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70137&status=done&style=none&taskId=uc063bb64-3d1e-4fc7-aaac-e0920054106&title=&width=316.24999528750783)
## 底部引导提示在滑动时需消失，但是当手指滑动到该图标上，会阻止上划
### 原因分析
可能是因为滑动时，将该提示`display:none`处理，导致图层之间的touch事件不连续

### 解决方法
`display:none`改为`opacity:0`
#### 注意 
考虑一下会不会遮挡后面图层的元素

# 结语
前端开发就是逆天而行，共勉。
