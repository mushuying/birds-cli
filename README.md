项目结构分成了四个部分：

- `@chicken2/cli` 脚手架命令行内容，通过命令去初始化项目等等操作。
- `@chicken2/scripts` 项目编译运行打包内容，暂未接入部署等流程。
- `@chicken2/template` 模板文件。
- `@chicken2/plugin-xxx` 项目封装的插件，例如`@chicken2/plugin-typescript`等等。

## cli

而在`@vue/cli`里，是通过多个插件组合成一个整体模板，在很多脚手架根目录下，都有一个`xxx.config.js`暴露出来。然后在运行`node`命令时，去读取配置文件，根据配置文件的内容去进行对应操作，例如：使用`webpack-chain`动态修改`config`，最后再调用`toConfig`去生成新的`webpack`配置内容。

### 插件判断，生成 pkg

一个基本的`package.json`模板，除了常规不变的`version`、`private`、`license`等等，像`name`，`scripts`，`dependencies`，`devDependencies`需要我们去手动添加进去。

`name`就使用脚手架初始化传入的参数，而`scripts`则是在成功引入`@chicken2/scripts`后，使用其运行命令。

像一些必备的，例如`react`，`react-dom`，我们可以直接放到`dependencies`里，而`devDependencies`一般是初始化时，用户手动选择的`plugins`。

借助`inquirer`去罗列插件，让用户选择需要引入哪些插件。

那么根据上面的`code`就可以获取到，用户需要哪些`plugin`了，那么可以将这些`plugin`放入到`json`的`devDependencies`里。

### 获取依赖最新版本

上述无论是`react`还是`plugin`，都需要一个版本号，我这里是采用命令行去获取最新版本，然后作为其`value`值。如果直接遍历运行`execSync`的话，会阻塞住，`ora`的`loading`也要卡着不动，于是选择`promise`去运行，通过`exec`回调来结束`promise`。

那么就获取到版本号后，再将其他数据一同填入到`json`中，将其作为`package.json`的值，在新项目目录下，新建它。

## scripts

因为是多页面项目，`scripts`里主要做了以下几件事情：

- 通过`glob`去匹配入口，然后将其作为`entry`动态传入，并动态传入多个`html-webpack-plugin`给`plugins`。
- 通过读取项目根目录下的`chicken2.config.js`文件，来动态修改`webpack`配置内容并调用对应的插件。
- 最后生成最终的`webpack`配置文件，传入给`webpack`去进行编译运行打包等等操作。

### 匹配入口

匹配入口主要使用`glob`去匹配，只有满足匹配要求，才作为入口。然后通过匹配到的信息，去生成对应的`entry`内容，和`plugin`内容，传递给`webpack`配置文件。

### 链式配置 config

链式配置推荐使用[webpack-chain](https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans)，`@vue/cli`也是使用它。因为我们原本就有一些基本配置内容，可以通过`config.merge`将我们已有的配置对象合并到配置实例中。

但是不支持直接转化，需要我们对某些配置内容，进行手动去转化，例如：`module`。而`plugins`不支持已经`new`的`plugin`，我这边的处理是跳过对`plugin`的合并，最后再使用`webpack-merge`将`config.toConfig()`和`plugins`再合并成最终的配置对象。

### 编译运行

在获取到`webpack config`后，那么可以根据是`dev`命令还是`build`命令，去调用对应的函数，进行编译运行打包等等操作了。(同理，根据`program.command`）

## template

`template`主要就是通过传入的参数，来判断是否要`copy`对应的文件，同时根据`options`来去修改对应文件内容和后缀。代码过于无趣，就不贴了。

## plugin-xx

`plugin`的话，可以做的事情比较多，我这边目前就只是来链式修改`webpack`配置信息。这只是其中一种功能，还有很多例如：自己写一个`webpack plugin / loader`去传入，去做一些其他事情。
