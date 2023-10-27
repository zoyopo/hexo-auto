# 方案草图
![FocusTalkImg_9e08833b-7516-4631-97fe-9eb31789618c.jpeg](https://cdn.nlark.com/yuque/0/2021/jpeg/1512483/1635146152036-bdc807d8-2f32-4549-82a2-d065ae71350c.jpeg#clientId=u4d16be1b-79a4-4&from=drop&id=u7b53bdec&originHeight=3000&originWidth=4000&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2259807&status=done&style=none&taskId=ua7f0cb24-a8c0-49f3-ada6-77f02456ba4&title=)
# 要解决的核心问题
因为右侧边栏的浮动效果，即fixed定位效果 ，在侧边内容过多的情况下，滚动至底部时，会出现侧边栏高度+底部页底高度>视窗高度，由于定位从视窗顶部开始，就会出现侧边栏盖在页底元素上的效果。
# 解决方案
如果内容超长（即会发生遮盖），滚动向下，在即将发生遮盖前，将侧边栏采用absolute定位进行相对于整个文档进行定位，否则就采用fixed定位。
# 问题点
1. 如何计算absolute定位的top值
2. 如何判断会发生遮盖
3.如何判断侧边栏已经到达页脚
4.如何解决异步数据导致页面的高度算不准问题

# 问题方案
1.如草图所示，先要算出侧边栏同级元素最大高度maxContentHeight，使用maxContentHeight+导航区域高度siderBarOffsetTop减去侧边栏本体高度siderBarOffsetHeight，就是absolute定位的top值
2.侧边栏高度siderBarOffsetHeight+底部页底高度footerHeight>视窗高度window.innerHeight 即会发生遮盖
3.判断垂直方向滚动的距离scrollY和第一个问题的出的top值进行比较 大于等于即已到达底部
4.在滚动的回调事件中，计算高度值并使用变量存储，直到前一个值和后一个相同
# 技术方案
1.使用dom API对一些元素高度进行计算，为了减少对dom的重复查询 使用react ref 进行保存
2.监听滚动事件，回调中计算一些需要实时计算的值，对比判断是否达到临界
3.给组件新增一个接口，方便外部属性传入，对侧边栏同级元素进行指定，防止布局复杂导致“侧边栏同级元素最大高度”计算方案无法满足
# 代码实现
```tsx
import React, { ReactNode, useEffect, useRef, useState } from 'react'
import classNames from 'classnames'

import { If } from 'babel-plugin-jsx-control-statements'
import styles from './index.less'
import { useScroll } from 'ahooks'

interface ISidebar {
  // 宽度，默认270
  width?: number
  // 是否展示顶部栏，默认true
  showLine?: boolean
  // 是否需要fixed定位，默认true
  needFixed?: boolean
  // 页面位置，默认右侧
  position?: 'left' | 'right'
  title?: ReactNode
  footer?: ReactNode
  children?: ReactNode
  wrapClassName?: string
  className?: string
  // 相邻内容元素
  nextElement?: HTMLElement
}

const Index = (props: ISidebar) => {
  const {
    width,
    showLine,
    needFixed,
    position,
    title,
    footer,
    children,
    wrapClassName,
    className,
    nextElement,
  } = props
  const [fixed, setFixed] = useState<boolean>(false)
  const sidebarWrapRef = useRef(undefined)
  const sidebarRef = useRef(undefined)
  const sidebarOffsetTopRef = useRef<number>(undefined)
  const sidebarOffsetHeightRef = useRef<number>(undefined)
  const documentScroll = useScroll()
  const [isReachFooter, setIsReach] = useState(false)

  const nextElementHeightRef = useRef<number>(undefined)
  const topValueWhenAbsolute = useRef<number>(undefined)
  const pageTotalHeightRef = useRef<number>(undefined)
  // 侧边栏内容长度是否能盖到底部
  const [contentOverFooter, setIsContentOverFooter] = useState(false)
  useEffect(() => {
    if (sidebarOffsetTopRef.current === undefined) {
      setSideBarContentInfo()
    }
  }, [])

  useEffect(() => {
    if (!needFixed) {
      return
    }
    if (documentScroll?.top > sidebarOffsetTopRef.current) {
      setFixed(true)
    } else {
      setFixed(false)
    }
    // 因为存在异步数据渲染 如果滚动中发现高度变化及时修正
    if (getMaxHeightOfNextElements() !== nextElementHeightRef.current) {
      setWrapHeightByParent()
      getTotalHeightOfPage()
      // setSideBarContentInfo()
      setIsContentOverFooter(isContentOverFooter())
    }
    setIsReach(getIsReachFooter())
  }, [documentScroll.top])

  const setSideBarContentInfo = () => {
    sidebarOffsetTopRef.current = sidebarRef.current.offsetTop
    sidebarOffsetHeightRef.current = sidebarRef.current.offsetHeight
  }

  // 记录相邻内容元素高度 方便计算到footer距离
  const setWrapHeightByParent = () => {
    // 如果传入 优先用传入
    if (nextElement) {
      nextElementHeightRef.current = nextElement?.offsetHeight
      return
    }
    const maxHeight = getMaxHeightOfNextElements()
    nextElementHeightRef.current = maxHeight
  }
  // 获取相邻元素中最大高度 默认
  const getMaxHeightOfNextElements = () => {
    const children = [...sidebarWrapRef.current?.parentElement.children]
    const heightArr = children.map((item) => item?.offsetHeight)
    const maxHeight = Math.max(...heightArr)

    return maxHeight
  }
  // critical value 获取到页脚时距离顶部距离
  const getTopValueWhenReachFooter = (sidebarOffsetHeight) => {
    topValueWhenAbsolute.current = getTopPlusBodyHeight() - sidebarOffsetHeight
    return topValueWhenAbsolute.current
  }

  // 头+内容主体高度
  const getTopPlusBodyHeight = () => {
    return nextElementHeightRef.current + sidebarOffsetTopRef.current
  }
  const getFooterHeight = () => {
    return pageTotalHeightRef.current - getTopPlusBodyHeight()
  }

  const isContentOverFooter = () => {
    const isOver = getFooterHeight() + sidebarRef.current.offsetHeight > window.innerHeight
    return isOver
  }
  const getContentStyle = () => {
    const style = { width }
    if (needFixed) {
      style['top'] = isReachFooter ? topValueWhenAbsolute.current : 0
    }
    return style
  }
  const contentCls = classNames(styles.sidebar__content, {
    [styles.fixed]: needFixed && fixed && !isReachFooter,
    [styles.showLine]: showLine,
    [className]: className,
    [styles.absolute]: isReachFooter && needFixed && contentOverFooter,
  })

  const getIsReachFooter = () => {
    const topValueWhenAbsolute = getTopValueWhenReachFooter(sidebarRef.current.offsetHeight)
    const isReach = documentScroll.top >= topValueWhenAbsolute
    return isReach
  }

  const getTotalHeightOfPage = () => {
    const nextWrapElement = document.querySelector('#__next')
    const layout = nextWrapElement.firstElementChild
    // @ts-ignore
    pageTotalHeightRef.current = layout?.offsetHeight
    return pageTotalHeightRef.current
  }

  return (
    <div
      ref={sidebarWrapRef}
      className={classNames(styles.sidebar, {
        [styles.atLeft]: position === 'left',
        [wrapClassName]: wrapClassName,
      })}
      style={{ width }}
    >
      {fixed && <div className={styles.sidebar__shadow} />}
      <div className={contentCls} ref={sidebarRef} style={getContentStyle()}>
        <If condition={title}>
          <div className={styles.sidebar__title}>{title}</div>
        </If>
        <If condition={children}>
          <div className={styles.sidebar__body}>{children}</div>
        </If>
        <If condition={footer}>
          <div className={styles.sidebar__footer}>{footer}</div>
        </If>
      </div>
    </div>
  )
}

Index.defaultProps = {
  showLine: true,
  needFixed: true,
  width: 270,
  position: 'right',
}

export default Index

```

