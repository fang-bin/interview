### 跨域问题

#### 跨域问题出现的原因

跨域问题是这是浏览器为了安全实施的同源策略导致的，同源策略限制了来自不同源的document、脚本，同源的意思就是**两个URL的域名、协议、端口要完全相同**（这里要注意，域名和域名对应的ip，也是存在跨域问题的）。

#### 同一个主域名不同的二级域名是否存在跨域问题？

存在，例如域名是 a.xx.com 和 b.xx.com 如果一个页面中引入多个iframe，要想能够操作所有iframe，必须都得设置相同domain。

如果iframe的时候 a包含b 为了a和b的站点都能起作用，最好是都在脚本里面加入一句

    <script type="text/javascript">   
      document.domain = 'xxx.com';  
    </script> 


### 浏览器渲染机制

#### 常见浏览器内核

| 浏览器 | IE | FireFox | Chrome Opera | Safari |
| --- | --- | --- | --- | --- |
| 内核 |  Trident | Gecko | Blink | Webkit |

浏览器内核分为两个部分: 渲染引擎和JS引擎

### 浏览器缓存策略

##### 浏览器缓存位置和优先级

1. **Service Worker**
    Service Worker 是浏览器在后台独立于网页运行的脚本，其无法直接访问Dom，可以通过 postMessage 接口发送消息来与其控制的页面通信，可以在这个线程中缓存文件（可以通过编辑拦截请求和返回，控制缓存的规则），并且该缓存是持续性的

2. **Memory Cache(内存缓存)**
    内存缓存不是持续性的，缓存会随着进程释放而释放

3. **Disk Cache(硬盘缓存)**
    持续性缓存，并且容量更优，它会根据HTTP header的字段判断哪些资源需要缓存

4. **Push Cache(推送缓存)**
    推送缓存是HTTP/2的内容，目前应用较少

##### 强缓存（不需要向服务器询问的缓存）

###### Expires
过期时间，会在设置的时间之后失效

###### Cache-Control
HTTP/1.1新增字段，`Cache-Control`可以通过`max-age`设置过期时间，如果设置`Cache-Control: no-cache，no-store, must-revalidate`；浏览器则不会做任何缓存。`Cache-Control`会覆盖`Expires`的设置(它会覆盖其他的一切缓存规则)

##### 协商缓存（需要向服务器询问缓存是否已经过期）
###### Last-Modified
最后修改的时间，浏览器第一次请求资源时，服务器会在响应头上加 `Last-Modified`，当浏览器再次加载该资源时，浏览器会在请求头中带上`If-Modified-Since`字段，值就是`Last-Modified`的值，服务器会比对两个时间，若相同则返回304，否则返回新资源，并更新`Last-Modified`。

###### ETag
HTTP/1.1新增字段，表示资源唯一表示，只要资源改动，就会重新计算`ETag`，缓存流程和`Last-Modified`一样，只是它是想服务器发送请求头中带`If-none-Match`字段，如果`ETag`字段不匹配，说明资源已经改变，返回新资源并更新`ETag`，若匹配则返回304。

**ETag 比 Last-Modified 更准确**:打开资源但是并未更改，Last-Modified也会更改，并且Last-Modified的单位时间为秒，如果更改文件的一秒内请求资源，还会命中缓存。
**如果什么缓存策略都没有设置，那么浏览器会取响应头中的Date减去Last-Modified值的10%做为缓存时间**



