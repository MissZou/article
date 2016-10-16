## 前言
来来来，我也终于开始用markdown语法写东西了呢！  
今天来聊聊webpack和gulp。这俩东西其实貌似已经火了很久了，但是我一直处于一知半解的阶段，今天好好来梳理一下，最后会附上两个我自己认为比较好的模板~
## 目录
- 为什么要用webpack
- 为什么要用gulp
- 两者的区别
- 模板
## 为什么要用webpack
官网对webpack的定义是MODULE BUNDLER，他的目的就是把有依赖关系的各种文件打包成一系列的静态资源。 

此外，现在有很多前端框架都有自己的文件后缀，比如.vue，这时浏览器是不能直接认识这些文件的，于是我们的webpack就用一系列的loader来处理这些文件，使它们最终变成浏览器可识别的内容。  
### loader

```
loaders: [
    {
        test: /\.css$/,
        loaders: ['style', 'css'],
        include: APP_PATH
    }
]
```
或者
```
{ 
    test: /\.css$/, 
    loader: 'style!css!autoprefixer'
}
```
注意loader的执行顺序是从右往左的哟~

这时我们就可以在js文件里引用
```
require('./main.css');
```
来执行我们的css代码了。

顺便介绍一个挺火的处理图像的loader:```url-loader```
```
{
    test: /\.(png|jpg)$/,
    loader: 'url?limit=40000'
}
```
注意后面那个limit的参数，当你图片大小小于这个限制的时候，会自动启用base64编码图片。
### 添加ES6的支持
之前我年少无知，以为ES6既然还没有被浏览器支持那我就先不学了= = 后来知道了有一个叫```Babel```的东西，可以将我们的es6语法转换成浏览器识别的js语句，这时我才知道，原来现在学es6是可以马上使用的了= =

首先装一下loader：
```
npm install babel-loader babel-preset-es2015 --save-dev
```
然后配置一下loader：
```
{
    test: /\.js?$/,
    loader: 'babel',
    include: APP_PATH,
    query: {
      presets: ['es2015']
    }
}

```
好了，现在我们就可以用es6语法来编写代码啦~
### 启用source-map
现在的代码是合并以后的代码，不利于排错和定位，只需要在config中添加
```
devtool: 'eval-source-map'
```
这样出错以后就会采用source-map的形式直接显示你出错代码的位置。

如果想给sass这样的文件开启source-map也很简单：
```
{
    test: /\.scss$/,
    loaders: ['style', 'css?sourceMap', 'sass?sourceMap'],
    include: APP_PATH
},
```
### 配置webpack-dev-server
安装webpack-dev-server
```
npm install webpack-dev-server --save-dev
```
安装完毕后 在config中添加配置
```
devServer: {
    historyApiFallback: true,
    hot: true,
    inline: true,
    progress: true,
},
```
然后再package.json里面配置一下运行的命令,npm支持自定义一些命令
```
"scripts": {
  "start": "webpack-dev-server --hot --inline"
},
```
好了，npm run start 一下，就可以在修改代码后自动刷新浏览器啦~

### 一个工具
webpack提供一个插件 把一个全局变量插入到所有的代码中，在config.js里面配置
```
 plugins: [
    //provide $, jQuery and window.jQuery to every script
    new webpack.ProvidePlugin({
      $: "jquery",
      jQuery: "jquery",
      "window.jQuery": "jquery"
    })
]
```
### 部署上线
刚才说的各种情况都是在开发时候的情况，那么假如项目已经开发完了，需要部署上线了。我们应该新创建一个单独的config文件，因为部署上线使用webpack的时候我们不需要一些dev-tools,dev-server和jshint校验等。

复制我们现有的config文件，命名为 webpack.production.config.js，将里面关于 devServer等和开发有关的东西删掉。

在package.json中添加一个命令。

```
"scripts": {
  "start": "webpack-dev-server --hot --inline",
  "build": "webpack --progress --profile --colors --config webpack.production.config.js"
},
```
### 参考文章
> https://zhuanlan.zhihu.com/p/20367175
> https://vikingmute.gitbooks.io/webpack-for-fools/content/entries/chapter-2.html