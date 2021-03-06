## 资源加载和网络栈

### 高性能网络栈

#### DNS 预取和 TCP 预连接（Preconnect）

DNS 查询和 TCP 三次握手都是比较耗费时间的操作。  
文中给出的 DNS 查询大概平均为 60~120ms，TCP 三次握手也大概是几十毫秒。  
看似不太长的操作，但是因为一个页面肯定不会只有一两个资源文件，当文件多了的情况下，数十个叠加在一起的时间就会变得很大，所以，为了减少这一部分的时间，Chrome 给出了如下两个解决方案：

1.  DNS 预取
2.  TCP 预连接

在 HTML 中，执行 DNS 预取的方式如下：

```html
<link rel="dns-prefetch" href="http://domain.com">
```

TCP 预连接则不能够通过代码进行控制，是由`Chrome`的一个追踪技术来获取用户的访问路径，进行预测下一步用户可能会触发什么超链接，如果计算得出概率较大，则会进行 TCP 预连接。

_P.S. 在 chrome 中，用户在地址栏中输入 URL 后，敲下回车之前其实已经开始进行 DNS 预取和 TCP 预连接了，可以自己本地启动一个服务查看 log 得到结果_

### 实践：高效的资源使用策略

#### DNS 和 TCP 连接

1.  减少链接的重定向，每次重定向都会导致浏览器去解析跳转后地址的 DNS
2.  利用上边提到的 DNS 预取策略（设置 meta 标签）
3.  搭建支持 SPDY 协议的服务器
4.  避免错误的链接请求，如果页面中有一些无效的链接，直接移除它，而不要留在页面中（404 也会占用资源）

#### 资源的数量

1.  每一个资源都会占用 TCP 的链接，所以可以选择将一些小型的资源（图片、css 或者 js 文件之类的，直接内嵌到 html 文件中）
2.  合并资源，现有的 webpack/gulp 就是一个很好的工具，能够合并 js 或者 css 为一个文件，极大的减少了 TCP 的连接数量

#### 资源的数据量

1.  善用缓存，将一些可预见的不太常更新的文件缓存到浏览器，下次请求直接读取缓存中的文件即可。
2.  对资源进行压缩，因为一个可读的 js 或者 css 文件，必然会有很多空行，或者注释，但是浏览器并不需要这些。所以对这些空行、注释进行删除，甚至是 js 中的混淆语法（将可读的变量名修改为`a`、`b`这样的），都能够减少资源的大小，从而优化加载速度。
