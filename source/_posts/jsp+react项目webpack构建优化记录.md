---
title: jsp + react项目webpack构建优化记录
date: 2018-03-08 16:23:23
tags: 学习
categories: 构建
---
# 前言
最近接手了一个项目，由于历史原因，项目是由jsp和react混合的，其中react部分通过webpack构建，打包生成js文件引入jsp页面。

# 优化
当时构建的时间如图所示，平均为40s左右。
{% qnimg 'article/jsp + react项目webpack构建优化记录/before.png' title:最初的构建 %}
要提升打包的速度，需要进行bundle分析，可以通过以下命令获得打包的输出结果：
```javascript
$ webpack --profile --json > stats.json
```
> 刚开始遇到了一个报错，***Output filename not configured***，排查之后发现是webpack.config.js文件中的output属性使用了chunkFilename造成的，改回filename的设置即可
```javascript
    output: {
      filename: '[name].bundle.js',
      chunkFilename: '[name].bundle.js', // 使用这个属性会造成stats.json生成失败，具体原因未知
      path: path.resolve(__dirname, 'dist')
    }
```
然后将得到的结果放到webpack的[官方分析工具](https://github.com/webpack/analyse)进行分析。
{% qnimg 'article/jsp + react项目webpack构建优化记录/multiCompile.png' title:多入口构建 %}
当我把项目的stats.json输出文件上传之后，看到了上图，正常来说上传之后会直接显示bundle分析，但是此时显示多个compile选择，可见这份json文件中包含了多次构建，于是我去查看了webpack的config文件，看到了下面的代码：
```javascript
function getConfg(resourse, env) {

    var resources = resourse,
        cfgArr = [];

    for (key in resources) {
        if(Object.prototype.hasOwnProperty.call(resources, key)){
            var obj = getBaseCfg(env);
            obj.entry = {};
            obj.entry[path.basename(key, '.jsx')] = key;
            obj.output = {};
            obj.output.path = resources[key];
            obj.output.filename = '[name].js';
            cfgArr.push(obj);
        }
    }

    return cfgArr;
}
```
可以看到返回的cfgArr是个数组，因此执行了多次webpack构建，降低了性能，可以通过多入口的方式，将这些构建合并成一个，来提高构建速度，修改如下：
```javascript
function getConfg(resourse, env) {

    var resources = resourse,
        cfg = getBaseCfg(env);
    cfg.entry = {};
    cfg.output = {
        path: path.resolve(__dirname, buildDir),
        filename: '[name].js'
    };

    resources.forEach(function (filePath) {
        cfg.entry[path.basename(filePath, '.jsx')] = filePath;
    });

    return cfg;
}
```
此外，由于这些模块之间存在公共部分，可以通过CommonsChunkPlugin提取出来，来避免打包时的代码冗余。
```javascript
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: "common"
        })
    ]
```
以下是优化后的构建时间，缩短了将近一半。
{% qnimg 'article/jsp + react项目webpack构建优化记录/afterCombine.png' %}

好，接下来继续进行分析，修改之后，重新生成一下stats.json，这次再上传到官方的分析工具之后，这次直接就展示了输出结果，我们先点击上方导航栏的hints，就能看到工具提供的优化建议。
{% qnimg 'article/jsp + react项目webpack构建优化记录/hints.png' %}
如图，看到的是可以提取的公共模块，其中有个`address.json`文件，体积高达1mb，被两个module所引用，造成两个module体积都较大，可以考虑打包成单独的chunk，动态加载。
按照建议，参考[官方文档](https://doc.webpack-china.org/api/module-methods/#import-)使用import方式动态加载。
> 工具建议中所用的ensure方法已被import取代
注意：此时需要修改webpack.config.js文件中的output属性，把filename改成chunkFilename，否则会出现分离出来的chunk文件名是数字的情况。

分离出address文件之后，可以看到原本引用了该文件的applyInfo和basicInfoList体积都减小了很多，不过经测试对构建时间并没有什么影响。而且因为使用到这个文件的两个模块其实也是一加载的时候就需要使用address，并不存在需要时再加载，因此对页面的性能也没什么提高……
{% qnimg 'article/jsp + react项目webpack构建优化记录/beforeSplit.png' %}
{% qnimg 'article/jsp + react项目webpack构建优化记录/afterSplit.png' %}


总之，继续来看下面的hints吧。
{% qnimg 'article/jsp + react项目webpack构建优化记录/longModuleBuildChains.png' %}
这个hint大意是有些模块的依赖路径太长，建议提前告知webpack。[Prefetch](https://webpack.js.org/plugins/prefetch-plugin/)是webpack的一个插件，但是官方文档的介绍比较简略，看不出这个插件是如何作用提高性能的。我按照文档介绍的方式尝试prefetch了几个模块，但是构建时间并没有减少，也并没有新的chunk输出。

```javascript
    plugins: [
        new webpack.PrefetchPlugin('./node_modules/dom-align/es/index.js'),
        new webpack.PrefetchPlugin('./node_modules/lodash/_mapCacheClear.js'),
        new webpack.PrefetchPlugin('./node_modules/lodash/_MapCache.js'),
        new webpack.PrefetchPlugin('./node_modules/async-validator/es/rule/index.js')
    ]
```

在网上找到webpack作者sokra的一段解释，完整的issues见[这里](https://github.com/webpack/webpack/issues/1566)
{% qnimg 'article/jsp + react项目webpack构建优化记录/prefetchPlugin.png' %}
大意是过长的chains可能导致webpack为了等待I/O而导致性能受影响，而prefetch可以避免这种情况发生。结合前面加了prefetch之后构建时间并没有减少的情况，我觉得prefetch应该只有在I/O成为性能瓶颈的情况下，才会提升构建性能。
此外，在另一个[issuse](https://github.com/webpack/webpack.js.org/issues/126)里，webpack成员TheLarkInn提到prefetch现在在webpack中已经是默认配置了，因此没必要特意做配置了。
{% qnimg 'article/jsp + react项目webpack构建优化记录/prefetchDefaultUsed.png' %}
