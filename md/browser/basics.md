## 1.跨域问题

###### 跨域问题出现的原因

跨域问题是这是浏览器为了安全实施的同源策略导致的，同源策略限制了来自不同源的document、脚本，同源的意思就是**两个URL的域名、协议、端口要完全相同**（这里要注意，域名和域名对应的ip，也是存在跨域问题的）。

###### 同一个主域名不同的二级域名是否存在跨域问题？

存在，例如域名是 a.xx.com 和 b.xx.com 如果一个页面中引入多个iframe，要想能够操作所有iframe，必须都得设置相同domain。

如果iframe的时候 a包含b 为了a和b的站点都能起作用，最好是都在脚本里面加入一句

    <script type="text/javascript">   
      document.domain = 'xxx.com';  
    </script> 