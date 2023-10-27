# 使用工具
## [speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin) 

测算打包过程中各plugin,loader所耗费的时间


## [webpack-bundle-analyzer](https://npmjs.com/package/webpack-bundle-analyzer)

分析打包出来的各资源大小，方便对过大的包进行优化

# 提速方案
## cache-loader

### 效果 

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646202167189-e06397aa-93de-4b3a-94f1-e9b9f8d17083.png#clientId=u334ae171-e172-4&from=paste&height=277&id=ue40fa45c&originHeight=277&originWidth=575&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17096&status=done&style=none&taskId=ue8488d49-681b-42e5-8f07-8cab761a6a8&title=&width=575)


![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646202211120-08df609a-a668-42c5-b4d0-17279a1265a5.png#clientId=u334ae171-e172-4&from=paste&height=506&id=u1a6277b0&originHeight=506&originWidth=452&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41918&status=done&style=none&taskId=u5d7562de-4445-41ac-8edc-bfd52113a60&title=&width=452)


![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646202357808-9f42983a-605f-41fa-a857-fe4ccace5753.png#clientId=u334ae171-e172-4&from=paste&height=178&id=uf02879a6&originHeight=178&originWidth=474&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13252&status=done&style=none&taskId=ufe328cc4-2989-423b-8fac-b37d4bc472f&title=&width=474)

### 原理

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646878510417-a9806a98-7198-49aa-b380-b0a41a5fe164.png#clientId=ua44d2009-3654-4&from=paste&height=672&id=uea4c4ca8&originHeight=672&originWidth=1114&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93698&status=done&style=none&taskId=uea20f36b-a802-4ffd-ba1a-113f24cb2b5&title=&width=1114)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646878931394-5b8a9e5f-3939-4f16-8c95-afff841da0ef.png#clientId=ua44d2009-3654-4&from=paste&height=618&id=u10ecdf78&originHeight=618&originWidth=891&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68606&status=done&style=none&taskId=ua61abcff-6333-43b0-bc1b-b03579c0d2d&title=&width=891)


## 公共模块提取
### [autodll-webpack-plugin](https://www.npmjs.com/package/autodll-webpack-plugin)
dll全称`dynamic link library`，是微软率先提出的概念，译作动态链接库。
分别通过两个`plugin`来运行
**dllPlugin**
进行构建资源文件，生成一个`mainfest.json`字典,字典里存储模块与id的映射
**dllRefrencePlugin**
通过id对模块进行引入
`autodll-webpack-plugin`是对两个插件的整合，减少使用的复杂度。
#### 原理
##### 资源加载方式
![配置](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646982295500-4dd7a9fd-f3c3-4e9b-8dd5-5fd1673609f5.png#clientId=u255c305b-d3e8-4&from=paste&height=337&id=u225dfb31&originHeight=337&originWidth=927&originalType=binary&ratio=1&rotation=0&showTitle=true&size=37026&status=done&style=none&taskId=u189d8d3d-0f7c-41aa-8c9c-d6d6d66cdf9&title=%E9%85%8D%E7%BD%AE&width=927 "配置")
![mainfest.json](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646982578385-aa71e0c7-4be9-4948-935c-9c98a29a2329.png#clientId=u255c305b-d3e8-4&from=paste&height=781&id=u1f2c96b9&originHeight=781&originWidth=860&originalType=binary&ratio=1&rotation=0&showTitle=true&size=60645&status=done&style=none&taskId=uac501025-6196-4f4d-9ec4-94d192df4aa&title=mainfest.json&width=860 "mainfest.json")

![vendor](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646982069952-c77a66df-b62b-40dd-9a6c-d566c7114eff.png#clientId=u255c305b-d3e8-4&from=paste&height=658&id=u18e8cb56&originHeight=658&originWidth=1735&originalType=binary&ratio=1&rotation=0&showTitle=true&size=134397&status=done&style=none&taskId=u04445020-3e16-4136-a85e-c70f89d9d06&title=vendor&width=1735 "vendor")
`vendor`不太好辨认，我们可以粘入控制台

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646982688619-dfc43ebe-b4a0-4097-9f0e-ff67f5eb0f8a.png#clientId=u255c305b-d3e8-4&from=paste&height=542&id=ud7ed91ec&originHeight=542&originWidth=895&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60918&status=done&style=none&taskId=u2291e118-0ea2-4912-8ce5-fa6a866aad7&title=&width=895)
其他模块同理，输入模块索引，获取对应模块。

##### 打包缓存方式
![package.json.hash](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646983152209-27cfa530-1477-4b53-9be7-8b90b0184e9e.png#clientId=uad38fd5a-1f8d-4&from=paste&height=720&id=u950f4c97&originHeight=720&originWidth=1773&originalType=binary&ratio=1&rotation=0&showTitle=true&size=143979&status=done&style=none&taskId=udfe632d5-3927-45b8-8cdd-90ebf8fc961&title=package.json.hash&width=1773 "package.json.hash")

![package.json](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646983079841-d6cccd34-d548-468e-a8e9-e3d823b6c04b.png#clientId=u255c305b-d3e8-4&from=paste&height=689&id=ua52dbdc7&originHeight=689&originWidth=1686&originalType=binary&ratio=1&rotation=0&showTitle=true&size=134440&status=done&style=none&taskId=u92f29d72-4924-4a92-8af6-58c6b7e8513&title=package.json&width=1686 "package.json")

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646983641898-7582de68-cacf-4591-81c8-6d337a3d50da.png#clientId=uad38fd5a-1f8d-4&from=paste&height=195&id=uda6e52d8&originHeight=195&originWidth=610&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14738&status=done&style=none&taskId=ufa4e6343-fa48-4991-85e6-6b81778bff7&title=&width=610)
![validateCache.js](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646983395743-a291c901-3873-498f-b294-ebff7ae2b7e0.png#clientId=uad38fd5a-1f8d-4&from=paste&height=658&id=ue295a2e7&originHeight=658&originWidth=1618&originalType=binary&ratio=1&rotation=0&showTitle=true&size=256943&status=done&style=none&taskId=u1ccfb131-baca-42b4-b644-4baa29b96c6&title=validateCache.js&width=1618 "validateCache.js")

1. 缓存目录下`autodll-webpack-plugin`内存在`production_instance_0_cda54207bafebb237a6e726378cf1b00`目录
2. 存在`packge.json.hash`
3. 当前`packge.json`和`packge.json.hash`对比无有变化

若不满足上述条件，就会重新写入`packge.json.hash`

### 对比dll方式和spiltChunks方式
可参考这个问题
[webpack-common-chunks-plugin-vs-webpack-dll-plugin](https://stackoverflow.com/questions/41890855/webpack-common-chunks-plugin-vs-webpack-dll-plugin)

因为在模块打包方式上，dll需要手动声明需要打入一起的模块
```javascript
const dllLibs = [
  'react',
  'react-dom',
  'scheduler',
  'lottie-web',
  'html2canvas',
  'lodash',
  '@xyz/antd-mobile',
  'dayjs'
]
new AutoDllPlugin({
    inject: true, // will inject the DLL bundle to index.html
    debug: true,
    filename: '[name]_[hash].js',
    path: './dll',
    context: __dirname,
    entry: {
      vendor: dllLibs
    }
  })
```
而`spiltChunks`可以自由定义模块提取的`最小引用`，`最小体积`，以及`正则匹配`，功能更加强大智能。
(需要webpack4及以上才能使用，低版本可用`commonChunkPlugin`替代)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879978421-ab7b8145-efa4-4d15-bdad-c4bf1580def2.png#clientId=ua44d2009-3654-4&from=paste&height=492&id=rP12a&originHeight=492&originWidth=881&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51481&status=done&style=none&taskId=ua965ca68-a2c9-40e5-b589-cc618dfa6f6&title=&width=881)

### 模块分布
**优化前**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879474228-349d9ae7-abe6-489a-a163-2f8533897691.png#clientId=ua44d2009-3654-4&from=paste&height=924&id=u7f1dbde3&originHeight=924&originWidth=1493&originalType=binary&ratio=1&rotation=0&showTitle=false&size=290394&status=done&style=none&taskId=ufef887e8-f86b-4fc6-8930-7f7a428eb3d&title=&width=1493)
**优化后**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879067298-f35f5381-8b5f-4fb5-86c6-f77a0015bbdd.png#clientId=ua44d2009-3654-4&from=paste&height=873&id=u4cc4bd2b&originHeight=873&originWidth=1433&originalType=binary&ratio=1&rotation=0&showTitle=false&size=313986&status=done&style=none&taskId=uc255be94-ebd1-45ef-996c-3cfdef37aee&title=&width=1433)
### 资源体积大小
**优化前**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879660780-95080077-f6aa-465b-991e-3b9ccfc1b235.png#clientId=ua44d2009-3654-4&from=paste&height=241&id=u1752fff3&originHeight=241&originWidth=826&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39255&status=done&style=none&taskId=u90f39c1b-13d9-4739-ab05-d034dddd6c5&title=&width=826)
**优化后**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879111370-9c5a1bff-cfd4-4224-a98b-4246150353f5.png#clientId=ua44d2009-3654-4&from=paste&height=236&id=ufac06b6a&originHeight=236&originWidth=834&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44408&status=done&style=none&taskId=u953e0fc9-9978-4a81-babb-d8879aea681&title=&width=834)

### 打包耗时
**优化前**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879557047-6ae3eb3f-98a9-400e-abf1-c8fbf3a40440.png#clientId=ua44d2009-3654-4&from=paste&height=269&id=u5090da4c&originHeight=269&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13914&status=done&style=none&taskId=u992e1e43-4193-4c58-94d9-8a346449c57&title=&width=1040)
**优化后**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646879146482-46828d71-efbc-432d-8cdf-6bca6327cb04.png#clientId=ua44d2009-3654-4&from=paste&height=209&id=u1d8d12ef&originHeight=209&originWidth=1140&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12586&status=done&style=none&taskId=ud0d12f28-033d-49b5-9ee2-869433ff4b1&title=&width=1140)

# 接入方式
因为`nextjs`内置了`webpack`打包脚本，所以我们要在外侧复写
## 内置配置 ![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646880511264-9e1b6154-f81e-41b5-8ff1-a69ca934f939.png#clientId=ua44d2009-3654-4&from=paste&height=494&id=ufaa186da&originHeight=494&originWidth=1741&originalType=binary&ratio=1&rotation=0&showTitle=false&size=103329&status=done&style=none&taskId=uf3218aa5-4cd7-4b62-9a4e-12c0e51674d&title=&width=1741)
## 复写
在`next.config.js`中进行配置
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881114743-cc4672b0-ab67-4aae-863c-8f7a1bd8492e.png#clientId=ua44d2009-3654-4&from=paste&height=399&id=ubcaab9d2&originHeight=399&originWidth=663&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48865&status=done&style=none&taskId=ud8b94c7e-af38-4d5e-968b-939c04abcff&title=&width=663)

```javascript
// customWebpack

/**
 * @title: webpack.extend.js
 * @projectName fe-xyz-wap-next
 * @description: webpack 扩展配置
 * @author zhangyunpeng0126
 * @date 2022/3/29:37
 */
const path = require('path')

module.exports = (config, {isServer}) => {
  const isDevelopment = process.env.NODE_ENV === 'development'

  let splitChunksConfig
  const splitChunksConfigs = {
    dev: {cacheGroups: {default: false, vendors: false}},
    prod: {
      chunks: 'all',
      cacheGroups: {
        default: false,
        vendors: false,
        commons: {name: 'commons', chunks: 'all', minChunks: 10},
        react: {
          name: 'commons',
          chunks: 'all',
          test: /[\\/]node_modules[\\/](react|react-dom|scheduler)[\\/]/
        },
      },
    },
  }

  if (isDevelopment) {
    splitChunksConfig = splitChunksConfigs.dev
  } else {
    splitChunksConfig = splitChunksConfigs.prod
  }

  config.optimization = {
    ...config.optimization,
    splitChunks: isServer ? false : splitChunksConfig,
  }


  config.module.rules = config.module.rules.filter(
    (item) => `${item.test}` !== '/\\.(jpe?g|png|svg|gif|ico|webp)$/',
  )


  config.module.rules.push({
    test: /\.(jpe?g|png|svg|gif|ico|webp)$/,
    use: [
      'cache-loader',
      {
        loader: 'url-loader',
        options: {
          limit: 2048,
          fallback: 'file-loader',
          publicPath: `/_next/static/images/`,
          outputPath: `${isServer ? '../' : ''}static/images/`,
          name: '[name]-[hash].[ext]',
        },
      },
    ],
  })
  config.resolve.alias = {
    ...config.resolve.alias,
    '@/components': path.resolve(__dirname, '.', 'src/components'),
    '@/utils': path.resolve(__dirname, '.', 'src/utils'),
    '@/models': path.resolve(__dirname, '.', 'src/models'),
    '@/requests': path.resolve(__dirname, '.', 'src/requests'),
    '@/assets': path.resolve(__dirname, '.', 'src/assets'),
  }
  return config
}

```

# 测试
## ci打包速度测试
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881233585-71c39386-2060-45ee-be8f-9f90a3e047be.png#clientId=ua44d2009-3654-4&from=paste&height=155&id=ub2df12e4&originHeight=155&originWidth=1184&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26017&status=done&style=none&taskId=u8ebfce28-def0-40f0-ace1-1e0f1b09170&title=&width=1184)

## 可用性测试

### 增量
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881375784-c53f0c2b-55ae-4d19-a102-439427f8f2b4.png#clientId=ua44d2009-3654-4&from=paste&height=164&id=u02ca4d2a&originHeight=164&originWidth=392&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7035&status=done&style=none&taskId=ub02ca986-f512-49b4-8e02-b214375a09a&title=&width=392)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881731322-24ace0c7-5998-4837-a9d0-4675c36ba5d4.png#clientId=ua44d2009-3654-4&from=paste&height=706&id=u13aee84f&originHeight=706&originWidth=441&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33131&status=done&style=none&taskId=u5f52dc06-4f5b-45eb-aca0-812bcf75234&title=&width=441)

### 存量更新
改变标签颜色
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881751932-f545b5ff-c9a5-4517-8086-59c96405a23b.png#clientId=ua44d2009-3654-4&from=paste&height=96&id=u65636859&originHeight=96&originWidth=369&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5823&status=done&style=none&taskId=u3fe6db5d-c5b7-4cb6-90c9-7db5064b4d8&title=&width=369)![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646881766804-1887c967-e29d-47bc-8965-1f779c949c9d.png#clientId=ua44d2009-3654-4&from=paste&height=683&id=ubca2dc25&originHeight=683&originWidth=385&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31587&status=done&style=none&taskId=u94eb40b5-2601-4284-bc7a-50c5adcc719&title=&width=385)

# 其他优化思路
## 提升依赖查找速度
webpack对模块的搜索本质上是遍历资源树

## 提升loader解析速度
loader要对对应扩展名的文件进行解析，遍历，替换，例如`babel-loader`

## 提高插件处理速度
插件对代码的压缩和混淆，cpu会是速度的瓶颈

好消息是`next`内置配置都从这三个方面进行了优化，让笔者轻松了不少。 

# 知识拓展
## IgnorePlugin
一个可忽略包中某些资源的库，比如`moment.js`中的时区信息
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646985696865-a88c3bdc-14b3-4860-b6a0-14011f69a1a6.png#clientId=u255c305b-d3e8-4&from=paste&height=52&id=uc331b379&originHeight=52&originWidth=689&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9709&status=done&style=none&taskId=ubd3c17ff-c53b-4fc1-9f17-39a85813746&title=&width=689)
方式是使用正则匹配
## [modules-with-no-loaders](https://stackoverflow.com/questions/61526290/webpack-what-are-modules-with-no-loaders-how-to-optimize-them)
使用[speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)时发现有些模块未被loader处理
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1512483/1646986290449-49a2a0b7-dc40-4cc6-b9b3-c9c095364fe2.png#clientId=u255c305b-d3e8-4&from=paste&height=297&id=ue4b146c6&originHeight=297&originWidth=679&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30491&status=done&style=none&taskId=u02b5b935-a031-4a0f-aaa7-6b7258fd7f3&title=&width=679)
因为存在`loader`需要`exclude`的模块，比如`node_modules`里的三方包，已经经过相应loader转换，我们不必再进行处理，这样可以提高`loader`转换速度，是正常的
如果`modules-with-no-loaders`耗时较长，可能是引入的库比较多导致的。



