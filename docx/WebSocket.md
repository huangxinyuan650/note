## Websocket
### 一. 概念
为了实现推动技术,传统的使用ajax轮询,设置特定的间隔时间由浏览器不断向服务器发送请求,获取到服务器返回的最新数据.消耗资源过多.
WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。websocket允许服务端主动向客户端推送数据,浏览器和服务器只需完成一次握手,两者之间就可以创建持久性的连接,并进行双向数据传输.
为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息"Upgrade: WebSocket"表明这是一个申请协议升级的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。
### 二. 前端调用方法
1.  创建一个websocket对象,并向后端发送连接请求
Var ws = new websocket(url,[protocol,] ) 
第一个参数为连接的url,第二个参数是可选的,指定了可接受的子协议
2. Websocket事件
Ws.onopen():连接建立时触发
Ws.onmessage() :客户端接受服务端数据时触发
Ws.onerror(): 通信发生错误时触发
Ws.onclose():连接关闭时触发
3.  Websocket方法
Ws.send()使用连接发送数据
Ws.close()关闭连接
### 三. 后端调用方法(tornado)
Tornado提供支持WebSocket的模块是tornado.websocket，其中提供了一个WebSocketHandler类用来处理通讯。
1. Websockethandler.open():当一个websocket连接建立后被调用
2. Web.sockethandler.on_message(message):当客户端发动消息过来时被调用,此方法必须被重写
3. -Web.sockethandler.on_close():当websocket连接关闭后被调用
4. Websockethandle.write_message(message,binary= False): 向客户端发送消息messagea，message可以是字符串或字典（字典会被转为json字符串）。若binary为False，则message以utf8编码发送；二进制模式（binary=True）时，可发送任何字节码。
5. Websockethandler.close():关闭websocket连接
6. WebSocketHandler.check_origin(origin)：判断源origin，对于符合条件（返回判断结果为True）的请求源origin允许其连接，否则返回403。可以重写此方法来解决WebSocket的跨域请求（如始终return True）。
注意：WebSocket可以共用Http对端口的监听和路由的配置。