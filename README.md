# 如何搭建一个基于 `dva+webpack+antd` 的工程

#### `自己以前初用dva的时候整理出来的一些东西，今天突然倒腾出来啦～现在分享给大家，现在这个dva是用的<1.0版本的。后期我会更新一个dva 2.0版本的`

## 创建工程

安装 `dva-cli` 和 `roadhog`（自行百度）

### 项目创建`以下介绍都是基于dva < 1.0 版本的。 请注意版本`

```bash
$ dva new project
```

### 启动项目
```bash
$ cd project
$ npm install
$ npm start
```
浏览器访问localhost:8000/ 看到小丑界面～

### 目录结构

```bash
.
├── dist                          # 编译结果
├── mock                       # 虚拟数据
├── public                      # 文件夹内所有文件将直接复制到根目录
├── src                          # Source directory
    ├── assets                 # Store images, icons, ...
    ├── components        # UI components
    ├── index.css             # CSS for entry file
    ├── index.html           # HTML for entry file
    ├── index.js               # Enry file
  ├── models               # Dva models
    ├── router.js             # Router configuration
    ├── routes                # Route components
    ├── services             # Used for communicate with server
    └── utils                   # Utils
        └── request.js      # A util wrapped dva/fetch
├── .editorconfig          #
├── .eslintrc                 # Eslint config
├── .gitignore              #
├── .roadhogrc            # Roadhog config
└── package.json         #
```

### 配置文件

由于 `dva` 和 `roadhog` 的版本还没有稳定，用命令生成的新目录不能直接用于开发，需要修改配置文件。

#### package.json

安装依赖并添加至 `package.json`

```bash
# lodash
$ npm i --save-dev lodash
# babel-plugin
$ npm i --save-dev babel-plugin-import babel-plugin-lodash
# webpack-plugin
$ npm i --save-dev lodash-webpack-plugin html-webpack-plugin
# shim
$ npm i --save-dev es5-shim console-polyfill
```
复制 `es5-shim.min.js` `es5-sham.min.js` `console-polyfill/index.js` 文件到 `public` 文件夹
`console-polyfill/index.js` 改名为 `console-polyfill.js`


####  新增 `webpack.config.js` 文件

当前版本 `roadhog` 功能不完善，不过幸好支持读取文件 `webpack.config.js` 修改配置。

```js
import webpack from 'webpack';
import { isRegExp } from 'lodash';
import HtmlWebpackPlugin from 'html-webpack-plugin';
import ExtractTextPlugin from 'extract-text-webpack-plugin';
import LodashModuleReplacementPlugin from 'lodash-webpack-plugin';

export default ( webpackConfig, env ) => {

  const loaders = webpackConfig.module.loaders;

  // 根目录使用相对地址
  webpackConfig.output.publicPath = '';

  // 不打包 moment 的语言包
  const noParse = webpackConfig.module.noParse;
  if ( Array.isArray( noParse ) ) {
    noParse.push( /moment.js/ );
  }
  else if ( noParse ) {
    webpackConfig.module.noParse = [ noParse, /moment.js/ ];
  }
  else {
    webpackConfig.module.noParse = [ /moment.js/ ];
  }

  // lodash
  webpackConfig.babel.plugins.push( 'lodash' );
  webpackConfig.plugins.push( new LodashModuleReplacementPlugin() );

  // 生成 HTML
  webpackConfig.module.loaders = loaders.filter(
    loader => isRegExp( loader.test ) && loader.test.toString() !== '/\\.html$/'
  );
  webpackConfig.plugins.push(
    new HtmlWebpackPlugin( {
      // favicon: './src/logo/logo.ico',
      template: './src/index.html',
      filename: 'index.html',
      inject: true
    } )
  );

  // 打包配置
  if ( env === 'production' ) {
    // 字体打包
    loaders.unshift( {
      test: /\.(woff|woff2|ttf|eot)(\?v=\d+\.\d+\.\d+)?$/,
      loader: 'file',
      query: {
        name: 'static/[name].[hash:8].[ext]'
      }
    } );

    // 所有输出文件添加 hash
    webpackConfig.output.filename = '[name].[chunkhash:6].js';
    webpackConfig.output.chunkFilename = '[name].[chunkhash:6].js';

    // css common 添加 hash
    webpackConfig.plugins.forEach( ( plugin, index, plugins ) => {
      if ( plugin instanceof ExtractTextPlugin ) {
        plugins[ index ] = new ExtractTextPlugin( '[name].[chunkhash:6].css' );
      }
      else if ( plugin instanceof webpack.optimize.CommonsChunkPlugin ) {
        plugins[ index ] = new webpack.optimize.CommonsChunkPlugin(
          'common',
          'common.[chunkhash:6].js'
        );
      }
    } );

  }

  return webpackConfig;
};
```

> 详细的 `webpack` 配置参考 `roadhog` 的 [webpack配置]
>
> 注：官方不推荐此方法修改配置，因为不便于以后的完美升级。
> 由于官方的配置不能完全满足我们的需求，因此 `webpack.config.js` 配置还是必要的。
> 当根目录存在 `webpack.config.js` 文件时，运行服务会有警告。

### .roadhogrc

> 详细文档参考 [https://github.com/sorrycc/roadhog](https://github.com/sorrycc/roadhog)
> 
> 不喜欢 `json` 格式的配置，可以改名为 `.roadhogrc.js` 使用 `javascript` 语法
> 支持 `es6` 语法

```diff
  {
+   "multipage": true,
-   "entry": "src/index.js",
+   "entry": ["src/common.js", "src/index.js"],
    ...
  }
```

### .eslintrc

编辑器语法检查配置。

> 统一团队代码风格，调试方便，提高开发效率

```diff
  {
    ...
-   "extends": "airbnb",
+   "extends": [
+     "airbnb",
+     "./eslint.config.js"
+   ],
    ...
  }
```

### 新增 `eslint.config.js` 文件


> 详细文档请访问 [http://eslint.org/docs/user-guide/configuring](http://eslint.org/docs/user-guide/configuring)


## Hello World

1. `src/index.html`
    
  ```html
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
      <meta charset="utf-8">
      <meta name="renderer" content="webkit">
      <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
      <meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0">
      <title></title>
      <!-- /path/to/ 或者 / -->
      <base id="basename" href="/" />
      <!--[if lt IE 10]>
      <script type="text/javascript" src="es5-shim.min.js"></script>
      <script type="text/javascript" src="es5-sham.min.js"></script>
      <script type="text/javascript" src="console-polyfill.js"></script>
      <![endif]-->
    </head>
    <body>
    <div id="root"></div>
    </body>
    </html>
  ```

2. `src/router.js`

  ```js
    import React from 'react';
    import { Router, Route } from 'dva/router';
    import IndexPage from './routes/IndexPage';
    
    function RouterConfig({ history }) {
      return (
        <Router history={history}>
          <Route path="/" component={IndexPage} />
        </Router>
      );
    }
    
    export default RouterConfig;
  ```

3. `src/routes/indexPage.jsx`

  ```js
    import React from 'react';
    import './IndexPage.less';
    
    function IndexPage() {
      return (
        <div style={{ textAlign: 'center' }}>
          <h1>Hello World</h1>
        </div>
      );
    }
    
    IndexPage.propTypes = {};
    
    export default IndexPage;
  ```
  
4. `src/routes/indexPage.less`

  ```less
    body {
      overflow: hidden;
    }
  ```
  
5. `src/index.js`

  ```js
    import dva from 'dva';
    import './index.css';
    
    // 1. Initialize
    const app = dva();
    
    // 2. Plugins
    // app.use({});
    
    // 3. Model
    // app.model(require('./models/example'));
    
    // 4. Router
    app.router(require('./router'));
    
    // 5. Start
    app.start('#root');
  ```
  
6. `src/common.js` 打包公共库

  ```js
    import 'dva';
    import 'react';
    import 'react-dom';
    import moment from 'moment';
    import 'moment/locale/zh-cn';
    
    // 全局设置 locale
    moment.locale( 'zh-cn' );
  ```

7. 删除多与文件，启动开发服务器

    ```bash
    $ npm start
    ```

打开浏览器访问 http://localhost:8000/ 可以看到 `Hello World`

## 添加 `antd` 支持

安装依赖并添加到`package.json`中
```bash
$ antd
$ npm install --save-dev antd
```

#### 修改.roadhogrc
添加 `antd`
```diff
  {
    ...
    "env": {
      "development": {
        "extraBabelPlugins": [
          ...
+         [ "import", { "libraryName": "antd", "style": true } ]
        ]
      },
      "production": {
        "extraBabelPlugins": [
          ...
+         [ "import", { "libraryName": "antd", "style": true } ]
        ]
      }
    },
    ...
  }
```
# 在项目中使用 Proxy
参考[webpack-dev-server#proxy](https://webpack.github.io/docs/webpack-dev-server.html#proxy)

修改配置文件 `.roadhogrc`，**改名为** `.roadhogrc.js`，内容改成 **Javascript** 格式

```diff
+ var proxy = require( './proxy.config.js' );

- {
+ module.exports = {
    ...
+   "proxy": proxy
    ...
- }
+ };
```

新增文件 `proxy.config.js`
```js
module.exports = {
  '/api': {
    target: 'http://10.8.6.27:3000/',
    changeOrigin: true,
    pathRewrite: { '^/api' : '' }
  }
}
```


到上一个成型的项目都有啦  关于后续开发大学可以参考crunchysoul 的[dva.js 知识导图](https://github.com/dvajs/dva-knowledgemap)

#### 后续
```
 以上是基于dva < 1.0.0， react  15.X，react-router 2.X， webpack 2.X 所搭建成
后续会跟大家分享一遍基于dva 2.X，react  16.X，react-router 4.X， webpack 3.X 的项目搭建
后续也会跟大家分享一遍基于 webpack+react-mobx 的架构心得
```