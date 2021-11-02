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




