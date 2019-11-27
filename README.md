
# Engine.IO: 实时引擎

[![Build Status](https://travis-ci.org/socketio/engine.io.svg?branch=master)](http://travis-ci.org/socketio/engine.io)
[![NPM version](https://badge.fury.io/js/engine.io.svg)](http://badge.fury.io/js/engine.io)

`Engine.IO` is the implementation of transport-based
cross-browser/cross-device bi-directional communication layer for
[Socket.IO](http://github.com/socketio/socket.io).


## 实现规范 ：https://github.com/socketio/engine.io-protocol


## 如何使用

### Server 服务端

#### (A) 监听一个端口

```js
var engine = require('engine.io');
var server = engine.listen(80);

server.on('connection', function(socket){
  socket.send('utf 8 string');
  socket.send(Buffer.from([0, 1, 2, 3, 4, 5])); // binary data
});
```

#### (B) 拦截对http.Server 的请求

```js
var engine = require('engine.io');
var http = require('http').createServer().listen(3000);
var server = engine.attach(http);

server.on('connection', function (socket) {
  socket.on('message', function(data){ });
  socket.on('close', function(){ });
});
```

#### (C) 传入请求

```js
var engine = require('engine.io');
var server = new engine.Server();

server.on('connection', function(socket){
  socket.send('hi');
});

// …
httpServer.on('upgrade', function(req, socket, head){
  server.handleUpgrade(req, socket, head);
});
httpServer.on('request', function(req, res){
  server.handleRequest(req, res);
});
```

### Client 客户端

```html
<script src="/path/to/engine.io.js"></script>
<script>
  var socket = new eio.Socket('ws://localhost/');
  socket.on('open', function(){
    socket.on('message', function(data){});
    socket.on('close', function(){});
  });
</script>
```

有关客户端的更多信息，请参阅
[engine-client](http://github.com/learnboost/engine.io-client) repo.

## 它有什么特性?

- **最大可靠性**. 即使存在以下情况也会建立连接:
  - 代理和负载均衡器
  - 个人防火墙和防病毒软件
  - 有关更多信息，请参阅 **Goals** 和 **Architecture** 部分
- **最小客户端规模** 辅助:
  - 闪存传输的延迟加载
  - 缺乏冗余传输的
- **可扩展性**
  - 负载均衡器友好
- **经得起考验**
- **100% Node.JS 核心风格**
  - 没有API语法糖 (留给更高层次的项目)
  - 用可读的JavaScript编写

## API

### Server 服务端

<hr><br>

#### 高等级

这些又`require('engine.io')`暴露出去:

##### 事件

- `flush`
    - 在刷新socket缓冲区时调用。
    - **Arguments**
      - `Socket`: 正在刷新socket
      - `Array`: 写buffer
- `drain`
    - 当套接字缓冲区被释放时调用
    - **Arguments**
      - `Socket`: 正在刷新socket

##### 属性

- `protocol` _(Number)_: protocol revision number
- `Server`: Server class constructor
- `Socket`: Socket class constructor
- `Transport` _(Function)_: transport constructor
- `transports` _(Object)_: 可用的传输通道映射

##### 方法

- `()`
    - 返回新的 `Server` 实例. 如果第一个参数是 `http.Server`，则新的
      `Server` 实例会附加到它上面，否则，这个参数会直接到`Server`的构造函数里去
    - **Parameters**
      - `http.Server`: 可选, 要附加的server
      - `Object`: 可选, 选项对象 (见 `Server#constructor` api文档)

  以下是实例化服务器然后附加服务器的相同方法。

```js
var httpServer; // previously created with `http.createServer();` from node.js api.

// create a server first, and then attach
var eioServer = require('engine.io').Server();
eioServer.attach(httpServer);

// or call the module as a function to get `Server`
var eioServer = require('engine.io')();
eioServer.attach(httpServer);

// immediately attach
var eioServer = require('engine.io')(httpServer);

// with custom options
var eioServer = require('engine.io')(httpServer, {
  maxHttpBufferSize: 1e3
});
```

- `listen`
    - Creates an `http.Server` which listens on the given port and attaches WS
      to it. It returns `501 Not Implemented` for regular http requests.
    - **Parameters**
      - `Number`: port to listen on.
      - `Object`: optional, options object
      - `Function`: callback for `listen`.
    - **Options**
      - All options from `Server.attach` method, documented below.
      - **Additionally** See Server `constructor` below for options you can pass for creating the new Server
    - **Returns** `Server`

```js
var engine = require('engine.io');
var server = engine.listen(3000, {
  pingTimeout: 2000,
  pingInterval: 10000
});

server.on('connection', /* ... */);
```

- `attach`
    - Captures `upgrade` requests for a `http.Server`. In other words, makes
      a regular http.Server WebSocket-compatible.
    - **Parameters**
      - `http.Server`: server to attach to.
      - `Object`: optional, options object
    - **Options**
      - All options from `Server.attach` method, documented below.
      - **Additionally** See Server `constructor` below for options you can pass for creating the new Server
    - **Returns** `Server` a new Server instance.

```js
var engine = require('engine.io');
var httpServer = require('http').createServer().listen(3000);
var server = engine.attach(httpServer, {
  wsEngine: 'uws' // requires having uws as dependency
});

server.on('connection', /* ... */);
```

#### Server 服务端

主要的 server/manager. 继承自EventEmitter

##### Events 事件

- `connection`
    - 建立新连接时触发。
    - **Arguments**
      - `Socket`: 一个 Socket 对象

##### 属性

**重要**: 如果您打算以可扩展的方式使用Engine.IO，请
请记住，以下属性仅反映连接的客户端到一个过程

- `clients` _(Object)_: 按ID连接的客户端的hash
- `clientsCount` _(Number)_: 连接的客户端数

##### 方法

- **constructor 构造函数**
    - 初始化 server
    - **Parameters 参数**
      - `Object`: 可选, options object
    - **Options 选项**
      - `pingTimeout` (`Number`): 没有pong 多少毫秒考虑连接已关闭(`5000`)
      - `pingInterval` (`Number`): 发送新的ping之前需要多少毫秒 (`25000`)
      - `upgradeTimeout` (`Number`): 取消未完成的传输升级之前需要多少毫秒 (`10000`)
      - `maxHttpBufferSize` (`Number`): 一条消息多少字节或字符，可以，在关闭会话之前（避免DoS）。 默认值是 `10E7`.
      - `allowRequest` (`Function`): 该函数接收给定的握手或升级请求作为其第一个参数，并可以决定是否继续。 第二个参数是需要使用已确定的信息调用的函数：`fn(err, success)`，其中`success`是布尔值，其中false表示请求被拒绝，而err是错误代码。
      - `transports` (`<Array> String`): 运输以允许连接到 (`['polling', 'websocket']`)
      - `allowUpgrades` (`Boolean`): 是否允许运输升级(`true`)
      - `perMessageDeflate` (`Object|Boolean`): WebSocket permessage-deflate扩展的参数
        (see [ws module](https://github.com/einaros/ws) api 文档). 设置 `false` 为禁用 (`true`)
        - `threshold` (`Number`): data is compressed only if the byte size is above this value (`1024`)
      - `httpCompression` (`Object|Boolean`): parameters of the http compression for the polling transports
        (see [zlib](http://nodejs.org/api/zlib.html#zlib_options) api docs). Set to `false` to disable. (`true`)
        - `threshold` (`Number`): data is compressed only if the byte size is above this value (`1024`)
      - `cookie` (`String|Boolean`): name of the HTTP cookie that
        contains the client sid to send as part of handshake response
        headers. Set to `false` to not send one. (`io`)
      - `cookiePath` (`String|Boolean`): path of the above `cookie`
        option. If false, no path will be sent, which means browsers will only send the cookie on the engine.io attached path (`/engine.io`).
        Set false to not save io cookie on all requests. (`/`)
      - `cookieHttpOnly` (`Boolean`): If `true` HttpOnly io cookie cannot be accessed by client-side APIs, such as JavaScript. (`true`) _This option has no effect if `cookie` or `cookiePath` is set to `false`._
      - `wsEngine` (`String`): what WebSocket server implementation to use. Specified module must conform to the `ws` interface (see [ws module api docs](https://github.com/websockets/ws/blob/master/doc/ws.md)). Default value is `ws`. An alternative c++ addon is also available by installing `uws` module.
      - `initialPacket` (`Object`): an optional packet which will be concatenated to the handshake packet emitted by Engine.IO.
- `close`
    - Closes all clients
    - **Returns** `Server` for chaining
- `handleRequest`
    - Called internally when a `Engine` request is intercepted.
    - **Parameters**
      - `http.IncomingMessage`: a node request object
      - `http.ServerResponse`: a node response object
    - **Returns** `Server` for chaining
- `handleUpgrade`
    - Called internally when a `Engine` ws upgrade is intercepted.
    - **Parameters** (same as `upgrade` event)
      - `http.IncomingMessage`: a node request object
      - `net.Stream`: TCP socket for the request
      - `Buffer`: legacy tail bytes
    - **Returns** `Server` for chaining
- `attach`
    - Attach this Server instance to an `http.Server`
    - Captures `upgrade` requests for a `http.Server`. In other words, makes
      a regular http.Server WebSocket-compatible.
    - **Parameters**
      - `http.Server`: server to attach to.
      - `Object`: optional, options object
    - **Options**
      - `path` (`String`): name of the path to capture (`/engine.io`).
      - `destroyUpgrade` (`Boolean`): destroy unhandled upgrade requests (`true`)
      - `destroyUpgradeTimeout` (`Number`): milliseconds after which unhandled requests are ended (`1000`)
      - `handlePreflightRequest` (`Boolean|Function`): whether to let engine.io handle the OPTIONS requests. You can also pass a custom function to handle the requests (`true`)
- `generateId`
    - Generate a socket id.
    - Overwrite this method to generate your custom socket id.
    - **Parameters**
      - `http.IncomingMessage`: a node request object
  - **Returns** A socket id for connected client.

<hr><br>

#### Socket

client 继承 EventEmitter

##### Events 事件

- `close`
    - 客户端断开连接时触发。
    - **Arguments**
      - `String`: reason for closing
      - `Object`: description object (optional)
- `message`
    - Fired when the client sends a message.
    - **Arguments**
      - `String` or `Buffer`: Unicode string or Buffer with binary contents
- `error`
    - Fired when an error occurs.
    - **Arguments**
      - `Error`: error object
- `flush`
    - Called when the write buffer is being flushed.
    - **Arguments**
      - `Array`: write buffer
- `drain`
    - Called when the write buffer is drained
- `packet`
    - Called when a socket received a packet (`message`, `ping`)
    - **Arguments**
      - `type`: packet type
      - `data`: packet data (if type is message)
- `packetCreate`
    - Called before a socket sends a packet (`message`, `pong`)
    - **Arguments**
      - `type`: packet type
      - `data`: packet data (if type is message)

##### 属性

- `id` _(String)_: unique identifier
- `server` _(Server)_: engine parent reference
- `request` _(http.IncomingMessage)_: request that originated the Socket
- `upgraded` _(Boolean)_: whether the transport has been upgraded
- `readyState` _(String)_: opening|open|closing|closed
- `transport` _(Transport)_: transport reference

##### 方法

- `send`:
    - Sends a message, performing `message = toString(arguments[0])` unless
      sending binary data, which is sent as is.
    - **Parameters**
      - `String` | `Buffer` | `ArrayBuffer` | `ArrayBufferView`: a string or any object implementing `toString()`, with outgoing data, or a Buffer or ArrayBuffer with binary data. Also any ArrayBufferView can be sent as is.
      - `Object`: optional, options object
      - `Function`: optional, a callback executed when the message gets flushed out by the transport
    - **Options**
      - `compress` (`Boolean`): whether to compress sending data. This option might be ignored and forced to be `true` when using polling. (`true`)
    - **Returns** `Socket` for chaining
- `close`
    - Disconnects the client
    - **Returns** `Socket` for chaining

### Client 客户端

<hr><br>

Exposed in the `eio` global namespace (in the browser), or by
`require('engine.io-client')` (in Node.JS).

For the client API refer to the
[engine-client](http://github.com/learnboost/engine.io-client) repository.

## Debug / logging

Engine.IO is powered by [debug](http://github.com/visionmedia/debug).
In order to see all the debug output, run your app with the environment variable
`DEBUG` including the desired scope.

To see the output from all of Engine.IO's debugging scopes you can use:

```
DEBUG=engine* node myapp
```

## Transports 传输通道

- `polling`: XHR / JSONP polling transport.
- `websocket`: WebSocket transport.

## Plugins 插件

- [engine.io-conflation](https://github.com/EugenDueck/engine.io-conflation): Makes **conflation and aggregation** of messages straightforward.

## Support 支持

The support channels for `engine.io` are the same as `socket.io`:
  - irc.freenode.net **#socket.io**
  - [Google Groups](http://groups.google.com/group/socket_io)
  - [Website](http://socket.io)

## Development 开发

要贡献补丁，运行测试或基准测试，请确保克隆存储库：

```
git clone git://github.com/LearnBoost/engine.io.git
```

Then:

```
cd engine.io
npm install
```

## Tests 测试

Tests run with `npm test`. It runs the server tests that are aided by
the usage of `engine.io-client`.

Make sure `npm install` is run first.

## 目标

 `Engine` 的主要目标是确保最可靠的实时通信。与以前的Socket.IO内核不同，它总是建立长轮询
首先连接，然后尝试升级到经过“测试”的更好的传输旁边。

在Socket.IO项目的整个生命周期中，我们发现了无数缺点,依靠“ HTML5 WebSocket”或“ Flash Socket”作为第一个连接机制。

两者显然都是建立双向通信的“正确方法”，HTML5 WebSocket是未来的方式。 但是，要回答大多数业务需求，替代传统的HTTP 1.1机制与交付一样好相同的解决方案。

基于WebSocket的连接有两个基本好处：

1. **更好的服务器性能**
  - _A: 加载均衡器_<br>
      负载平衡长轮询连接构成了严重的架构梦night，因为用户代理可以从任意数量的打开套接字发出请求，但是它们都需要路由到拥有**引擎**连接的进程和计算机。 这会对RAM和CPU使用率产生负面影响。
  - _B: 网络流量_<br>
     WebSocket的设计前提是每个消息帧必须被最少的数据包围。 在HTTP 1.1传输中，每个消息帧都被HTTP标头和分块的编码帧包围。 如果您尝试通过xhr-polling发送消息_“ Hello world” _，则该消息最终将变得比使用WebSocket发送时更大。
  - _C: 轻量解析器_<br>
     由于**B**的影响，在使用传统的HTTP请求时（如在长轮询中），服务器必须做更多的工作来解析网络数据并找出消息。 这意味着WebSocket的另一个优势是更少的服务器CPU使用率。

2. **更好的用户体验**

    由于第 **1** 点中所述的原因，能够建立WebSocket连接的最重要的效果是原始数据传输速度，在某些情况下可以转化为更好的用户体验

    具有大量实时交互的应用程序（例如游戏）将受益匪浅，而实时聊天（Gmail / Facebook），新闻流（Facebook）或时间轴（Twitter）等应用程序的用户体验改善将可忽略不计。

话虽如此，到目前为止，尝试直接建立WebSocket连接已经证明有问题：

1. **代理**<br>
    许多公司代理阻止WebSocket通信

2. **个人防火墙和防病毒软件**<br>
    经过我们的研究，我们发现至少3个人安全应用程序阻止WebSocket通信。

3. **云应用平台**<br>
    像Heroku或No.de这样的平台在跟上WebSocket协议发展的快节奏本质方面遇到了麻烦。 因此，应用程序不可避免地要使用长时间轮询，但是我们一直在努力的Socket.IO无缝安装体验（_“ require（）它并能正常工作” _）消失了。


其中一些问题有解决方案。 但是，对于代理和个人程序，解决方案很多时候都涉及软件升级。 经验表明，依靠客户端软件升级来提供业务解决方案是徒劳的：该项目的存在与用户代理分发的零散全景有关，客户端与最新版本的最新用户代理（Chrome，Firefox和 Safari），但其他版本低于
IE 5.5。

从用户的角度来看，不成功的WebSocket连接可能至少需要等待10秒钟的时间才能转换为等待实时应用程序开始交换数据。 这**察觉**地伤害了用户体验。

总而言之，**Engnie**首先关注可靠性和用户体验，其次是潜在的UX改进，其次是提高服务器性能。 “引擎”是从WebSocket狂暴汲取的所有教训的结果。

## Architecture 建筑

`Engine` 主要前提, 它存在的核心是即时交换传输的能力。 连接从xhr-polling开始，但是可以切换到WebSocket。

造成的跟问题是：如何在不丢失消息的情况下切换传输？

`Engine` 仅在轮询之间从轮询切换到另一个传输周期。 由于服务器在没有活动的某个超时后关闭连接，并且轮询传输实现会在连接之间缓冲消息，因此可以确保没有消息丢失和最佳性能。


这种设计的另一个好处是，我们可以解决**Flash Socket**的几乎所有局限性，例如连接速度慢，文件大小增加（我们可以安全地延迟加载它而不会损害用户体验）等。


## FAQ 文档

### 我可以使用没有Socket.IO的引擎吗

可以。尽管推荐用于构建实时应用程序的框架
是Socket.IO，因为它为实际应用提供了基本功能
例如多路传输，重新连接支持等。

`Engine` 是Socket.IO，而Connect是Express。 建筑必不可少的一块
实时框架，但是您_可能_不会将其用于构建实际应用。

### 服务器为客户服务吗？

不是。主要原因是`Engine`与框架捆绑在一起,Socket.IO包含`Engine`，因此不必为两个客户端提供服务。 如果
您使用Socket.IO，包括

```html
<script src="/socket.io/socket.io.js">
```

你覆盖了吗？

### 我可以用其他语言实现`Engine`吗？

可以. The [engine.io-protocol](https://github.com/LearnBoost/engine.io-protocol)
repo 包含规范的最新描述任何时候都可以使用JavaScript解析器实现。

## License

(The MIT License)

Copyright (c) 2014 Guillermo Rauch &lt;guillermo@learnboost.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
