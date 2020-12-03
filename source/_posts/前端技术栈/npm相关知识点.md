---
title: npm相关知识点
toc: true
date: 2020-12-03 09:35:34
tags:
categories:
---

## npm相关文件
.npmrc文件：npm的配置文件, npm config命令能更新和配置这个文件，也可直接编辑，可配置选项可参考npm config命令支持选项，四个相关文件是
1. per-project config file (/path/to/my/project/.npmrc)
2. per-user config file (~/.npmrc)
3. global config file ($PREFIX/etc/npmrc)
4. npm builtin config file (/path/to/npm/npmrc)

package-lock.json: npm5的新特性，它描述了生成的确切node_modules树，因此无论中间依赖项更新如何，后续安装都可以生成相同的node_modules树。
npm-shrinkwrap.json: npm4及之前的lock文件，它是npm rinkwrap命令创建的文件，与package-lock.json类似。

## 主要文件夹
- prefix目录，默认是node安装目录，可用npm config list查看
- 本地安装模块时，模块会被安装在./node_modules，适用于你将会在项目中require()它使用，如果有可执行文件会安装在./node_modules/.bin下，这些命令可以用npx调用，npx 的原理很简单，就是运行的时候，会到node_modules/.bin路径和环境变量$PATH里面，检查命令是否存在。
- 全局安装模块时，即带-g标记，模块会被安装在{prefix}/node_modules目录中，适用于你将会要命令行使用它，其中可执行文件，在win下会被安装到{prefix}/下，而Unix则是在{prefix}/bin下

## Scope 
所有npm软件包都有一个名称。某些软件包名称也具​​有Scope。Scope遵循包名称的常用规则（不能用URL安全字符，不能点开头或下划线开头）。在软件包名称中使用时，作用域以@符号开头，后跟斜杠，例如
```
@somescope/somepackagename
```
Scope是将相关软件包组合在一起的一种方式，并且还会影响npm处理软件包的方式。每个npm用户/组织都有自己的Scope，只有您可以在自己的Scope内添加软件包。


## npm安装模块机制
npm官网：https://www.npmjs.com/

从 npm install 说起，npm install 命令用来安装模块到node_modules目录。
安装之前，npm install会先检查，node_modules目录之中是否已经存在指定模块。如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也是如此。如果你希望，一个模块不管是否安装过，npm 都要强制重新安装，可以使用-f或--force参数。

如果想更新已安装模块，就要用到npm update命令。它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。

### registry
npm update命令怎么知道每个模块的最新版本呢？

答案是 npm 模块仓库提供了一个查询服务，叫做 registry。同时也有三方的registry，这里以 npmjs.org 为例，它的查询服务网址是 https://registry.npmjs.org/ 。这个网址后面跟上模块名，就会得到一个 JSON 对象，里面是该模块所有版本的信息。比如，访问 https://registry.npmjs.org/react，就会看到 react 模块所有版本的信息。registry 网址的模块名后面，还可以跟上版本号或者标签，用来查询某个具体版本的信息。比如， 访问 https://registry.npmjs.org/react/v0.14.6 ，就可以看到 React 的 0.14.6 版。返回的 JSON 对象里面，有一个dist.tarball属性，是该版本压缩包的网址。

```
dist: {
  shasum: '2a57c2cf8747b483759ad8de0fa47fb0c5cf5c6a',
  tarball: 'http://registry.npmjs.org/react/-/react-0.14.6.tgz' 
}
```
到这个网址下载压缩包，在本地解压，就得到了模块的源码。npm install和npm update命令，都是通过这种方式安装模块的。

### 缓存目录
npm install或npm update命令，从 registry 下载压缩包之后，都存放在本地的缓存目录。这个缓存目录，在 Linux 或 Mac 默认是用户主目录下的.npm目录，在 Windows 默认是%AppData%/npm-cache。通过配置命令，可以查看这个目录的具体位置。
```
$ npm config get cache
```
你会看到里面存放着大量的模块，每个模块的每个版本，都有一个自己的子目录，里面是代码的压缩包package.tgz文件，以及一个描述文件package/package.json。

总结一下，Node模块的安装过程是这样的：

1. 发出npm install命令
2. npm 向 registry 查询模块压缩包的网址
3. 下载压缩包，存放在缓存目录
4. 解压压缩包到当前项目的node_modules目录

## 常用命令
### node init
* 交互式创建一个package.json文件

### npm install X -g
* 全局安装X
* 会安装到node安装目录下的node_modules

### npm install X

- 会把X包安装到当前目录下的node_modules目录中
- 不会修改package.json
- 之后运行npm install命令时，不会自动安装X

### npm install X –save

- 会把X包安装到node_modules目录中
- 会在package.json的dependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中

### npm install X –save-dev

- 会把X包安装到node_modules目录中
- 会在package.json的devDependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中
- 安装开发阶段依赖，一般调试工具居多


## 参考资料
> - []()
> - []()
