# peopleswarm
A distributed networking stack for connecting peers.

Peropleswarm is a high-level API for finding and connecting to peers who are interested in a "topic." 


## Installation
```
npm install peopleswarm
```

## Usage
In #1 peer: 
```js
//swarm1.js


const Swarm = require('peopleswarm')

start()

async function start () {
  const swarm1 = new Swarm({ seed: Buffer.alloc(32).fill(4) })
  
  console.log('SWARM 1 KEYPAIR:', swarm1.keyPair)
  
  swarm1.on('connection', function (connection, info) {
    console.log('swarm 1 got a server connection:', connection.remotePublicKey, connection.publicKey, connection.handshakeHash)
    connection.on('error', err => console.error('1 CONN ERR:', err))
    // Do something with `connection`
    console.log('new connection ', info)

    // `info` is a PeerInfo object
  })
  
  const key = Buffer.alloc(32).fill(7)

  const discovery1 = swarm1.join(key)
  await discovery1.flushed() // Wait for the first lookup/annnounce to complete.

  
}
```

In #2 node:
```js
//swarm2.js
const Swarm = require('hyperswarm')

start()

async function start () {
  
  const swarm2 =  new Swarm({ seed: Buffer.alloc(32).fill(5) })

  
  console.log('SWARM 2 KEYPAIR:', swarm2.keyPair)

  
  swarm2.on('connection', function (connection, info) {
    console.log('swarm 2 got a client connection:', connection.remotePublicKey, connection.publicKey, connection.handshakeHash)
    connection.on('error', err => console.error('2 CONN ERR:', err))
  })

  const key = Buffer.alloc(32).fill(7)


  swarm2.join(key)

  await swarm2.flush()
  // await discovery.destroy() // Stop lookup up and announcing this topic.
}
```

Then you can see the two node connected by the same "topic". The output looks like:

In #1 peer terminal: 

```bash
SWARM 1 KEYPAIR: {
  publicKey: <Buffer ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ec 45 30 57 5d 77 26 87 93 33 db da be 7c>,
  secretKey: <Buffer 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 04 ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ... 14 more bytes>
}
swarm 1 got a server connection: <Buffer 6e 7a 1c dd 29 b0 b7 8f d1 3a f4 c5 59 8f ef f4 ef 2a 97 16 6e 3c a6 f2 e4 fb fc cd 80 50 5b f1> <Buffer ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ec 45 30 57 5d 77 26 87 93 33 db da be 7c> <Buffer aa 11 c8 42 ba 5c 1c 51 d1 b7 ee 17 7d f0 d9 52 b5 71 f7 8d 9c dc 82 c8 6d 72 07 7e a2 60 b2 bb 01 32 c0 8f 49 f0 5b 04 a5 87 d4 70 47 be a7 61 87 11 ... 14 more bytes>
new connection  PeerInfo {
  _events: [Object: null prototype] {},
  _eventsCount: 0,
  _maxListeners: undefined,
  publicKey: <Buffer 6e 7a 1c dd 29 b0 b7 8f d1 3a f4 c5 59 8f ef f4 ef 2a 97 16 6e 3c a6 f2 e4 fb fc cd 80 50 5b f1>,
  relayAddresses: [],
  reconnecting: true,
  proven: true,
  banned: false,
  tried: false,
  explicit: false,
  queued: false,
  client: true,
  topics: [
    <Buffer 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07>
  ],
  attempts: 0,
  priority: 2,
  _index: 0,
  _flushTick: 0,
  _seenTopics: Set(1) {
    '0707070707070707070707070707070707070707070707070707070707070707'
  },
  [Symbol(kCapture)]: false
}
```

In #2 peer terminal:

```bash
swarm 2 got a client connection: <Buffer ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ec 45 30 57 5d 77 26 87 93 33 db da be 7c> <Buffer 6e 7a 1c dd 29 b0 b7 8f d1 3a f4 c5 59 8f ef f4 ef 2a 97 16 6e 3c a6 f2 e4 fb fc cd 80 50 5b f1> <Buffer 6d 34 54 7d 51 85 c2 39 76 ab 25 e7 bc f0 f6 69 d1 ac b9 e8 57 0e a5 70 4c 51 5b ba f8 ef f1 b5 4f 2f 0a 77 24 35 a2 e3 84 64 88 1e 19 82 e1 96 51 6d ... 14 more bytes>
2 CONN ERR: Error: ECONNRESET: stream destroyed by remote
    at UDXStream._onclose (/home/ubuntu/papaya/client/node_modules/udx-native/lib/stream.js:286:17) {
  errno: -104,
  code: 'ECONNRESET'
}
swarm 2 got a client connection: <Buffer ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ec 45 30 57 5d 77 26 87 93 33 db da be 7c> <Buffer 6e 7a 1c dd 29 b0 b7 8f d1 3a f4 c5 59 8f ef f4 ef 2a 97 16 6e 3c a6 f2 e4 fb fc cd 80 50 5b f1> <Buffer 48 06 68 a1 76 2a b6 c9 7b c5 f2 63 f7 76 b2 65 dd 7e a0 a4 4f 1c 56 92 59 af 92 7e df 84 ba af c0 82 f7 ed 74 94 19 f4 0d 1a 7d 43 c1 23 5a 44 97 85 ... 14 more bytes>
2 CONN ERR: Error: ECONNRESET: stream destroyed by remote
    at UDXStream._onclose (/home/ubuntu/papaya/client/node_modules/udx-native/lib/stream.js:286:17) {
  errno: -104,
  code: 'ECONNRESET'
}
swarm 2 got a client connection: <Buffer ca 93 ac 17 05 18 70 71 d6 7b 83 c7 ff 0e fe 81 08 e8 ec 45 30 57 5d 77 26 87 93 33 db da be 7c> <Buffer 6e 7a 1c dd 29 b0 b7 8f d1 3a f4 c5 59 8f ef f4 ef 2a 97 16 6e 3c a6 f2 e4 fb fc cd 80 50 5b f1> <Buffer 81 da bf 03 82 05 cb a9 a0 0b 41 c8 fa 6b b2 37 89 8d 08 2a e0 08 dd 44 1e bf 60 02 d5 12 e7 84 19 12 b2 ea 02 cf d2 e2 28 55 1e 0b b3 42 a6 fe 76 bd ... 14 more bytes>
2 CONN ERR: Error: ECONNRESET: stream destroyed by remote
    at UDXStream._onclose (/home/ubuntu/papaya/client/node_modules/udx-native/lib/stream.js:286:17) {
  errno: -104,
  code: 'ECONNRESET'
}
```





## peopleswarm API

#### `const swarm = new peopleswarm(opts = {})`
Construct a new `peopleswarm` instance.

`opts` can include:
* `keyPair`: A Noise keypair that will be used to listen/connect on the DHT. Defaults to a new key pair.
* `seed`: A unique, 32-byte, random seed that can be used to deterministically generate the key pair.
* `maxPeers`: The maximum number of peer connections to allow.
* `firewall`: A sync function of the form `remotePublicKey => (true|false)`. If true, the connection will be rejected. Defaults to allowing all connections.
* `dht`: A DHT instance. Defaults to a new instance.

#### `swarm.connections`
A set of all active client/server connections.

#### `swarm.peers`
A Map containing all connected peers, of the form: `(Noise public key hex string) -> PeerInfo object`

#### `swarm.dht`
A [`@hyperswarm/dht`](https://github.com/hyperswarm/dht) instance. Useful if you want lower-level control over peopleswarm's networking.

#### `swarm.on('connection', (socket, peerInfo) => {})`
Emitted whenever the swarm connects to a new peer.

`socket` is an end-to-end (Noise) encrypted Duplex stream

`peerInfo` is a [`PeerInfo`]instance

#### `const discovery = swarm.join(topic, opts = {})`
Start discovering and connecting to peers sharing a common topic. As new peers are connected to, they will be emitted from the swarm as `connection` events.

`topic` must be a 32-byte Buffer
`opts` can include:
* `server`: Accept server connections for this topic by announcing yourself to the DHT. Defaults to `true`.
* `client`: Actively search for and connect to discovered servers. Defaults to `true`.


#### Clients and Servers
In peopleswarm, there are two ways for peers to join the swarm: client mode and server mode. 

When you join a topic as a server, the swarm will start accepting incoming connections from clients (peers that have joined the same topic in client mode). Server mode will announce your keypair to the DHT, so that other peers can discover your server. When server connections are emitted, they are not associated with a specific topic -- the server only knows it received an incoming connection.

When you join a topic as a client, the swarm will do a query to discover available servers, and will eagerly connect to them. As with server mode, these connections will be emitted as `connection` events, but in client mode they __will__ be associated with the topic (`info.topics` will be set in the `connection` event).

#### `await swarm.leave(topic)`
Stop discovering peers for the given topic.

`topic` must be a 32-byte Buffer

If a topic was previously joined in server mode, `leave` will stop announcing the topic on the DHT. If a topic was previously joined in client mode, `leave` will stop searching for servers announcing the topic.

`leave` will __not__ close any existing connections.

#### `swarm.joinPeer(noisePublicKey)`
Establish a direct connection to a known peer.

`noisePublicKey` must be a 32-byte Buffer

As with the standard `join` method, `joinPeer` will ensure that peer connections are reestablished in the event of failures.

#### `swarm.leavePeer(noisePublicKey)`
Stop attempting direct connections to a known peer.

`noisePublicKey` must be a 32-byte Buffer

If a direct connection is already established, that connection will __not__ be destroyed by `leavePeer`.

#### `const discovery = swarm.status(topic)`
Get the [`PeerDiscovery`] object associated with the topic, if it exists.

#### `await swarm.listen()`
Explicitly start listening for incoming connections. This will be called internally after the first `join`, so it rarely needs to be called manually.

#### `await swarm.flush()`
Wait for any pending DHT announces, and for the swarm to connect to any pending peers (peers that have been discovered, but are still in the queue awaiting processing).

Once a `flush()` has completed, the swarm will have connected to every peer it can discover from the current set of topics it's managing.

`flush()` is not topic-specific, so it will wait for every pending DHT operation and connection to be processed -- it's quite heavyweight, so it could take a while. In most cases, it's not necessary, as connections are emitted by `swarm.on('connection')` immediately after they're opened.  

## PeerDiscovery API

`swarm.join` returns a `PeerDiscovery` instance which allows you to both control discovery behavior, and respond to lifecycle changes during discovery.

#### `await discovery.flushed()`
Wait until the topic has been fully announced to the DHT. This method is only relevant in server mode. When `flushed()` has completed, the server will be available to the network.

#### `await discovery.refresh({ client, server })`
Update the `PeerDiscovery` configuration, optionally toggling client and server modes. This will also trigger an immediate re-announce of the topic, when the `PeerDiscovery` is in server mode.

#### `await discovery.destroy()`
Stop discovering peers for the given topic. 

If a topic was previously joined in server mode, `leave` will stop announcing the topic on the DHT. If a topic was previously joined in client mode, `leave` will stop searching for servers announcing the topic.

## PeerInfo API

`swarm.on('connection', ...)` emits a `PeerInfo` instance whenever a new connection is established.

There is a one-to-one relationship between connections and `PeerInfo` objects -- if a single peer announces multiple topics, those topics will be multiplexed over a single connection.

#### `peerInfo.publicKey`
The peer's Noise public key.

#### `peerInfo.topics`
An Array of topics that this Peer is associated with -- `topics` will only be updated when the Peer is in client mode.

#### `peerInfo.prioritized`
If true, the swarm will rapidly attempt to reconnect to this peer.

#### `peerInfo.ban()`
Ban the peer. This will prevent any future reconnection attempts, but it will __not__ close any existing connections.

## License
MIT
