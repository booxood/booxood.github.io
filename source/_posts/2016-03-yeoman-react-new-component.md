title: Yeoman 创建一个 React Component 生成器
date: 2016-03-28 16:47:22
tags: [Node.js, Yeoman, React]
categories: Tools
---

> 在做 React 开发时，每次新建一个组件都要需要创建一个 Xxx 目录，然后在目录中创建两个文件 Xxx.jsx 和 Xxx.css，然后引入巴拉巴拉。。。然后。。。

Yeoman 是一个脚手架工具，可以帮助我们创建目录，创建文件，执行 Grunt/Gulp，安装 Npm/Bower 依赖等等。我们用它来创建一个 generator-react-new-component。


### 准备工具

在编写生成器之前，先安装 `yo` 和 `generator-generator`。
```shell
npm install -g yo generator-generator
```

### 了解工具

通常一个 `yo` 生成器的目录结果是
```
├───package.json
├───app/
│   └───index.js
└───router/
    └───index.js
```

或者希望保持生成器根目录的整洁，也可以这样
```
├───package.json
└───generators/
    ├───app/
    │   └───index.js
    └───router/
        └───index.js
```

我们可以通过 `yo` 提供的脚手架来生成基本的目录结构，运行
```shell
yo generator
```
按照交互提示输入，最终会得到
```shell
? Your generator name generator-xxx
? Description xxx
? Project homepage url xxx
? Author's Name xxx
? Author's Email xxx@mail.com
? Author's Homepage
? Package keywords (comma to split) xxx,xx
? Send coverage reports to coveralls Yes
? GitHub username or organization xxx
? Which license do you want to use? MIT
   create package.json
   create README.md
   create .editorconfig
   create .gitattributes
   create .gitignore
   create generators/app/index.js
   create generators/app/templates/dummyfile.txt
   create test/app.js
   create .travis.yml
   create gulpfile.js
   create LICENSE
```

其中，我们主要关注 `generators/app/index.js`
```javascript
'use strict';
var yeoman = require('yeoman-generator');
var chalk = require('chalk');
var yosay = require('yosay');

module.exports = yeoman.generators.Base.extend({
  prompting: function () {
    var done = this.async();

    // Have Yeoman greet the user.
    this.log(yosay(
      'Welcome to the amazing ' + chalk.red('generator-xxx') + ' generator!'
    ));

    var prompts = [{
      type: 'confirm',
      name: 'someOption',
      message: 'Would you like to enable this option?',
      default: true
    }];

    this.prompt(prompts, function (props) {
      this.props = props;
      // To access props later use this.props.someOption;

      done();
    }.bind(this));
  },

  writing: function () {
    this.fs.copy(
      this.templatePath('dummyfile.txt'),
      this.destinationPath('dummyfile.txt')
    );
  },

  install: function () {
    this.installDependencies();
  }
});

```

每一个生成器都是 `yeoman.generators.Base.extend` 的扩展，你可以扩展一些处理函数。其中 `constructor` 是构造函数，只会执行一次，有些 `yeoman.generators` 提供的函数只能在这里执行
```javascript
module.exports = generators.Base.extend({
  // The name `constructor` is important here
  constructor: function () {
    // Calling the super constructor is important so our generator is correctly set up
    generators.Base.apply(this, arguments);

    // Next, add your custom code
    this.option('coffee'); // This method adds support for a `--coffee` flag
  }
});
```

还有一些 `yeoman.generators` 预定义的函数，它们会按照优先级的顺序执行这些函数

> initializing - Your initialization methods (checking current project state, getting configs, etc)
>
> prompting - Where you prompt users for options (where you'd call this.prompt())
>
> configuring - Saving configurations and configure the project (creating .editorconfig files and other metadata files)
>
> default - If the method name doesn't match a priority, it will be pushed to this group.
>
> writing - Where you write the generator specific files (routes, controllers, etc)
>
> conflicts - Where conflicts are handled (used internally)
>
> install - Where installation are run (npm, bower)
>
> end - Called last, cleanup, say good bye, etc

你也可以自定义一些其他名字的函数
```
module.exports = generators.Base.extend({
  method: function () {
    console.log('method 1 just ran');
  },
  _privateMethod: function () {
    console.log('method 2 just ran');
  }
});
```
如果函数是 `_` 开头，则为私有函数，不会自动执行，否则这些自定义的函数的优先级为 `default`，会自动执行。

### 开始编写

在了解了 `yeoman.generators` 的基本功能之后，我们在回头看看我们的需求，其实可以把要做的工作分为几步

- 获取用户输入的 componont name
- 选择创建的 path
- 创建相应的 dir/files

开始编写处理函数

1. 前面两把都是通过跟用户交互获取输入，放在 `prompting` 这层做
  ```javascript
    prompting: function () {
      var done = this.async();

      this.log(yosay(
        'Welcome to the gnarly ' + chalk.red('generator-react-new-component') + ' generator!'
      ));

      var prompts = [{
        type: 'input',
        name: 'componentName',
        message: 'What\'s name new component?'
      }, {
        type: 'list',
        name: 'componentPath',
        message: 'Choices component path:',
        choices: function() {
          var dirs = [];
          var reg = new RegExp('^(client|app|src)/(component|container)(s)?/([a-z]*/)*$');
          var entries = walkSync.entries('.', {globs: ['client/**/*', 'app/**/*', 'src/**/*']})
          entries.forEach(function(entry) {
            if (entry.isDirectory() && reg.test(entry.relativePath)) {
              dirs.push(entry.relativePath);
            }
          });
          return dirs;
        },
        default: 'client/components/',
      }];

      this.prompt(prompts, function (props) {
        this.props = props;
        done();
      }.bind(this));
    },
  ```
  通过定义 `prompts`（基于 [Inquirer.js](https://github.com/SBoudrias/Inquirer.js)）来跟用户交互，获取用户的输入，然后保存到 `this.props` 上，给后续的处理函数使用。
2. 创建目录文件，放在 `writing` 这层做
  ```javascript
    writing: function () {
      var jsPath = this.props.componentPath + '/' + this.props.componentName;
      this.fs.copy(
        this.templatePath('jsTemplateFile'),
        this.destinationPath(jsPath),
        { componentName: this.props.componentName }
      );
    }
  ```
  根据用户输入的 `componentName` 和 `componentPath` 值，得到我们生成文件的`路径`。指定渲染的模板 `jsTemplateFile`，模板一般放在同级目录 `templates` 文件夹下，
  ```javascript
  import React, { Component } from 'react';

  class <%= componentName %> extends Component {
    render() {
      return (
        <div>
          <h1> <%= componentName %> </h1>
        </div>
      );
    }
  }

  export default <%= componentName %>;
  ```
  模板支持 `ejs` 的语法，可以传值渲染
3. 在完成了基本开发之后，在发布我们的生成器之前，我们可以先本地调试下，确保没问题了再发布到 `Npm` 上。在 `generator-xxx` 目录下执行
  ```
  npm link
  ```
  完成之后，再执行
  ```
  yo xxx
  ```
  就可以调用我们的生成器了。确保没问题之后再执行 `npm publish` 发布。

### 后记


完整的版本在这里 [generator-react-new-component](https://github.com/booxood/generator-react-new-component)。增加了几个 `prompt` 用于选择文件名后缀、样式文件后缀等。
之前看过一句话，大概意思是 `懒` 是程序员的第一生产力，貌似很有道理。
另外，上面只是介绍了 `yeoman.generators` 的基本功能，还有很多有意思、很人性化的功能都没提到，感兴趣可以继续深入。

### 相关

http://yeoman.io/authoring/index.html
https://github.com/SBoudrias/Inquirer.js
https://github.com/booxood/generator-react-new-component
