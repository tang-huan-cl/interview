# 前端模块化

前端模块化就是复杂的文件编程一个一个独立的模块，比如 JS 文件等等，分成独立的 模块有利于重用（复用性）和维护（版本迭代），这样会引来模块之间相互依赖的问题， 所以有了 commonJS 规范，AMD，CMD 规范等等，以及用于 JS 打包（编译等处理）的 工具 webpack

## CommonJS：

开始于服务器端的模块化，同步定义的模块化，每个模块都是一个单独的 作用域，模块输出，modules.exports，模块加载 require()引入模块

## AMD：

中文名异步模块定义的意思
requireJS 实现了 AMD 规范，主要用于解决下述两个问题。

1. 多个文件有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
2. 加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应的时间越长。

### 语法：

requireJS 定义了一个函数 define，它是全局变量，用来定义模块。
requireJS 的例子：

```js
//定义模块
define(["dependency"], function () {
  var name = "Byron";
  function printName() {
    console.log(name);
  }
  return { printName: printName };
});

//加载模块
require(["myModule"], function (my) {
  my.printName();
});
```

RequireJS 定义了一个函数 define,它是全局变量，用来定义模块：

```js
define(id?dependencies?,factory)
```

在页面上使用模块加载函数：

```js
require([dependencies],factory)；
```

总结 AMD 规范：require（）函数在加载依赖函数的时候是异步加载的，这样浏览器不 会失去响应，它指定的回调函数，只有前面的模块加载成功，才会去执行。 因为网页在加载 JS 的时候会停止渲染，因此我们可以通过异步的方式去加载 JS,而如果 需要依赖某些，也是异步去依赖，依赖后再执行某些方法。
