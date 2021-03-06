# Vue 学习记录


## Webpack
使用可以参考下面的两篇文章：

[使用webpack](http://www.cnblogs.com/vajoy/p/4650467.html)

[webpack 教程](https://www.zfanw.com/blog/webpack-tutorial.html)

### 和 gulp 集成
在 `gulpfile.js` 里编写 webpack 的配置即可。例如下面的形式：
```js
var gulp = require("gulp");
var webpack = require("webpack");

gulp.task("webpack", function (callback) {
    //webpack配置文件
    var config = {
        watch: true,
        entry: {

            index: __dirname + '/src/js/index.js'
        },
        output: {
            path: __dirname + '/dist/js',
            filename: '[name].js'
        }
        //........
    };
    webpack(config, function (err, stats) {
        console.log(stats.toString());
    });
});

gulp.task('default', [ 'webpack']);
```
在配置里加上 `watch: true`，就可以使 webpack 监听文件的变化。这里使用 `browserSync` 来作为 web 服务器，为了实现文件改动自动刷新网页，只需要使用 gulp 监听 webpack 生成的文件，然后调用 `browserSync.reload` 即可，下面是配置：
```js
gulp.task("watch", function () {
    gulp.watch("./**/*.html", browserSync.reload);
    gulp.watch("dist/**/*.js", browserSync.reload);
    gulp.watch("dist/**/*.css", browserSync.reload);

});
gulp.task('default', ['browserSync', 'watch', 'webpack']);
```

### 提取 css 文件

使用 [`extract-text-webpack-plugin`](https://github.com/webpack-contrib/extract-text-webpack-plugin) 插件，不过安装使需要注意 webpack 1 和 2的安装方式不同：
 ```
 # for webpack 1
 npm install --save-dev extract-text-webpack-plugin
 # for webpack 2
 npm install --save-dev extract-text-webpack-plugin@beta
 ```
webpack 2 配置的时候也略有不同：
```js
{
    plugins: [commonsPlugin, new ExtractTextPlugin("../css/[name].css")],

    module: {
        loaders: [{
            test: /\.css$/,
            loader: ExtractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader"
            })
        },
        {
            test: /\.less$/,
            loader: ExtractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader!less-loader"
            })
        }
    },
}
```


### 压缩文件
压缩js，使用 `UglifyJsPlugin` 即可，压缩 css 可以使用 `optimize-css-assets-webpack-plugin`，下面是配置代码：
```js
var uglifyJsPlugin = new webpack.optimize.UglifyJsPlugin({
    compress: {
        warnings: false
    }
});
var OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
//.....
var config = {
    plugins: [

    uglifyJsPlugin, new OptimizeCssAssetsPlugin({
        assetNameRegExp: /\.css$/g,
        cssProcessor: require('cssnano'),
        cssProcessorOptions: {
            discardComments: {
                removeAll: true
            }
        },
        canPrint: true
    })],
}
```

## ES6 语法
### export
`export default` 默认导出，不需要指定变量名，默认是文件名。

## Vue
### Vue文件中导入外部文件
如果在 Vue 文件中导入了样式文件，如果想要提取出来，需要配置 Vue 的loader，具体方式还没查

### 缩写
Vue.js 为两个最为常用的指令提供了特别的缩写：

#### `v-bind` 缩写
```html
<!-- 完整语法 -->
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
```

#### `v-on` 缩写
```html
<!-- 完整语法 -->
<a v-on:click="doSomething"></a>
<!-- 缩写 -->
<a @click="doSomething"></a>
```

### 向单文件组件中传输
* 使用 `v-bind:prop` 指令即可，可以简写为 `:prop`。首先在 index.js 里定义数据：
```js
var vm = new Vue({
    el: '#app',
    data: {
        projects: [{id:1}, {id:2}]
    },
});
```
* 在组件文件中定义要接受数据的变量名，并使用接受的数据
```js
export default {
    props: ["project"]
}
```
```html
 <p>{{project.id}}</p>
```
* 在 html 添加组件标签，并向组件传入数据对象，这里使用 for 指令创建了一个循环
```html
<div id="app">
    <project v-for="project in projects" :project="project"></project>
</div>
```