## 包管理工具

#### npm 和 yarn

###### npm 缺点

* 安装速度慢 （npm 是按照队列执行每个 package，也就是说必须要等到当前 package 安装完成之后，才能继续后面的安装）
* 同一个项目，安装的时候无法保持一致性。
* 输出日志太多，容易错过报错。

新版本npm的改进：

* 默认新增了类似yarn.lock的 package-lock.json （统一安装版本）
* 文件依赖优化 （在之前的版本，如果将本地目录作为依赖来安装，将会把文件目录作为副本拷贝到 node_modules 中。而在 npm5 中，将改为使用创建 symlinks 的方式来实现（使用本地 tarball 包除外），而不再执行文件拷贝。这将会提升安装速度。目前yarn还不支持）


###### yarn 优点

* 速度快（并行安装、离线缓存）
* 安装版本统一(yarn.lock)
* 更简洁的输出
* 多注册来源处理（所有的依赖包，不管他被不同的库间接关联引用多少次，安装这个包时，只会从一个注册来源去装，要么是 npm 要么是 bower, 防止出现混乱不一致。）
* 命令使用更好的语义化

##### npm 和 yarn 命令

| npm | yarn |
| --- | --- |
| npm init | yarn init |
| npm install |	yarn |
| npm install xxx --save | yarn add xxx |
| npm uninstall xxx --save | yarn remove xxx |
| npm install xxx --save-dev | yarn add xxx --dev |
| npm update --save | yarn upgrade |
| npm un/uninstall xxx | yarn remove xxx |
| npm i -g xxx | yarn global add xxx |
| npm un -g xxx | yarn global remove xxx |

##### npm包中@开头的包

如果包的名称以@开头,则它是一个范围包.范围是@和斜杠之间的所有内容  `@scope/project-name`

类似的有 `@babel/core`、 `@babel/preset-env` 等

##### node_modules中很多_开头的包

cnpm 安装的 包，淘宝镜像，和npm安装一样的。

##### 包版本

`major.minor.patch-premajor|preminor|prepatch`

* major 主版本
* minor 次版本
* patch 补丁版本

| 版本 | 含义 |
| --- | --- |
| ~ | 匹配大版本下最新发布的最小一级版本 |
| ^ | 匹配大版本下最新发布的次一级版本(第一个非零版本的次一级) |
| >= && < | 匹配大于等于或小于某个版本号的最新发布的版本 |
| * | 匹配最新发布的版本 |


### npm包本地调试 `npm link`

###### 相同目录下的链接

项目和模块在同一个目录下，可以直接使用相对路径链接

```javascript
// 进入项目目录
$ cd path/to/my-project
// 链接模块目录
$ npm link path/to/my-module
```

npm link 操作会在项目的 node_modules 目录下创建一个 module-name 的超链接（类似 Windows 的快捷方式）或称 “symlink”，链接到 path/to/my-module。

path/to/my-project 和 path/to/my-module 都是二者 package.json 文件所在的目录。

module-name 依据 npm 模块的 package.json 指定。

###### 不同目录下的链接

项目和模块不在同一个目录下，需要先把模块链接到全局，然后再在项目中链接模块

```javascript
// 先去到模块目录，把它链接到全局
$ cd path/to/my-module
$ npm link

// 再去项目目录
$ cd path/to/my-project
// 通过包名建立链接
$ npm link module-name
```

npm link 操作会在全局 node_modules 目录（如 MacOS 默认的是 /usr/local/lib/node_modules）下创建一个 module-name 的超链接。

此时只需要指定 module-name，在项目的 node_modules 目录下创建一个 module-name 的超链接，链接到 /usr/local/lib/node_modules/module-name，然后再由全局目录下的超链接，链接到具体的代码目录下。

###### 解除链接

解除项目和模块的链接

```javascript
// 进入项目目录，解除链接
$ cd path/to/my-project
$ npm unlink module-name
```

解除模块的全局链接
```javascript
// 进入模块目录，解除链接
$ cd path/to/my-module
$ npm unlink module-name
```

#### pnpm

pnpm是高性能的npm，通过**内容可寻址存储(CAS)**、**符号链接(Symbolic Link)**、**硬链接(Hard Link)**等管理依赖包，达到多项目之间依赖共享，减少安装时间。

1. 如果你用到了某依赖项的不同版本，只会将不同版本间有差异的文件添加到仓库。 例如，如果某个包有100个文件，而它的新版本只改变了其中1个文件。那么 pnpm update 时只会向存储中心额外添加1个新文件，而不会因为仅仅一个文件的改变复制整新版本包的内容。
2. 所有文件都会存储在硬盘上的某一位置。 当软件包被被安装时，包里的文件会硬链接到这一位置，而不会占用额外的磁盘空间。 这允许你跨项目地共享同一版本的依赖。

