## 1. render props

带有渲染属性(Render Props)的组件需要一个返回 React 元素并调用它的函数，而不是实现自己的渲染逻辑。

render prop 是一个组件用来了解要渲染什么内容的函数 prop

#### 高阶函数 和 Render Props

高阶组件其实只是一个装饰器模式，用于增强原有组件(抽离抽象逻辑，对逻辑进行复用)的功能。

高阶组件其实并不是组件，只是一个函数而已。它接收一个组件作为参数，返回一个新的组件。我们可以在新的组件中做一些功能增加，渲染原有的组件。这样返回的组件增强了功能，但渲染与原有保持一致，没有破坏原有组件的逻辑。

render props，同样也是提高组件复用和抽象的手段。

区别：

抽象来说，区别在于控制权

* HOC 的控制权在 Wrapper 组件，最后的渲染结果其实是由父组件决定的
* render props 则是控制反转再反转，控制权在子组件，父组件可以传一些“参考”给子组件，但最后渲染的结果还是完完全全由子组件控制的。

如果高阶组件依赖被高阶组件的执行上下文，则必须使用render props，换句话说 render props可以完成高阶组件能做的事，但反过来并不成立

```javascript
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

## 2. token 失效

基本思路：

**单个 token** 的情况下：

1. token 过期时间为 x 分钟
2. 前端发起情趣，后端验证 token 是否过期，如果过期，前端发起刷新 token 的请求，后端设置已再次授权标志为 true，请求成功
3. 前端发起请求，后端验证再次授权标记，如果已经再次授权，则拒绝刷新 token 的请求，请求成功
4. 如果前端每隔72小时，必须重新登录，后端检查用户最后一次登录日期，如超过72小时，则拒绝刷新token的请求，请求失败

**两个 token** 的情况下：

`access_token` 和 `refresh_token`

* `access_token` 主要用来请求（过期时间较短）
* `refresh_token` 主要用来刷新 `asccess_token` （过期时间较长）

#### 有状态登陆 和 无状态登陆

#### 有状态登陆

传统上，我们会使用 Session 和 Cookie 来保存用户的授权信息。第一步，登录过程，用户使用用户名和密码来登录系统，服务器会来验证用户名和密码是否正确，如果正确，服务器会给这个用户创建一个包含用户登录信息、角色、权限的一个叫做 Session 的东西，然后把这个 Session 保存起来，同时把这个 Session 的 ID 以 Cookie 的形式发送给前端，表示用户验证成功，登录完成了；接下来如果用户希望访问某些资源，前端要向后端发起一个 HTTP 的请求，同时相应的 Cookie 也会跟随请求一起发送给服务器，而服务器取得 Cookie 以后就会去查找是否有 Session ID ，然后通过 Session ID 提取相应的 Session 来确定用户的身份与权限，如果 Session 与 ID 相符，同时用户的信息也能提供相应的权限，服务器就会认为这个用户已经登录了，随后资源信息就会通过 HTTP 响应给前端。

**缺点**:

* 服务端保存大量数据，增加服务端压力
* 服务端保存用户状态，无法进行水平扩展
* 客户端请求依赖服务端，多次请求必须访问同一台服务器（如果集群了，相当于启动了多个tomcat，就需要在多个tomcat之间共享数据）

#### 无状态登陆

第一步，同样是登陆，用户使用用户名和密码登陆，如果登陆成功，服务器就会返回一个加密文档，这个文档就是 JWT ，其中包含用户密码以外，全部的认证信息，包括用户名、Email、角色、权限等等，而前端在拿到 这个JWT 以后就可以把它保存起来了，可以保存到 Cookie 中，也可以保存到浏览器的 LocaStorage 里面，而生成的 JWT 不需要在后端保存，接下来第二步，用户如果需要访问某些权限的时候，这时候，用户就要把 JWT 放在 HTTP 请求 herder 中与请求一起发送给服务器，服务器取得 JWT 以后 会使用私钥给 JWT 文档解密 ，如果解密成功而且数据依然有效则代表用户已经登陆了，如果 JWT 所描述的用户权限允许该用户访问资源，那么服务器就会把资源的信息，通过 HTTP 响应发回给前端。

**优点**
* 客户端请求不依赖服务端的信息，任何多次请求不需要必须访问到同一台服务器
* 服务端的集群和状态对客户端透明
* 服务端可以任意的迁移和伸缩
* 减小服务端存储压力

**缺点**

* 占带宽
* 无法在服务端注销（如果没有在服务端redis中存储信息）
* 性能问题（加密签名）

#### 区别

传统上用户登陆状态会以 Session 的形式保存在服务器上，而 Session ID 则保存在前端的 Cookie 中；而使用 JWT 以后，用户的认证信息将会以 Token 的形式保存在前端，服务器不需要保存任何的用户状态，这也就是为什么 JWT 被称为无状态登陆的原因，无状态登陆最大的优势就是完美支持分布式部署，可以使用一个 Token 发送给不同的服务器，而所有的服务器都会返回同样的结果。

### 单点登陆

单点登录SSO（Single Sign On）说得简单点就是在一个多系统共存的环境下，用户在一处登录后，就不用在其他系统中登录，也就是用户的一次登录能得到其他所有系统的信任。

实现方式:

1. 解决系统之间Session不共享问题(把Session数据放在Redis中，使用redis模拟Session)
2. JWT

## 3. CI/CD

CI(Continuous Integration): 持续继承
CD(Continuous Delivery): 持续交付

* gitlab 用于代码版本管理，并通过其提供的 webhook 功能，触发 jenkins job 的运行

* jenkins 用来执行项目中 单元测试，编译打包相关 npm 命令，并发送反馈邮件，执行远程部署脚本。

* nodejs 用于提供单元测试，编译打包功能的 npm 命令

**如果没有 CI/CD， 我们的前端从开发到提测工作流程可能如下：**

1. 本地机器上写代码
2. 在命令行输入 npm run unit，查看单元测试结果
3. 提交代码，push 到 git 远程仓库
4. 登录测试服务器，拉取代码，执行 npm run build，构建项目
5. 如果测试服务器是基于 pm2 的 proxy server，还需要重启 server

这个流程中，每一个步骤都要重复人工操作，很大增加了时间成本，不能保证操作的准确性。对于 unit 或者 build 的结果，没有一个自动的反馈机制，需要人工 check 运行结果，最后部署也是人工登录服务器执行脚本，非常繁琐。

**引入 CI/CD 以后，整个流程变成：**

1. 本地机器上写代码
2. 提交代码，push 到 git 远程仓库
3. git hook 触发 jenkins 的构建 job （自动）
4. jenkins job 中拉取项目代码，运行 npm run unit 和 npm run build，如果失败，发送邮件通知相关人。（自动）
5. jenkins job 中执行测试服务器的部署脚本 （自动）

在 CI/CD 流程中，只有步骤1和步骤2需要人工操作，其他步骤都是自动运行，是一个非常标准化的流程，减少了人工操作的风险，省去了重复性工作，增强了项目的可见性。接下来我们将通过配置 jenkins 和 gitlab webhook 来实现这个流程。

[前端开发如何让持续集成/持续部署(CI/CD)跑起来](https://zhuanlan.zhihu.com/p/26701038)
[前端自动构建和部署](https://segmentfault.com/a/1190000038472366?utm_source=tag-newest)

## 4. 埋点

![tracking](https://github.com/fang-bin/interview/blob/master/image/tracking.jpeg)

前端埋点的目的: 获取用户基本信息、行为以及跟踪产品在用户端的使用情况，并以监控数据为基础，指明产品优化的方向。

前端监控可以分为三类：**数据监控、性能监控和异常监控**。

前端埋点又可以分为三类埋点方法:**代码埋点，可视化埋点和无埋点**。

##### 代码埋点

###### 优点

* 可自定义属性，自定义事件
* 可以细化需求
* 相比其他埋点方式减少服务器压力

###### 缺点

* 工程量大的话，手动埋点会出现疏漏，不方便审查。
* 需求变更要重新埋点，成本高。
* 每次需求变更都要重新发布版本，对线上系统稳定性有一定危害

##### 可视化埋点

可视化埋点与好多年前比较流行的面向业务人员的网页制作工具Dreamweaver 类似，即所见即所得，通过点击交互替代手写代码。可视化埋点参考Visual Studio 等一系列IDE做法，用可视化的页面交互手段来代替代码编写，从而大幅缩减工作量和沟通成本，同时降低出错几率。

从流程上讲，每次埋点后，业务人员都还要等待APP/网页/小程序的更新发版or上线，才能看到数据，这种时间滞后性，大大伤害了业务人员的数据使用需求。所以，参考很多手游的做法，把核心代码、配置、资源分开，通过网络更新配置和资源从而实现采集代码下发。达到所见即所得的效果。

相比于手动埋点更新困难，埋点成本高的问题，可视化埋点优化了移动运营中数据采集的流程，能够支持产品运营随时调整埋点，无需再走发版流程，直接把配置结果推入到前端，数据采集流程更简化，也更方便产品的迭代。

##### 无埋点

无埋点则是前端自动采集全部事件，上报埋点数据，由后端来过滤和计算出有用的数据。

###### 优点

* 前端只要一次加载埋点脚本

###### 缺点

* 服务器性能压力山大

埋点事件: **曝光事件，页面停留事件，点击事件**

如何设计前端代码埋点：

1. 同种类的多个时间要命名为一个埋点事件
2. 不同属性的多个事件应命名为多个埋点事件

Key-Value形式的埋点设计规则：一个Key-某个事件，可以对应一个或者多个Value-事件对应的值。

实现前端监控，第一步肯定是将我们要监控的事项（数据）给收集起来，再提交给后台，最后进行数据分析。数据收集的丰富性和准确性会直接影响到我们做前端监控的质量，因为我们会以此为基础，为产品的未来发展指引方向。

## 5. 权限管理

[权限管理设计](https://juejin.cn/post/6844903988945485837)

## 6. react hook缺点(闭包陷阱)

```javascript
import "./styles.css";
import {useEffect, useState} from 'react'

export default function App() {
  const [count, setCount] = useState(0)
  useEffect(()=>{
    setInterval(() => {
      setCount(count + 1); //count 的值会一直保持 1
    }, 1000);
  }, [])
  return (
       <p>count={count}</p>
  );
}
```

产生原因：

1. 函数、控制块执行时会产生新的词法环境
2. 当前词法环境找不到的变量会继续外外部词法环境寻找

当某个组件需要更新时，总是会调用这个组件对应的函数或者 render 函数。而每次函数调用总会生成一个新的词法环境。但是，对于 useEffect 中的回调函数，它始终捕获第一个函数调用时生成的词法环境中的 count(因为它只会执行一次，没有机会捕获第二次，第三次函数执行产生的词法环境中的 count)，因此被捕获的 count 的值始终为 0，而 count + 1 的值也始终是 1。对于这种情况，如果希望出现预期的效果，就需要 useEffect 中的函数多次运行，不断的捕获新的词法环境中的 count。将 useEffect 改为:

```javascript
useEffect(()=>{
  setInterval(() => {
      setCount(count + 1)
    }, 1000);
}, [count]);
```

## 7. 前端低代码 无代码(页面可视化搭建工具)

低代码平台主要针对技术开发人员，使他们能够在几天甚至几个小时内快速构建应用前端。这使得他们能够更快地进入软件开发中最有趣的部分:定制。低代码平台适合创建更复杂的应用程序和流程，这些应用程序和流程需要与其他应用程序数据库或系统集成。

无代码平台面向的是那些没有预算外包开发或内部雇佣开发人员的小型企业。使用无代码平台，业务开发人员无需编写代码就可以创建和部署完整的应用程序。这种速度、易用性和简单性的缺点是，无代码平台只能够真正开发不需要与任何其他系统集成的基本应用程序。它们可能有助于简化手工的内部流程，但它们根本没有能力开发具有竞争力的创新软件，也很难做到一些个性化的自定义功能。

低代码/无代码开发与软件工程领域的一些经典思想、方法和技术，例如软件复用与构件组装、软件产品线、DSL（领域特定语言）、可视化快速开发工具、可定制工作流，以及此前业界流行的中台等概念，之间是什么关系？

[前端的低代码/无代码](https://mp.weixin.qq.com/s/p6vnrlM7OPDyu-dWzf94kA)

从库、框架、脚手架开始，软件工程就踏上了追求效率的道路。在这个道路之上，低代码、无代码的开发方式算是宏愿。复用、组件化和模块化、DSL、可视化、流程编排……都是在达成宏愿过程中的尝试，要么在不同环节、要么以不同方式，但都还在软件工程领域内思考。中台概念更多是在业务视角下提出的，软件工程和技术领域内类似的概念更多是叫：平台。不论中台还是平台，就不仅是在过程中的尝试，而是整体和系统的创新尝试。我提出前端智能化的“人机协同编程”应该同属于软件工程和技术领域，在类似中台的业务领域我提出“需求暨生产”的全新业务研发模式，则属于业务领域。这些概念之间无非：左右、上下、新旧关系而已。

#### 页面可视化搭建工具

前端页面早在十几年前就能用 Dreamweaver, Frontpage 等工具可视化搭建出来。但是现在已经很少人使用 Dreamweaver 了, 其主要原因是页面承载的内容已经和页面源码分离, 由后端接口返回再渲染到页面, 静态页面网站无法承载大量的动态内容.

页面可视化搭建：飞冰、云凤蝶

页面模块化搭建： 斑马

## 8. ts 中的泛型

组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。

在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

```javascript
function identity<T>(arg: T): T {
  return arg;
}
// 第一种是，传入所有的参数，包含类型参数：
let output = identity<string>("myString");

// 第二种方法更普遍。利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定T的类型：
let output = identity("myString"); 
```

常见的泛型

```javascript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

## 9. ts 中的 type interface 的区别，其他类型 Tuple Enum 等怎么样

##### 相同

interface 和 type 都可以用来描述一个函数或者对象、类 （但是interface一般不用来描述基本类型）

* 都允许拓展 `interface extends interface`、`type & type`、`interface extends type`、`type & interface`

##### 不同点

* type 可以声明基本类型别名，联合类型，元组，交叉类型等类型
* type 语句中还可以使用 typeof 获取实例的 类型进行赋值
    ```javascript
    // 当你想获取一个变量的类型时，使用 typeof
    let div = document.createElement('div');
    type B = typeof div
    ```
* interface 能够声明合并
    ```javascript
    interface User {
      name: string
      age: number
    }
    
    interface User {
      sex: string
    }
    
    /*
    User 接口为 {
      name: string
      age: number
      sex: string 
    }
    */
    ```

##### Enum

使用枚举我们可以定义一些带名字的常量。 使用枚举可以清晰地表达意图或创建一组有区别的用例。TypeScript支持数字的和基于字符串的枚举。

##### Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 string和number类型的元组。

## 10. https 抓包

#### http 的抓包原理

![charles-http图片](https://github.com/fang-bin/interview/blob/master/image/charles-http.jpeg)

1. 首先抓包工具会提供出代理服务，客户端需要连接该代理；
2. 客户端发出 HTTP 请求时，会经过抓包工具的代理，抓包工具将请求的原文进行展示；
3. 抓包工具使用该原文将请求发送给服务器；
4. 服务器返回结果给抓包工具，抓包工具将返回结果进行展示；
5. 抓包工具将服务器返回的结果原样返回给客户端；

抓包工具就相当于个透明的中间人，数据经过的时候它一只手接到数据，然后另一只手把数据传出去。

#### https 的抓包原理

![charles-https图片](https://github.com/fang-bin/interview/blob/master/image/charles-https.jpeg)

这个时候抓包工具对客户端来说相当于服务器，对服务器来说相当于客户端。在这个传输过程中，客户端会以为它就是目标服务器，服务器也会以为它就是请求发起的客户端。

1. 客户端连接抓包工具提供的代理服务；
2. 客户端需要安装抓包工具的根证书；
3. 客户端发出 HTTPS 请求，抓包工具模拟服务器与客户端进行 TLS 握手交换密钥等流程；
4. 抓包工具发送一个 HTTPS 请求给客户端请求的目标服务器，并与目标服务器进行 TLS 握手交换密钥等流程；
5. 客户端使用与抓包工具协定好的密钥加密数据后发送给抓包工具；
6. 抓包工具使用与客户端协定好的密钥解密数据，并将结果进行展示；
7. 抓包工具将解密后的客户端数据，使用与服务器协定好的密钥进行加密后发送给目标服务器；
8. 服务器解密数据后，做对应的逻辑处理，然后将返回结果使用与抓包工具协定好的密钥进行加密发送给抓包工具；
9. 抓包工具将服务器返回的结果，用与服务器协定好的密钥解密，并将结果进行展示；
10. 抓包工具将解密后的服务器返回数据，使用与客户端协定好的密钥进行加密后发送给客户端；
11. 客户端解密数据；

## 11. ESLint TSLint

ESLint 和 TSLint 分别是 JavaScript 和 TypeScript 的代码检查工具。

代码检查是一种静态的分析，常用于寻找有问题的模式或者代码，并且不依赖于具体的编码风格。对大多数编程语言来说都会有代码检查，一般来说编译程序会内置检查工具。

JavaScript 是一个动态的弱类型语言，在开发中比较容易出错。因为没有编译程序，为了寻找 JavaScript 代码错误通常需要在执行过程中不断调试。像 ESLint 这样的可以让程序员在编码的过程中发现问题而不是在执行的过程中。

ESLint 使用 Node.js 编写，这样既可以有一个快速的运行环境的同时也便于安装。

**ESLint 和 TSLint 的目标是保证代码的一致性和避免错误。**

##### 配置

两种配置方式:

1. 使用 JavaScript 注释把配置信息直接嵌入到一个代码源文件中。
2. 使用 JavaScript、JSON 或者 YAML 文件为整个目录（处理你的主目录）和它的子目录指定配置信息。可以配置一个独立的 .eslintrc.* 文件，或者直接在 package.json 文件里的 eslintConfig 字段指定配置，ESLint 会查找和自动读取它们，再者，你可以在命令行运行时指定一个任意的配置文件。

ESLint 的一般默认配置文件是 `.eslintrc.json` 。

##### 主要配置项

* `parserOptions` 指定要支持的 JavaScript 语言选项（支持 JSX 语法并不等同于支持 React，要使用 eslint-plugin-react 才行）
* `rules` 启用的规则及其各自的错误级别
* `env` 指定脚本的运行环境。每种环境都有一组特定的预定义全局变量。
* `extends` "eslint:recommended" 属性启用一系列核心规则。
  如果觉得官方提供的默认规则不好用，可以自定义规则配置文件，然后发布成 Npm 包，然后 `"extends": ["npm包"]`;

rules 中的错误级别:

* `"off"` or `0` - 关闭规则
* `"warn"` or `1` - 将规则视为一个警告（不会影响退出码）
* `"error"` or `2` - 将规则视为一个错误 (退出码为1)

##### ignore

* `.eslintignore` 文件中指定的文件或目录不会执行 eslint
* 命令行中设置忽略文件、目录
* 行内注释 `//eslint-disable-line`  （其他一些方式注释进行忽略）

## 12. 国际化

在计算机领域，国际化是指设计能够适应各种区域和语言环境的软件的过程。可以把它理解为一个页面可以使用不同语言进行切换显示的一个过程。

一般来说前端国际化有两种实现方式:

1. 针对不同的语言，各写一套界面。
2. 使用配置文件的方式，使用一套界面，同样的样式文件，调用相对应的语言文件进行DOM渲染。且只需维护一套前端文件，根据语言的不同加载对应的配置文件。

第一种方式最主要的问题是 维护成本高 并且 每次切换的时候都需要重新发送请求，每次都要重新加载整个页面，对性能的影响较大。

通常在 react 中使用 react-intl

## 13. 微前端

## 14. 封装 xhr 或 fetch 实现拦截器功能

* 请求拦截器
* 响应拦截器

axios 主要实现的功能：
* 创建请求 XMLHttpRequest
* 拦截请求和响应
* 转换请求数据和相应数据
* 取消请求
* 自动转换JSON
* 其他的一些基础配置，例：method、type、timeout、withCredentials、request-headers、proxy等

## 15. 服务端渲染

React 服务端渲染框架 Next.js

## 16. websocket 实践

WebSocket 是 HTML5 开始提供的一种浏览器与服务器间进行全双工通讯的网络技术。在 WebSocket API 中，浏览器和服务器只需要要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

websocket相对与HTTP协议来说是一个持久化的协议。

特点：
* 建立于 TCP 协议之上的应用层；
* 一旦建立连接（直到断开或者出错），服务端与客户端握手后则一直保持连接状态，是持久化连接；
* 服务端可通过实时通道主动下发消息；
* 数据接收的「实时性（相对）」与「时序性」；
* 较少的控制开销。连接创建后，ws客户端、服务端进行数据交换时，协议控制的数据包头部较小。在不包含头部的情况下，服务端到客户端的包头只有2~10字节（取决于数据包长度），客户端到服务端的的话，需要加上额外的4字节的掩码。而HTTP协议每次通信都需要携带完整的头部。
* 支持扩展。ws协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议。（比如支持自定义压缩算法等）

#### 使用

在浏览器中使用 Websocket 非常简单，在支持 Websocket 的浏览器中会提供了原生的 WebSocekt 对象，其中对于消息的接收与数据帧处理在浏览器中已经封装好了。

`new WebSocket(String url[, optional String | Array<protocols>]);`

##### 方法

* `WebSocket.prototype.send(String|ArrayBuffer|Blob)` 发送数据到服务端
* `WebSocket.prototype.close(code, reason)` 接收一个（可选）的 code（关闭状态号，默认为 1000） 与一个（可选）的字符串（表示断开原因），客户端主动断开连接；

##### 常量、状态

* `WebSocket.prototype.CONNECTING 0` 与 `WebSocket.CONNECTING 0` 连接还没开启；
* `WebSocket.prototype.OPEN 1` 与 `WebSocket.OPEN 1` 连接已开启并准备好进行通信；
* `WebSocket.prototype.CLOSING 3` 与 `WebSocket.CLOSING 3` 连接正在关闭的过程中；
* `WebSocket.prototype.CLOSED 4` 与 `WebSocket.CLOSED 4` 连接已经关闭，或者连接无法建立；
* **`WebSocket.prototype.readyState`** 判断当前状态；

##### 监听事件

* `WebSocket.prototype.onopen` 连接打开的回调事件，这时 readyState 变为 OPEN；
* `WebSocket.prototype.onmessage` 收到消息的回调事件，同时回调函数接收到一个 MessageEvent 数据；
* `WebSocket.prototype.onclose` 连接关闭的回调事件，这时 readyState 变为 CLOSED；
* `WebSocket.prototype.onerror` 建立与连接过程发生错误的回调事件；

添加事件两种方式: 一：`on{事件} = fn` 二：`websocket.addEventListener({事件}, fn)`

其中 MessageEvent 对象中 可以拿到 data 属性，这就是数据。

不论服务端与客户端，其接受到的数据都是序列化后的字符串（当然也有 ArrayBuffer|Blob 类型数据），很多时候我们需要解析处理数据，比如 JSON.parse(e.data)；

##### 连接稳定性

由于网络环境复杂，某些情况会出现断开连接或者连接出错，需要我们在 close 或者 error 事件中监听非正常断开并重连；

由于一些原因在 error 时浏览器并不会响应回调事件，因此稳妥的做法还需要在 open 之后开启一个定时任务去判断当前的连接状态 readyState ，在出现异常情况下尝试重连；

###### 心跳

websocket规范定义了心跳机制，一方可以通过发送ping（opcode 0x9）消息给另一方，另一方收到ping后应该尽可能快的返回pong（0xA）。

心跳机制是用于检测连接的对方在线状态，因此如果没有心跳，那么无法判断一方还在连接状态中，一些网络层比如 nginx 或者浏览器层会主动断开连接，

在 JavaScript 中，WebSocket 并没有开放 ping/pong 的 API ，虽然浏览器自带了心跳处理，然而不同厂商的实现也不尽相同，因此需要在我们开发时候与服务端约定好一个自实现的心跳机制；

比如浏览器中，检测到 open 事件后，启动一个定时任务，每次发送数据 0x9 给服务端，而服务端返回 0xA 作为响应；

实践下来，心跳的定时任务一般是相隔 15-20 秒发送一次。

#### WebSocket 建立连接

WebSocket 连接分为建立连接阶段和连接阶段，建立连接阶段需要借助 HTTP ，连接阶段则和 HTTP 无关。

请求头:

```javascript
Connection: Upgrade  //要升级协议
Upgrade: websocket //要升级到 webscoket 协议
Sec-WebSocket-Version: 13  //表示websocket的版本。如果服务端不支持该版本，需要返回一个Sec-WebSocket-Versionheader，里面包含服务端支持的版本号。
Sec-WebSocket-Key: 2idFk3+96Hs5hh+c9GOQCg==  //是一个 Base64 encode 的值，由浏览器随机生成的，用于验证服务器连接的正确性；与后面服务端响应首部的Sec-WebSocket-Accept是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。
```

Connection 为 Upgrade ，Upgrade 为 websocket ，表示告知 Nginx 与 Apache 等服务器该次连接并非为 HTTP 连接，实质上是一个 websocket ，因此服务器会转发到相应的 websocket 任务处理；

响应头:

```javascript
HTTP/1.1 101 Switching Protocols  //返回状态码为 101 ，表示切换协议；
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: py9bt3HbjicUUmFWJfI0nhGombo=
```

Upgrade 与 Connection 用于回复客户端表示已经切换协议成功；

Sec-WebSocket-Accept 字段与 Sec-WebSocket-Key 相对应，用于验证服务的正确性；

#### WSS

在 Websocket 协议中，可以使用加密传输的 —— wss 

使用的也是与 HTTPS 一样的证书，在这里一般是交由 Nginx 等服务层去做证书处理。

##### Sec-WebSocket-Key/Accept的作用

`Sec-WebSocket-Key/Sec-WebSocket-Accept`在主要作用在于提供基础的防护，减少恶意连接、意外连接。

1. 避免服务端收到非法的websocket连接（比如http客户端不小心请求连接websocket服务，此时服务端可以直接拒绝连接）
2. 确保服务端理解websocket连接。因为ws握手阶段采用的是http协议，因此可能ws连接是被一个http服务器处理并返回的，此时客户端可以通过Sec-WebSocket-Key来确保服务端认识ws协议。（并非百分百保险，比如总是存在那么些无聊的http服务器，光处理Sec-WebSocket-Key，但并没有实现ws协议。。。）
3. 用浏览器里发起ajax请求，设置header时，Sec-WebSocket-Key以及其他相关的header是被禁止的。这样可以避免客户端发送ajax请求时，意外请求协议升级（websocket upgrade）
4. 可以防止反向代理（不理解ws协议）返回错误的数据。比如反向代理前后收到两次ws连接的升级请求，反向代理把第一次请求的返回给cache住，然后第二次请求到来时直接把cache住的请求给返回（无意义的返回）。
5. Sec-WebSocket-Key主要目的并不是确保数据的安全性，因为Sec-WebSocket-Key、Sec-WebSocket-Accept的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。

## 17. 动态表单

业务场景：

场景一： 最初工作流软件表单功能相对比较落后，每一个表单制作都要代码去写，但这种方式不能满足企业变化的需求。

场景二： OA系统，基本上采用表单+流程就可以实现，然而大量的表单和表单的易变动性，对于开发人员维护时非常痛苦的。比如：许多公文在多部门直接传递审批，不同的公文有不同的要求，内容与格式，这时候就适合使用动态表单。

场景三： 简单企业网站，用内容管理系统cms建设，有点资源浪费。

**目的**：动态表单，可以灵活配置并扩展业务、避免在系统中硬编码的数据采集及处理表单，提高系统的可维护性。动态表单，使开发人员把注意力集中在业务流程上，同时，也可以让系统操作人员参与表单的管理。

动态表单分为两部分：一部分**动态表单的定义和显示**，另一部分**动态表单内容的接收和存储**

设计思路：

利用横向表纵向存储的思路，即一张表保存表单的定义信息（分组），一张表保存表单的字段配置信息（分组字段），这样做可以灵活扩展表单（后期如果添加一个字段，只需要往表里插入一条数据即可）。

利用mongodb数据库对表单数据内容存储。（不足之处，数据统计功能弱）

## 18. 大文件断点上传

现在前端上传文件主要是通过 FromData 上传大文件。

其中，File 对象继承自 Blob 对象。（`Object.getPrototypeOf(File.prototype) === Blob.prototype`）

可以对文件进行切片上传，加快上传速度，原理即是 `Blob.prototype.slice` 方法，可以对 Blob 对象进行分割。

[大文件上传](https://juejin.cn/post/6844904046436843527)

## 19. HTTP 状态码  401 和 403 的区别

* 401 未授权
* 403 禁止

##### 401
该状态码表示发送的请求需要HTTP认证的认证信息，如果之前已经发送过一次请求，则表示认证失败。

##### 403
该状态码表示对请求资源的访问被服务器拒绝了。（服务器没有必要给出详细的拒绝理由）

**区别**

401返回的含义主要是**要么没有经过身份验证，要么验证不正确**这算是暂时的，服务器要求再试一次

403则更多的表示拒绝的含义，不同于401，它要表达的意思是“服务器知道请求端是谁，也相信它的身份，但是拒绝访问（一般原因是权限问题）”