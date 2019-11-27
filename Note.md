## engine.io 对象

```js
{ [Function]
  protocol: 1,
  Server:
   { [Function: Server]
     errors:
      { UNKNOWN_TRANSPORT: 0,
        UNKNOWN_SID: 1,
        BAD_HANDSHAKE_METHOD: 2,
        BAD_REQUEST: 3,
        FORBIDDEN: 4 },
     errorMessages:
      { '0': 'Transport unknown',
        '1': 'Session ID unknown',
        '2': 'Bad handshake method',
        '3': 'Bad request',
        '4': 'Forbidden' },
     super_:
      { [Function: EventEmitter]
        EventEmitter: [Circular],
        usingDomains: true,
        defaultMaxListeners: [Getter/Setter],
        init: [Function],
        listenerCount: [Function] } },
  Socket:
   { [Function: Socket]
     super_:
      { [Function: EventEmitter]
        EventEmitter: [Circular],
        usingDomains: true,
        defaultMaxListeners: [Getter/Setter],
        init: [Function],
        listenerCount: [Function] } },
  Transport:
   { [Function: Transport]
     super_:
      { [Function: EventEmitter]
        EventEmitter: [Circular],
        usingDomains: true,
        defaultMaxListeners: [Getter/Setter],
        init: [Function],
        listenerCount: [Function] } },
  transports:
   { polling: { [Function: polling] upgradesTo: [Array] },
     websocket: { [Function: WebSocket] super_: [Function] } },
  parser:
   { protocol: 3,
     packets:
      { open: 0,
        close: 1,
        ping: 2,
        pong: 3,
        message: 4,
        upgrade: 5,
        noop: 6 },
     encodePacket: [Function],
     encodeBase64Packet: [Function],
     decodePacket: [Function],
     decodeBase64Packet: [Function],
     encodePayload: [Function],
     decodePayload: [Function],
     encodePayloadAsBinary: [Function],
     decodePayloadAsBinary: [Function] },
  listen: [Function: listen],
  attach: [Function: attach] }

```

## engine()

```js
Server {
  clients: {},
  clientsCount: 0,
  wsEngine: 'ws',
  pingTimeout: 5000,
  pingInterval: 25000,
  upgradeTimeout: 10000,
  maxHttpBufferSize: 100000000,
  transports: [ 'polling', 'websocket' ],
  allowUpgrades: true,
  allowRequest: undefined,
  cookie: 'io',
  cookiePath: '/',
  cookieHttpOnly: true,
  perMessageDeflate: { threshold: 1024 },
  httpCompression: { threshold: 1024 },
  initialPacket: undefined,
  ws:
   WebSocketServer {
     domain:
      Domain {
        domain: null,
        _events: [Object],
        _eventsCount: 3,
        _maxListeners: undefined,
        members: [] },
     _events: {},
     _eventsCount: 0,
     _maxListeners: undefined,
     options:
      { maxPayload: 100000000,
        perMessageDeflate: [Object],
        handleProtocols: null,
        clientTracking: false,
        verifyClient: null,
        noServer: true,
        backlog: null,
        server: null,
        host: null,
        path: null,
        port: null } } }


```