title: Node.js 中的 this
date: 2015-09-28 11:24:15
tags: Node.js
categories: Node.js
---

在 Node.js 环境中和浏览器环境中，`this` 绝大多时候的表现是一样的。唯一不同的就是在全局运行上下文中。
因为在全局运行上下文中，`this` 指向**全局对象**，无论是否在严格模式下。

浏览器环境中，全局对象是`window`。而 Node.js 环境中，全局对象是 `global`。
所以，造成了两者之间的不同。

另外，在 Node.js 环境中，也还有一点不同。

当在 `RELP` 中时
```
$ node
> this === global
true
> this === module.exports
false
```
`this` 是指向 `global` 的，跟预期的一致。

但如果是用 `node xx.js` 的方式运行下面的代码
```
console.log('Func out:', this === module.exports); // true
console.log('Func out:', this === global); // false


function func() {
    console.log('Func in:', this === module.exports); // false
    console.log('Func in:', this === global); // true
}
func();
```
在函数外面的 `this` 不是指向 `global`，而是指向 `module.exports`。

因为通过 `node xx.js` 方式运行，其实是先将 `xx.js` 以文件模块的形式加载，然后再编译，执行。

主要的代码逻辑在[这里](https://github.com/nodejs/node/blob/master/lib/module.js#L434)
```
// Run the file contents in the correct scope or sandbox. Expose
// the correct helper variables (require, module, exports) to
// the file.
// Returns exception, if any.
Module.prototype._compile = function(content, filename) {
  var self = this;

  ...

  // create wrapper function
  var wrapper = Module.wrap(content);

  var compiledWrapper = runInThisContext(wrapper, { filename: filename });

  ...

  var args = [self.exports, require, self, filename, dirname];
  return compiledWrapper.apply(self.exports, args);
};

```

`content` 是源代码的内容，经过

```
var wrapper = Module.wrap(content);

var compiledWrapper = runInThisContext(wrapper, { filename: filename });
```

处理后，代码会变成

```
(function (exports, require, module, __filename, __dirname) { console.log('Func out:', this === module.exports); // true
console.log('Func out:', this === global); // false


function func() {
    console.log('Func in:', this === module.exports); // false
    console.log('Func in:', this === global); // true
}
func();

});
```

然后调用执行
```
compiledWrapper.apply(self.exports, args);
```

通过 `apply` 将上下文设置为 `self.exports`，而 `self` 指向 `Module.prototype`（也就是 `Module` 的实例对象 `module`），所以，`this === module.exports`。


**参考链接**

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this
http://www.infoq.com/cn/articles/nodejs-module-mechanism
https://github.com/nodejs/node/blob/master/lib/module.js
