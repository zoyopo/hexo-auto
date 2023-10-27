
## 安装相应的包

```json
 "devDependencies": {
    "@babel/core": "^7.10.2",
    "@babel/plugin-proposal-class-properties": "^7.10.1",
    "@babel/plugin-transform-runtime": "^7.10.1",
    "@babel/preset-env": "^7.10.2",
    "@babel/preset-react": "^7.10.1",
    "babel-loader": "^8.1.0",
    "cross-env": "^7.0.2",
    "css-loader": "^3.5.3",
    "html-webpack-plugin": "^4.3.0",
    "react-hot-loader": "^4.12.21",
    "rimraf": "^3.0.2",
    "style-loader": "^1.2.1",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.11.0",
    "websocket": "^1.0.31"
  },
  "dependencies": {
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  }
```

| name | description |
| --- | --- |
| @babel/core | babel核心模块，包含转换代码成为AST语法树等功能 |
| @babel/plugin-proposal-class-properties | 转换类属性插件（react类组件中属性方法转换） |
| @babel/plugin-transform-runtime | 加上es6语法和API的polyfill 并且避免手动引入 import，做了公用方法的抽离 |
| @babel/preset-env | 根据目标环境选择不支持的新特性来转译(stage-0是最全的特性转换，可以使用target指定兼容条件) |
| @babel/preset-react | react相关插件预设 |
| babel-loader | webpack loader |
| cross-env | windows下执行命令加入环境变量 |
| rimraf | windows下的rm |


## webpack配置

```javascript
const path = require("path");
const htmlWebpackPlugin = require("html-webpack-plugin");

const webpack = require('webpack')
const isDev = process.env.NODE_ENV === "development";
const config = {
  entry: "./src/index.js",
  output: {
    filename: "main.js",
    path: path.join(__dirname, "dist"),
    publicPath: "/public/",
  },

  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.js|jsx$/,
        use: [          
          { loader: "babel-loader" },         
        ],
      },
     
    ],
  },
  mode: "development", // 开发环境打包

  plugins: [
    // 以模板生成入口页面
    new htmlWebpackPlugin({
      template: path.join(__dirname, "client/index.html"),
    })  
  ],
};
// 开发环境配置
if (isDev) {
  config.entry = ["react-hot-loader/patch", "./src/index.js"];
  config.devtool = "#cheap-module-eval-source-map"; // source-map 方便排错
  config.devServer = {
    host: "127.0.0.1",
    port: "8888",
     //告诉服务器从哪个目录中提供内容
    contentBase: path.join(__dirname, "dist"),
    //启用 webpack 的模块热替换特性
    hot: true,
    //当出现编译器错误或警告时，就在网页上显示一层黑色的背景层和错误信息
    overlay: {
      errors: true,
    },
    //webpack-dev-server打包的内容是放在内存中的，这些打包后的资源对外的的根目录就是publicPath，换句话说，这里我们设置的是打包后资源存放的位置
    publicPath: "/public/",
    // 404时页面跳转
    historyApiFallback: {
      index: "/public/index.html",
    },
  };
  config.plugins = [
    ...config.plugins,
    new webpack.HotModuleReplacementPlugin(),
  ];
}
module.exports = config;
```
`webpack` 在执行 `babel-loader` 时 会去check是否存在 `.babelrc` ，存在就去读取里面的配置

### .babelrc

```json

{
  "presets": [ "@babel/preset-env", "@babel/preset-react" ],
  "plugins": [
      "react-hot-loader/babel",
      "@babel/plugin-proposal-class-properties"
  ]
}
```

## 配置react-hot-loader

### 为啥需要这个库呢

按道理， `webpack-dev-server` 已经满足热更新了，为啥使用它？
不卖关子， `webpack-dev-server` 是检测到变化(socket) 去重新打包 ，这样会存在两个问题

1. 当项目代码量非常大的时候，构建缓慢
2. 每次重新构建 之前react里的属性数据会全部丢失

为了解决这两个问题，故使用 `react-hot-loader` 实现局部刷新

### 配置

1. 启动webpack热模块替换（详情见上文webpack配置）
2. 在babel中启用`react-hot-loader`插件（详情见上文babel配置）
3. webpack入口js中引入
```javascript
// 引入HOC组件 给内部组件加上热更新feature
import { AppContainer } from 'react-hot-loader';
// .............
 import Home from './components/Home.jsx'
const render = (App) => {
  console.log('=======begin render======')
	ReactDom.render(
		<AppContainer>
			<App />
		</AppContainer>,
	document.getElementById('root')
	)
}
render(Home)
// 代码变更 进行局部渲染
if (module.hot) { 
  module.hot.accept('./components/Home.jsx', () => { 
      render(require('./components/Home.jsx').default)
  })
}
```


## 项目地址

[ReactCustomWebpack](https://github.com/godlikedeveloper/ReactCustomWebpack)
