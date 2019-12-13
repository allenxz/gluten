## 1. loader和plugin的不同

* loader是使webpack拥有加载和解析非js文件的能力，如：

  * css-loader：加载CSS，支持模块化，压缩，文件导入等能力
  * style-loader：把CSS代码注入到js中
  * sass-loader、less-loader

* plugin目的在于解决loader无法实现的其他事，拓展webpack的能力，使得webpack更加灵活，功能更加强大。如：

   * html-webpack-plugin：为 html 文件中引入的外部资源如 script、link动态添加每次 compile 后的 hash，防止引用缓存的外部文件问题
   * HotModuleReplacementPlugin：热更新替换，在不刷新重载页面的情况下更换编辑修改后的代码
   * clean-webpack-plugin：删除打包文件



## 2. webpack的构建流程

* 初始化参数，从配置文件和shell语句中读到的参数合并，得到最后的参数
* 开始编译：用合并得到的参数初始化complier对象，加载是所有配置的插件，执行run方法开始编译
* 确定入口，通过entry找到入口文件
* 编译模块，从入口文件出发，调用所有配置的loader对模块进行解析翻译，在找到该模块依赖的模块进行处理
* 完成模块编译，得到每个模块被翻译之后的最终的内容和依赖关系
* 输出资源，根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk，在把每个chunk转换成一个单独的文件加载到输出列表
* 输出完成，确定输出的路径和文件名，把内容写到文件系统中



## 3. webpack热加载执行的原理

构建 bundle 的时候，加入一段 HMR runtime 的 js 和一段和服务沟通的 js 。文件修改会触发 webpack 重新构建，服务器通过向浏览器发送更新消息，浏览器通过 jsonp 拉取更新的模块文件，jsonp 回调触发模块热替换逻辑。



## 4. 如何使用webpack来优化前端性能

* 压缩代码。uglifyJsPlugin 压缩js代码， mini-css-extract-plugin 压缩css代码
* 利用CDN加速，将引用的静态资源修改为CDN上对应的路径，可以利用webpack对于output参数和loader的publicpath参数来修改资源路径
* 删除死代码（tree shaking），css需要使用Purify-CSS
* 提取公共代码。webpack4移除了CommonsChunkPlugin (提取公共代码)，用optimization.splitChunks和optimization.runtimeChunk来代替



## 5. 什么是bundle、什么是chunk、什么是module

* chunk：webpack在进行模块的依赖分析时，代码分割出来的代码块
* bundle：由webpack打包出来的文件，一个bundle中包含了多个chunk
* module：开发时一个个独立的模块



## 6. 说说webpack的作用

webpack是一个打包模块化JavaScript的工具，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（SCSS，TypeScript等），并将其打包为合适的格式以供浏览器使用。



## 7. 什么是source map、webpack中的source map

简单的说source-map是一个信息文件，里面储存着位置的信息。经过转换后的代码的每一个位置都对应其转换前的位置。这样，代码出错的时候能够更加清除的定位错误的发生位置。



在webpack中使用：

```js
const path = require('path');

module.exports = {
  devtool: '参数', 
  entry: './src/index.js',  // 入口文件
  output: {
    filename: 'bundle.js',  // 文件名
    path: path.resolve(__dirname, 'dist')  // 文件夹
  }
}
```

* 参数：

  1. source-map：生成map格式文件，里面包含映射关系的代码
  2. inline-source-map：不会生成map格式的代码，包含映射关系的代码会放在打包后生成的代码中
  3. inline-cheap-source-map ：只将错误定位到行，不定位到列；映射业务代码，不映射loader和第三方库。提高打包构建的速度。
  4. inline-cheap-module-source-map：module会映射loader和第三方库
  5. eval：用eval的方式生成映射关系代码，效率和性能最佳。但是当代码复杂时，提示信息可能不精确。

推荐：

开发环境下：`devtool: 'cheap-module-eval-source-map'`

生产环境下：`devtool: 'cheap-module-source-map'`









> 参考：
>
> * https://www.jianshu.com/p/e80d38661358
> * <https://blog.csdn.net/jiaojsun/article/details/93852086>
> * <https://www.leiue.com/what-is-webpack>
> * <http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html>
> * <https://www.jianshu.com/p/f20d4ceb8827>



