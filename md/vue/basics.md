# VUE

## MVVM框架

## vue、react、angular优缺点

## vue中的Virtual DOM(虚拟DOM),虚拟DOM的好处

减少了同一时间内的页面多处内容修改所触发的浏览器reflow和repaint的次数，可能把多个不同的DOM操作集中减少到了几次甚至一次，优化了触发浏览器reflow和repaint的次数。

## vue声明周期

beforeCreate之前主要是初始化事件和初始化生命周期

1. **beforeCreate**

在beforeCreate和created之间，进行数据观测(data observer) ，也就是在这个时候开始监控data中的数据变化了

2。 **created**

对于created钩子函数和beforeMount之间，执行的操作是：

首先系统会判断对象中有没有el选项，有el选项，则继续编译过程，没有el选项，则停止编译，也意味着暂时停止了生命周期，直到vm.$mount(el)，如果el选项填写且正确的时候，生命周期将正常进行。而当将el去掉时，生命周期的钩子函数执行到created就结束了，当我们不加el选项，但是手动执行vm.$mount(el)方法的话，也能够使暂停的生命周期进行下去。

接下来，如果Vue实例对象中有template参数选项，则将其作为模板编译成render函数。如果没有template参数选项，则将外部的HTML作为模板编译（template），也就是说，template参数选项的优先级要比外部的HTML高（同时具备的话，以template参数为准）。如果既没有template选项，又没有外部HTML,则直接报错。

注意在VUE中有render函数这个选项，它以createElement作为参数，做渲染操作，当然你也可以不调用createElement，而直接嵌入JSX，优先级比template参数更高

3. **beforeMount**

正因为render函数和template选项的“优先级”比外部HTML更高，所以最后一般最后会存在一个外部HTML模板被Vue实例本身配置的模板所“替代”的过程。

4. **mounted**

5. **beforeUpdate**

在Vue中，数据更改会导致虚拟 DOM 重新渲染，并先后调用beforeUpdate钩子函数和updated钩子函数。

重渲染（调用这两个钩子函数）的前提是被更改的数据已经被写入模板中！！（这点很重要），如果更改的数据并没有被写入模版中，那么不会出发重渲染，也就不会调用两个钩子函数

6. **updated**

7. **beforeDestory**

beforeDestroy钩子函数在实例销毁之前调用。在这一步，实例仍然完全可用。

destroyed钩子函数在Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

8. **destoryed**

【注意】就如同调用在Vue实例上调用mounted会使暂停的生命周期继续一样，调用mounted会使暂停的生命周期继续一样，调用destroy()会直接销毁实例

关于vue声明周期几个需要注意的地方:

* beforeCreate，在实例初始化之后，此时正在进行数据观测和事件初始化，此时，data,watcher,methods统统没有。此时的vue实例还什么都没有，但是$route对象是存在的，可以根据路由信息进行重定向之类的操作。

* created：在实例已经创建完成之后被调用。在这一步，实例已完成以下配置：数据观测(data observer) ，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没开始，**\$el属性目前不可见。即已经有了data,但是el没有**

* beforeMount：在挂载开始之前被调用，相关的 render 函数 首次被调用。但是render正在执行中，**此时DOM还是无法操作的。**我打印了此时的vue实例对象，相比于created生命周期，此时只是多了一个\$el的属性，然而其值为undefined。
使用场景我上文已经提到了，页面渲染时所需要的数据，应尽量在这之前完成赋值。

* mounted：在挂载之后被调用。在这一步 创建vm.\$el并替换el，并挂载到实例上。（官方文档中的 “如果root实例挂载了一个文档内元素，当mounted被调用时vm.\$el也在文档内” 这句话存疑）
此时元素已经渲染完成了，依赖于DOM的代码就放在这里吧~比如监听DOM事件。

* beforeUpdate：\$vm.data更新之后，虚拟DOM重新渲染 和打补丁之前被调用。你可以在这个钩子中进一步地修改$vm.data，这不会触发附加的重渲染过程。

* updated：虚拟DOM重新渲染 和打补丁之后被调用。当这个钩子被调用时，组件DOM的data已经更新，所以你现在可以执行依赖于DOM的操作。但是不要在此时修改data，否则会继续触发beforeUpdate、updated这两个生命周期，进入死循环！

* beforeDestroy：实例被销毁之前调用。在这一步，实例仍然完全可用。

  **注意：路由跳转的时候会触发beforeDestory和destoryed，但是如果对router-view加了keep-alive，就会导致组件被缓存，路由跳转并不会触发beforeDestroy和destoryed**

* destroyed：Vue实例销毁后调用。此时，Vue实例指示的所有东西已经解绑定，所有的事件监听器都已经被移除，所有的子实例也已经被销毁。

**注：beforeMount、mounted、beforeUpdate、updated、beforeDestroy、destroyed这几个钩子函数，在服务器端渲染期间不被调用。**

## Vuex
Vuex是vue中集中管理状态的机制。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。有以下作用：让多个vue组件中共享状态，可以实现组件中通讯。

Vuex的状态存储是响应式的，当Vue组件从store中读取状态时，若store中状态发生改变，响应的组件也会得到更新状态。但不能直接改变state,必须通过显示的提交(commit)mutations来追踪每一个状态的变化。

**Vuex存储在内存当中**主要用于组件之间的传值，用来做状态管理，当刷新页面（这里的刷新页面指的是 --> F5刷新,属于清除内存了）时vuex存储的值会丢失，可以结合sessionStorage解决此类问题。（选择sessionStorage的原因是,vue是单页面应用，操作都是在一个页面跳转路由，另一个原因是sessionStorage可以保证打开页面时sessionStorage的数据为空，而如果是localStorage则会读取上一次打开页面的数据。）

其实，我们只需要在点击页面刷新时先将state数据保存到sessionStorage,然后才真正刷新页面，之后再将保存在sessionStorage中的数据取出放入Vuex中即可。

beforeunload这个事件在页面刷新时先触发的,我们可以在app.vue这个入口组件中进行监听，这样就可以保证每次刷新页面都可以触发。

```javascript
export default {
  name: 'App',
  created () {
    //在页面加载时读取sessionStorage里的状态信息
    if (sessionStorage.getItem("store") ) {
        this.$store.replaceState(Object.assign({}, this.$store.state,JSON.parse(sessionStorage.getItem("store"))))
    } 

    //在页面刷新时将vuex里的信息保存到sessionStorage里
    window.addEventListener("beforeunload",()=>{
        sessionStorage.setItem("store",JSON.stringify(this.$store.state))
    })
  }
}
```

## Vue2.0怎样实现双向绑定

## Vue3.0怎样实现双向绑定

#### 简述Vu2.0和Vue3.0实现双向绑定的区别

#### 使用过哪些vue全家桶，解决了什么问题

## Vue 的diff算法

## Vue-router

#### vue-router中hash路由和history路由、区别

#### Vue 计算属性和 watch 在什么场景下使用

## Vue 的 nexttick 实现的原理







