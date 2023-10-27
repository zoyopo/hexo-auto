```tsx
import React, { useState, useEffect } from 'react'
import { Flex } from '@xyz/antd-mobile'
import styles from '../index.less'

type NewFeatureMaskProps = {
  dom: any
}
/***
 *新功能上线的遮罩
 * 实现思路 获取到渲染完的区域 得到宽高和位置
 * 再在遮罩上面定义一个和其大小一致的区域 把位置定上去 再用‘box-shadow’形成遮罩
 */

const NewFeatureMask = ({ dom }: NewFeatureMaskProps) => {
  const [layerStyle, setLayerStyle]: [object, Function] = useState({})
  const [tipStyle, setTipStyle]: [object, Function] = useState({})
  const [visible, setVisible]: [boolean, Function] = useState(true)
  const [isFirstTime, setIsFirstTime]: [boolean, Function] = useState(true)

  useEffect(() => {
    const isFirstTime = window.localStorage.getItem('isFirstTime')
    if (isFirstTime !== 'false') {
      // 限制页面滚动
      document.querySelector('html').style['overflow'] = 'hidden'
      document.querySelector('body').style['position'] = 'fixed'
      document.querySelector('body').style['top'] = '0'
      document.querySelector('body').style['left'] = '0'
      if (dom) {
        const style = {
          width: `${dom.offsetWidth}px`,
          height: `${dom.offsetHeight + 15}px`,
          top: `${dom.offsetTop}px`,
          left: `${dom.offsetLeft}px`,
          boxShadow: 'rgba(29,32,35,0.70) 0px 0px 0px 9999px',
          zIndex: '99',
        }
        setLayerStyle(style)
        const tipStyle = {
          bottom: `${window.innerHeight - dom.offsetTop}px`,
          zIndex: 100,
        }
        setTipStyle(tipStyle)
        window.localStorage.setItem('isFirstTime', 'false')
      }
    } else {
      setIsFirstTime(false)
    }
  }, [dom])

  const onKnow = () => {
    setVisible(false)
    document.querySelector('html').style['overflow'] = 'auto'
    document.querySelector('body').style['position'] = 'static'
  }
  return isFirstTime && visible && dom ? (
    <div className={styles['mask-container']}>
      <Flex direction="column" className={styles['mask-container__tip']} style={tipStyle}>
        <div className={styles['mask-container__tip__msg']}>保单凭证在这里查看哦</div>
        <div className={styles['mask-container__tip__btn']}>
          <button type="button" onClick={onKnow}>
            知道了
          </button>
        </div>
        <div className={styles['mask-container__tip__line']}></div>
      </Flex>
      <div className={styles['mask-container__layer']} style={layerStyle}></div>
    </div>
  ) : null
}

export default NewFeatureMask

```
