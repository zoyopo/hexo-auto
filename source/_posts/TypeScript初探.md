tags: [typescript]
categories: [前端]

---

引言
    因为工作中的项目有用到，并且部门leader比较中意吧，在接下来的项目中可能要继续推行，正好趁此闲暇，了解了解。

## 什么是TypeScript

    摘录自github:
> TypeScript is a language for application-scale JavaScript. TypeScript adds optional types to JavaScript that support tools for large-scale JavaScript applications for any browser, for any host, on any OS. TypeScript compiles to readable, standards-based JavaScript.

    其实就4点

- 包含 `javascript` 在此基础上对 `javascript`加了类型的概念

- 微软开发且开源

- 能运行在任何浏览器 主机 操作系统

- 需要通过构建工具编译变成 `javascript`


    那微软开发TypeScript又是为什么呢，官网是这样解释的：
> 类型允许JavaScript开发者在开发JavaScript应用程序时使用高效的开发工具和常用操作比如静态检查和代码重构。
> 类型是可选的，类型推断让一些类型的注释使你的代码的静态验证有很大的不同。类型让你定义软件组件之间的接口和洞察现有JavaScript库的行为。

    什么意思呢 ?    因为 `javascript`是一个弱类型语言，运行时编译的，会导致无法像 `java`, `c#`一些服务端语言一样 通过 `ide`进行类型推断和检查举个例子    以前我们定义一个字符串，通常会这样
`var str="String"` 
    用 `typescript`
`var str:string="String"` 
    这样就会把类型带着一起声明了。

## 哪些项目适用

     `typescript`的诞生其实就是为了解决一个痛点。伴随着前端项目越来越庞大，业务逻辑越加复杂，项目代码的可维护性以及代码可读性问题。    所以只有几个页面的小项目，其实就没有使用的必要性了。

## 简要介绍ts的一些基本语法

####   基础类型
    分别是

- `boolean`布尔值

- `number`数字

- `string`字符串

- `[]`数组 exp `letstrArr:string[]=['1','2','3']`

- 元组 exp `letTuple:[string,number]=['1',1]`

- 枚举
```typescript
enum Color {Red, Green, Blue} 
let c: Color = Color.Green;
```

    默认 Red往后 从0开始 要覆盖的话 `enumColor{Red=1,Green,Blue}`就从1开始

- `any`不清楚的变量类型 比如动态变量 或者类库

- `void`没有返回类型的函数

- never 永不存在的值的类型(例如 异常) `constthrowErr:never=(err)=>{thrownewError(err)}`

- null undefined类型（两个类型就两值 孤独）

- Object 也就是除 `number`， `string`， `boolean`， `symbol`， `null`或 `undefined`之外的类型

- as语法（写过c#的应该比较熟悉） 
exp:
```typescript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```
# 接口
## 
      今天来扯扯接口，其实我对这个概念也不是很清楚，看了一番文档 ，发现其实和命题作文还挺像。给你规定一个题目和基本立意，不同的人写出来的肯定不一样，一样那肯定坐的比较近啦（^o^）。这里，题目就是一个接口，他将万千思绪约束起来，但是具体怎么写它不管。      但是也有些不一样，程序里的接口往往定义的更加精确，实现也更加明确。
## 官方介绍
TypeScript的核心原则之一是对值所具有的结构进行类型检查。它有时被称做“鸭式辨型法”或“结构性子类型化”。在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
## 声明方式
```typescript
interface Iwatch{
  hour:string,
  minute:string
}
```
## 实现方式
```typescript
let getClockTime=(params:Iwatch):void=>{
   console.log(`now is:${params.hour}H ${params.hour}M`)
}
// 调用
getClockTime({hour:'12',minute:'30'})
```
## 几个特性
1 .可选属性

- 例子

```typescript
let getClockTime=(params:Iwatch):void=>{
   console.log(`now is:${params.hour}H ${params.hour}M`)
}
// 调用
getClockTime({hour:'12',minute:'30'})
```

- 注：用 `?`表示不是必填


2 .只读属性

- 例子

```typescript
interface Iwatch{
 readonly hour:string,
 readonly  minute:string
}
let getClockTime=(params:Iwatch):void=>{
// Cannot assign to 'hour' because it is a read-only property
   params.hour='14' //error
   console.log(`now is:${params.hour}H ${params.hour}M`)
}
```
如果接口属性数量是动态不定的 可以这样
```typescript
interface Iwatch{
 readonly hour:string,
 readonly  minute:string,
 [propName: string]: any;
}
```
3 .可索引类型
```typescript
interface Iwatch{
 [index:number]:string
}
let watches:Iwatch
watches=['huaweiWatch','appleWatch']
```
4 . 函数型

- 例子

```typescript
// 声明接口
interface IplayGame{
  (gameName:string,playHour:number):string
}
// 定义实现
let playGame:IplayGame=(name,hour)=>{
  return `I have play ${name} ${hour} hour${hour>1?'s':''}`
}
// 函数调用
const playGameMsg=playGame('LoL',3)
```

- 注：声明了 函数的参数及返回结果的规约 


5 . 类

- 例子

```typescript
interface Iwatch{
  hour?:string,
  minute?:string,
  setHour:(hour:string)=>void,
  setMinute:(min:string)=>void
}
class Watch implements Iwatch{
   hour?:string;
   minute?:string;
   setHour(hour:string){
    this.hour=hour
   }
   setMinute(min:string){
    this.minute=min
   }
   constructor(h?:string,m?:string){
       this.hour=h
       this.minute=m
   }
}
// 构造函数
let appleWatch=new Watch('10','20')
 console.log(`now is:${appleWatch.hour}H ${appleWatch.hour}M`)
//调用method
let huaweiWatch=new Watch()
appleWatch.setHour('10')
appleWatch.setMinute('30')
console.log(`now is:${huaweiWatch.hour}H ${appleWatch.hour}M`)
```

- 注：类通过 `implements`关键字来实现接口，这里写了相当于一个构造函数


6 .接口之间的继承
```typescript
interface IappleDevice{
  color:string,
  type:string
}
interface Iwatch extends IappleDevice{
  hour?:string,
  minute?:string,
  setHour:(hour:string)=>void,
  setMinute:(min:string)=>void
}
interface IMac extends IappleDevice{
 ...
}
```

- 注：有利于公共部分与定制部分拆分，功能更加单一

## 后记
还有接口签名，接口对类的继承，混合类型接口，个人感觉用到的场景比较少，此处不作介绍，感兴趣可以去官网瞅瞅。写这篇其实也是加深加深自己的印象，欢迎探讨指正。
## 参考
接口 · TypeScript中文网 · TypeScript——JavaScript的超集
