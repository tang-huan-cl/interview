# 从输⼊ URL 到⻚⾯加载发⽣了什么？

这个问题很经典，也是很多⾯试⾼级前端时必问的问题。我也在⾯试时遇到过，只不过不 同的是，⾯试官在这之前还问了⼀个问题，那就是从打开⼀个浏览器标签⻚开始，发⽣了 什么。

也就是说，考察的是⾯试者对浏览器进程与线程的认知程度。下⾯是浏览器中进程的相关 概念：

- 浏览器是多进程的
- 浏览器之所以能够运⾏，是因为系统给它的进程分配了资源（cpu、内存）
- 简单点理解，每打开⼀个 Tab ⻚，就相当于创建了⼀个独⽴的浏览器进程。

也就是说，新打开⼀个 TAB ⻚，实际上就创建了⼀个浏览器进程，但是有时候会有不 同，为了性能考虑，浏览器的优化策略会将多个空的 TAB ⻚进程合并成⼀个，在有输⼊ 内容之后才分离出来创建另⼀个新的浏览器进程。

下⾯来说说当输⼊ url 之后，到底发⽣了什么。总体来说分为以下⼏个过程:

- DNS 解析
- TCP 连接
- 发送 HTTP 请求
- 服务器处理请求并返回 HTTP 报⽂
- 浏览器解析渲染⻚⾯
- 连接结束

## DNS 解析

这个过程实际上是浏览器将输⼊的 url 发送到 DNS 服务器进⾏查询，DNS 服务器会返回 当前查询 url 的 IP 地址。它实际上充当了⼀个翻译的⾓⾊，实现了⽹址到 IP 地址的转换

DNS 解析是⼀个递归查询的过程。

上图中演⽰的过程经历了 8 个步骤，如果每次都是这样，必然会损耗⼤量的资源，所以 我们必须对 DNS 解析进⾏优化。

- DNS 缓存：DNS 存在着多级缓存，从离浏览器的距离排序的话，有以下⼏种: 浏览器缓存，系统缓存，路由器缓存，IPS 服务器缓存，根域名服务器缓存，顶级域名服务器缓存，主域名服务器缓存

- DNS 负载均衡：DNS 可以返回⼀个合适的机器的 IP 给⽤户，例如可以根据每台机 器的负载量，该机器离⽤户地理位置的距离等等，这种过程就是 DNS 负载均衡，⼜ 叫做 DNS 重定向。⼤家⽿熟能详的 CDN(Content Delivery Network) 就是利⽤ DNS 的重定向技术，DNS 服务器会返回⼀个跟⽤户最接近的点的 IP 地址给⽤户，CDN 节点的服务器负责响应⽤户的请求，提供所需的内容

## TCP 连接

HTTP 协议是使⽤ TCP 作为其传输层协议的，当 TCP 出现瓶颈时，HTTP 也会受到影 响。HTTP 报⽂是包裹在 TCP 报⽂中发送的，服务器端收到 TCP 报⽂时会解包提取出
HTTP 报⽂。但是这个过程中存在⼀定的⻛险，HTTP 报⽂是明⽂，如果中间被截取的话 会存在⼀些信息泄露的⻛险。那么在进⼊ TCP 报⽂之前对 HTTP 做⼀次加密就可以解决 这个问题了。HTTPS 协议的本质就是 HTTP + SSL(or TLS)。在 HTTP 报⽂进⼊ TCP 报 ⽂之前，先使⽤ SSL 对 HTTP 报⽂进⾏加密。从⽹络的层级结构看它位于 HTTP 协议与
TCP 协议之间。

HTTPS 在传输数据之前需要客户端与服务器进⾏⼀个握⼿ (TLS/SSL 握⼿)，在握⼿过程 中将确⽴双⽅加密传输数据的密码信息。TLS/SSL 使⽤了⾮对称加密，对称加密以及
hash 等

## HTTP 请求

发送 HTTP 请求的过程就是构建 HTTP 请求报⽂并通过 TCP 协议中发送到服务器指定端 ⼝ (HTTP 协议 80/8080, HTTPS 协议 443)。HTTP 请求报⽂是由三部分组成: 请求⾏, 请 求报头和请求正⽂。 请求⾏的格式如下：

```js
Method Request-URL HTTP-Version CRLF
```

例如：

```js
eg: GET index.html HTTP/1.1
```

常⽤的⽅法有: GET, POST, PUT, DELETE, OPTIONS, HEAD。

请求报头：请求报头允许客户端向服务器传递请求的附加信息和客户端⾃⾝的信息。常⻅ 的请求报头有: Accept , Accept-Charset , Accept-Encoding , Accept-Language , Content-Type , Authorization , Cookie , User-Agent 等。

- Accept ⽤于指定客户端⽤于接受哪些类型的信息;
- Accept-Encoding 与 Accept 类似，它⽤于 指定接受的编码⽅式;
- Connection 设置为 Keep-alive ⽤于告诉客户端本次 HTTP 请求结束 之后并不需要关闭 TCP 连接，这样可以使下次 HTTP 请求使⽤相同的 TCP 通道，节省 TCP 连接建⽴的时间。

请求正⽂： 当使⽤ POST, PUT 等⽅法时，通常需要客户端向服务器传递数据。这些数据 就储存在请求正⽂中。在请求包头中有⼀些与请求正⽂相关的信息，例如: 现在的 Web
应⽤通常采⽤ Rest 架构，请求的数据格式⼀般为 json。这时就需要设置 Content-Type: application/json 。

## 服务器处理请求并返回 HTTP 报⽂

后端从在固定的端⼝接收到 TCP 报⽂开始，这⼀部分对应于编程语⾔中的 socket。它会 对 TCP 连接进⾏处理，对 HTTP 协议进⾏解析，并按照报⽂格式进⼀步封装成 HTTP
Request 对象，供上层使⽤。这⼀部分⼯作⼀般是由 Web 服务器去进⾏
HTTP 响应报⽂也是由三部分组成:

- 状态码
- 响应报头
- 响应报⽂

状态码是由 3 位数组成，第⼀个数字定义了响应的类别，且有五种可能取值:

- 1xx：指⽰信息–表⽰请求已接收，继续处理。
- 2xx：成功–表⽰请求已被成功接收、理解、接受。
- 3xx：重定向–要完成请求必须进⾏更进⼀步的操作。
- 4xx：客户端错误–请求有语法错误或请求⽆法实现。
- 5xx：服务器端错误–服务器未能实现合法的请求。

平时遇到⽐较常⻅的状态码有: 200, 204, 301, 302, 304, 400, 401, 403, 404, 422, 500
响应报头: 常⻅的响应报头字段有: Server, Connection…。 响应报⽂: 服务器返回给浏览器的⽂本信息，通常 HTML, CSS, JS, 图⽚等⽂件就放在这 ⼀部分

## 浏览器解析渲染⻚⾯

浏览器在收到 HTML,CSS,JS ⽂件后，它是如何把⻚⾯呈现到屏幕上的 浏览器是⼀个边解析边渲染的过程。⾸先浏览器解析 HTML ⽂件构建 DOM 树，然后解 析 CSS ⽂件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上, 这个过程⽐较复杂，涉及到两个概念: reflow (回流) 和 repain (重绘)。

- reflow ： DOM 节点中的各个元素都是以盒模型的形式存在，这些都需要浏览器去计 算其位置和⼤⼩等，这个过程称为 reflow

- repain 当盒模型的位置, ⼤⼩以及其他属性，如颜⾊, 字体, 等确定下来之后，浏览器 便开始绘制内容，这个过程称为 repain

⻚⾯在⾸次加载时必然会经历 reflow 和 repain 。 reflow 和 repain 过程是⾮常消耗性能 的，尤其是在移动设备上，它会破坏⽤户体验，有时会造成⻚⾯卡顿。所以我们应该尽可 能少的减少 reflow 和 repain 。

JS 的解析是由浏览器中的 JS 解析引擎完成的。
JS 是单线程运⾏，也就是说，在同⼀个时间内只能做⼀件事，所有的任务都需要排队，前⼀个任务结束，后⼀个任务才能开始。 但是⼜存在某些任务⽐较耗时，如 IO 读写等，所以需要⼀种机制可以先执⾏排在后⾯的 任务，这就是：同步任务 (synchronous) 和异步任务(asynchronous)。

JS 的执⾏机制就可以看做是⼀个主线程加上⼀个任务队列 (task queue)。同步任务就是 放在主线程上执⾏的任务，异步任务是放在任务队列中的任务。所有的同步任务在主线程 上执⾏，形成⼀个执⾏栈; 异步任务有了运⾏结果就会在任务队列中放置⼀个事件；脚本 运⾏时先依次运⾏执⾏栈，然后会从任务队列⾥提取事件，运⾏任务队列中的任务，这个 过程是不断重复的，所以⼜叫做事件循环 (Event loop)。

浏览器在解析过程中，如果遇到请求外部资源时，如图像, iconfont,JS 等。浏览器将重复 上⾯的过程下载该资源。请求过程是异步的，并不会影响 HTML ⽂档进⾏加载，但是当 ⽂档加载过程中遇到 JS ⽂件，HTML ⽂档会挂起渲染过程，不仅要等到⽂档中 JS ⽂件 加载完毕还要等待解析执⾏完毕，才会继续 HTML 的渲染过程。原因是因为 JS 有可能修 改 DOM 结构，这就意味着 JS 执⾏完成前，后续所有资源的下载是没有必要的，这就是 JS 阻塞后续资源下载的根本原因。CSS ⽂件的加载不影响 JS ⽂件的加载，但是却影响
JS ⽂件的执⾏。JS 代码执⾏前浏览器必须保证 CSS ⽂件已经下载并加载完毕
