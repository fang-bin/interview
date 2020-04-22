#### webpack打包原理和流程

本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

四个核心概念：入口(entry)，输出(output)，loader，插件(plugins)

###### webpack打包流程

1. 读取文件，分析模块依赖
2. 对模块进行解析执行（深度遍历）
3. 针对不同的模块使用不同的 loader
4. 编译模块，生成抽象语法树（AST）
5. 遍历 AST，输出 JS


#### webpack和其他自动化构建工具(gulp,grunt, rollup)有什么区别
webpack 是 module bundle
gulp 是 tast runner
Rollup 是在 Webpack 流行后出现的替代品。Rollup 在用于打包 JavaScript 库时比 Webpack 更加有优势，因为其打包出来的代码更小更快。 但功能不够完善，很多场景都找不到现成的解决方案。

#### webpack 的 loader 和 plugin 区别，举几个常用的 loader 和 plugin 并说出作用

loader 用于对模块的源代码进行转换。loader 可以使你在 import 或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 import CSS文件！ 因为 webpack 本身只能处理 JavaScript，如果要处理其他类型的文件，就需要使用 loader 进行转换，loader 本身就是一个函数，接受源文件为参数，返回转换的结果。
Plugin 是用来扩展 Webpack 功能的，通过在构建流程里注入钩子实现，它给 Webpack 带来了很大的灵活性。 通过plugin（插件）webpack可以实 loader 所不能完成的复杂功能，使用 plugin 丰富的自定义 API 以及生命周期事件，可以控制 webpack 打包流程的每个环节，实现对 webpack 的自定义功能扩展。

#### 模块化解决了哪些痛点
* 命名冲突
* 文件依赖
* 代码复用

###### webpack又是通过什么方式解决了这些痛点


#### webpack打包之后生成哪些文件

#### webpack 热部署的原理

#### webpack 打包出来的文件体积过大怎么办

#### webpack 打包速度过慢怎么办？