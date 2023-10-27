## 引言

我们在日常开发中，经常在异步请求的时候 加上loading 来提升用户体验，反映在代码里，就是在各种逻辑中穿插操作loding的操作，那么可不可以倒过来呢

## PART ONE

我们来尝试一下
```javascript
var showLoading = () =>{
    const y = setInterval (()=>{ console.log('loading~~')},500)    
    return  clearInterval(y)    
}
```
 

首先想到写一个函数来模拟loading，但是显然这个函数还缺点东西
```javascript
var showLoading = aysnc (cb) =>{
    const y = setInterval (()=>{ console.log('loading~~')},500)    
    cb && await cb()
    return  clearInterval(y)    
}
```

这样我们可以将一些异步流程塞到这个函数里喽 o(∩_∩)o

## PART TWO

接下来再写一个模拟异步的函数`sleep`
```javascript
var sleep =(time) =>{
  return new Promise((resolve,reject)=>{
       setTimeout(()=>{
            resolve()
     },time)
 })
}
```

ok 执行一下
![loadingA.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1617096340541-1072c361-0328-436b-9891-ea16356b48b3.png#align=left&display=inline&height=149&originHeight=149&originWidth=848&size=11379&status=done&style=none&width=848)
没问题 o(∩_∩)o
但还是缺少了点什么，我们应该在结束后给个`loading complete`提示吧

## PART THREE

那么写一个[AOP](https://baike.baidu.com/item/AOP/1332219?fr=aladdin)函数
就像下面这个


测一下，么问题 

![loadingD.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1617096419370-a5ab369e-61fa-4463-a095-8899f7805c27.png#align=left&display=inline&height=194&originHeight=194&originWidth=411&size=11212&status=done&style=none&width=411)
但是用起来 (┬＿┬)
![loadingB.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1617096445631-0c829ae0-b7a2-4735-a1eb-fc4f8be60871.png#align=left&display=inline&height=83&originHeight=83&originWidth=628&size=8359&status=done&style=none&width=628)
![loadingC,png.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1617096454366-af98e9f2-08cf-4938-804a-f85bd76b3cf7.png#align=left&display=inline&height=192&originHeight=192&originWidth=665&size=10416&status=done&style=none&width=665)

就翻车了 (>_<)

原因仔细想了一下 o_O

- 第一个是因为 `after包装之后的高阶函数`并没有返回`Promise`
- 第二个是因为`事件循环`，详见[事件循环](https://zhuanlan.zhihu.com/p/33058983)

那么再改造一下喽


主要对Promise兼容了一下
![loadingE.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1617096479198-ea044cd7-1f98-47b3-8454-ffb5c0955cb7.png#align=left&display=inline&height=187&originHeight=187&originWidth=647&size=11313&status=done&style=none&width=647)
成功！o(∩_∩)o

## 后记

个人脑洞，欢迎指正
