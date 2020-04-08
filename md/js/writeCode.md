## Promise/Generator/Async实现原理解析

Promise调用流程

1. Promise的构造方法接收一个executor()，在new Promise()时就立刻执行这个executor回调
2. executor()内部的异步任务被放入宏/微任务队列，等待执行
3. then()被执行，收集成功/失败回调，放入成功/失败队列
4. executor()的异步任务被执行，触发resolve/reject，从成功/失败队列中取出回调依次执行

以上流程就是观察者模式，**收集依赖 -> 触发通知 -> 取出依赖执行**，在Promise里，执行顺序是**then收集依赖 -> 异步触发resolve -> resolve执行依赖**

