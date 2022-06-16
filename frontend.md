# frontend

## JavaScript module

### CommonJS

仅用于服务端，使用`require('./module_name')`，需要使用相对路径，没有路径的直接引入核心模块

使用`module.exports = {}`，暴露模块的方法和属性

### AMD

异步加载，一般用在浏览器

```js
/** 网页中引入require.js及main.js **/
<script src="js/require.js" data-main="js/main"></script>

/** main.js 入口文件/主模块 **/
// 首先用config()指定各模块路径和引用名
require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",
    "math": "math",
  }
});
// 执行基本操作
require(["jquery","math"],function($, math){
  // some code here
});
// 定义math.js模块
define(function () {
    var basicNum = 0;
    var add = function (x, y) {
        return x + y;
    };
    return {
        add: add,
        basicNum :basicNum
    };
});
```

### CMD

依赖的模块用到时加载

```js
// 定义模块 math.js
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
});

// 加载模块
seajs.use(['math.js'], function(math){
    var sum = math.add(1+2);
});
```

### UMD

智能判断并使用AMD，CommonJS还是浏览器环境

```js
((root, factory) => {
  if (typeof define === 'function' && define.amd) {
    //AMD
    define(['jquery'], factory);
  } else if (typeof exports === 'object') {
    //CommonJS
    var $ = requie('jquery');
    module.exports = factory($);
  } else {
    //都不是，浏览器全局定义
    root.math = factory(root.jQuery);
  }
})(this, ($) => {
  //do something...
});
```

### ES6

使用import引入，export导出，导出的是模块的引用

