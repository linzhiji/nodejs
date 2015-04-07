# TLS (SSL)

    Stability: 3 - Stable

可以使用 `require('tls')` 来访问这个模块。


`tls` 模块 使用 OpenSSL 来提供传输层（Transport Layer）安全性和（或）安全套接层（Secure Socket Layer）：加密过的流通讯。

TLS/SSL 是一种公钥/私钥基础架构。每个客户端和服务端都需要一个私钥。私钥可以用以下方法创建的：

    openssl genrsa -out ryans-key.pem 2048

所有服务器和某些客户端需要证书。证书由认证中心（Certificate Authority）签名，或者自签名。获得证书第一步是创建一个证书签名请求 "Certificate Signing Request" (CSR)文件。证书可以用以下方法创建的：

    openssl req -new -sha256 -key ryans-key.pem -out ryans-csr.pem

使用CSR创建一个自签名的证书：

    openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem

或者你可以发送 CSR 给认证中心（Certificate Authority）来签名。

(TODO: 创建 CA 的文档, 感兴趣的读者可以在 Node 源码 `test/fixtures/keys/Makefile` 里查看)

创建 .pfx 或 .p12，可以这么做：

    openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
        -certfile ca-cert.pem -out agent5.pfx

  - `in`:  certificate
  - `inkey`: private key
  - `certfile`: all CA certs concatenated in one file like
    `cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`

## 协议支持-Protocol support

Node.js 默认遵循 SSLv2 和 SSLv3 协议，不过这些协议被禁用。因为他们不太可靠，很容易受到威胁，参见 [CVE-2014-3566][]。某些情况下，旧版本客户端/服务器（比如 IE6）可能会产生问题。如果你想启用 SSLv2 或 SSLv3 ，使用参数`--enable-ssl2` 或 `--enable-ssl3` 运行 Node。Node.js 的未来版本中不会再默认编译 SSLv2 和 SSLv3。

有一个办法可以强制 node 进入仅使用  SSLv3 或 SSLv2 模式，分别指定`secureProtocol` 为 `'SSLv3_method'` 或 `'SSLv2_method'`。

Node.js 使用的默认协议方法准确名字是 `AutoNegotiate_method`， 这个方法会尝试并协商客户端支持的从高到底协议。为了提供默认的安全级别，Node.js(v0.10.33 版本之后)通过将 `secureOptions` 设为`SSL_OP_NO_SSLv3|SSL_OP_NO_SSLv2` ，明确的禁用了 SSLv3 和 SSLv2（除非你给`secureProtocol` 传值`--enable-ssl3`, 或 `--enable-ssl2`, 或 `SSLv3_method` ）。

如果你设置了 `secureOptions`，我们不会重新这个参数。


改变这个行为的后果：

 * 如果你的应用被当做为安全服务器，`SSLv3` 客户端不能协商建立连接，会被拒绝。这种情况下，你的服务器会触发 `clientError` 事件。错误消息会包含错误版本数字( `'wrong version number'`).
 * 如果你的应用被当做安全客户端，和一个不支持比 SSLv3 更高安全性的方法的服务器通讯，你的连接不会协商成功。这种情况下，你的客户端会触发 `clientError` 事件。错误消息会包含错误版本数字( `'wrong version number'`).

## Client-initiated renegotiation attack mitigation

<!-- type=misc -->

TLS 协议让客户端协商 TLS 会话的某些方法内容。但是，会话协商需要服务器端响应的资源，这回让它成为阻断服务攻击（denial-of-service attacks）的潜在媒介。

为了降低这种情况的发生，重新协商被限制为每10分钟3次。当超出这个界限时，在 [tls.TLSSocket][] 实例上会触发错误。这个限制可设置：

  - `tls.CLIENT_RENEG_LIMIT`: 重新协商 limit, 默认是 3.

  - `tls.CLIENT_RENEG_WINDOW`: 重新协商窗口的时间，单位秒, 默认是 10 分钟.

除非你明确知道自己在干什么，否则不要改变默认值。


要测试你的服务器的话，使用`openssl s_client -connect address:port` 连接服务器，并敲 `R<CR>`（字母 R 键加回车）几次。


## NPN and SNI

<!-- type=misc -->

NPN (Next Protocol Negotiation 下次协议协商) 和 SNI (Server Name Indication 域名指示) 都是 TLS 握手扩展，运行你：

  * NPN - 同一个TLS服务器使用多种协议 (HTTP, SPDY)
  * SNI - 同一个TLS服务器使用多个主机名（不同的 SSL 证书）。
    certificates.


## 完全正向保密-Perfect Forward Secrecy

<!-- type=misc -->
"[Forward Secrecy]" 或 "Perfect Forward Secrecy-完全正向保密" 协议描述了秘钥协商（比如秘钥交换）方法的特点。实际上这意味着及时你的服务器的秘钥有危险，通讯仅有可能被一类人窃听，他们必须设法获的每次会话都会生成的秘钥对。

完全正向保密是通过每次握手时为秘钥协商随机生成密钥对来完成（和所有会话一个 key 相反）。实现这个技术（提供完全正向保密-Perfect Forward Secrecy）的方法被称为 "ephemeral"。

通常目前有2个方法用于完成完全正向保密（Perfect Forward Secrecy）

Currently two methods are commonly used to achieve Perfect Forward Secrecy (note
the character "E" appended to the traditional abbreviations):

  * [DHE] - 一个迪菲-赫尔曼密钥交换密钥协议（Diffie Hellman key-agreement protocol）短暂（ephemeral）版本。
  * [ECDHE] - 一个椭圆曲线密钥交换密钥协议（ Elliptic Curve Diffie Hellman key-agreement protocol）短暂（ephemeral）版本。

短暂（ephemeral）方法有性能缺点，因为生成 key 非常耗费资源。


## tls.getCiphers()

返回支持的 SSL 密码名数组。

例子：

    var ciphers = tls.getCiphers();
    console.log(ciphers); // ['AES128-SHA', 'AES256-SHA', ...]


## tls.createServer(options[, secureConnectionListener])

创建一个新的 [tls.Server][]。参数 `connectionListener` 会自动设置为 [secureConnection][]  事件的监听器。参数 `options` 对象有以下可能性：

  - `pfx`: 包含私钥，证书和服务器的 CA 证书（PFX 或 PKCS12 格式）字符串或缓存`Buffer`。（`key`, `cert` 和 `ca`互斥）。

  - `key`: 包含服务器私钥（PEM 格式）字符串或缓存`Buffer`。（可以是keys的数组）（必传）。

  - `passphrase`: 私钥或 pfx 的密码字符串  

  - `cert`: 包含服务器证书key（PEM 格式）字符串或缓存`Buffer`。（可以是certs的数组）（必传）。

  - `ca`: 信任的证书（PEM 格式）的字符串/缓存数组。如果忽略这个参数，将会使用"root" CAs ，比如 VeriSign。用来授权连接。

  - `crl` : 不是 PEM 编码 CRLs （证书撤销列表 Certificate Revocation List）的字符串就是字符串列表.

  - `ciphers`: 要使用或排除的密码（cipher）字符串

    为了减轻[BEAST attacks] ，推荐使用这个参数和之后会提到的 `honorCipherOrder` 参数来优化non-CBC 密码（cipher）

    默认：`ECDHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA256:AES128-GCM-SHA256:RC4:HIGH:!MD5:!aNULL`.
    格式上更多细节参见 [OpenSSL cipher list format documentation]

    `ECDHE-RSA-AES128-SHA256`, `DHE-RSA-AES128-SHA256` 和 `AES128-GCM-SHA256` 都是 TLS v1.2 密码（cipher），当 node.js 连接 OpenSSL 1.0.1 或更早版本（比如）时使用。注意， `honorCipherOrder` 设置为 enabled 后，现在任然可以和 TLS v1.2 客户端协商弱密码（cipher），

    `RC4` 可作为客户端和老版本 TLS 协议通讯的备用方法。`RC4` 这些年受到怀疑，任何对信任敏感的对象都会考虑其威胁性。国家级别（state-level）的参与者拥有中断它的能力。

    **注意**: 早些版本的修订建议， `AES256-SHA` 作为可以接受的密码（cipher）.Unfortunately, `AES256-SHA` 是一个 CBC 密码（cipher），容易受到 [BEAST attacks] 攻击。 *不要* 使用它。  

  - `ecdhCurve`: 包含用来 ECDH 秘钥交换弧形（curve）名字符串，或者 false 禁用 ECDH。

    默认 `prime256v1`. 更多细节参考 [RFC 4492] 。

  - `dhparam`: DH 参数文件，用于 DHE 秘钥协商。使用 `openssl dhparam` 命令来创建。如果加载文件失败，会悄悄的抛弃它。

  - `handshakeTimeout`: 如果 SSL/TLS 握手事件超过这个参数，会放弃里连接。 默认是 120 秒.

    握手超时后， `tls.Server` 对象会触发 `'clientError`' 事件。

  - `honorCipherOrder` : 当选择一个密码（cipher） 时, 使用服务器配置，而不是客户端的。  

    虽然这个参数默认不可用，还是推荐你用这个参数，和  `ciphers` 参数连接使用，减轻 BEAST 攻击。

    注意，如果使用了 SSLv2，服务器会发送自己的配置列表给客户端，客户端会挑选密码（cipher）。默认不支持 SSLv2，除非 node.js 配置了`./configure --with-sslv2`。

  - `requestCert`: 如果设为 `true`，服务器会要求连接的客户端发送证书，并尝试验证证书。默认：`false`。

  - `rejectUnauthorized`: If `true` the server will reject any connection
    which is not authorized with the list of supplied CAs. This option only
    has an effect if `requestCert` is `true`. Default: `false`.

  - `checkServerIdentity(servername, cert)`: Provide an override for checking
    server's hostname against the certificate. Should return an error if verification
    fails. Return `undefined` if passing.

  - `NPNProtocols`: An array or `Buffer` of possible NPN protocols. (Protocols
    should be ordered by their priority).

  - `SNICallback(servername, cb)`: A function that will be called if client
    supports SNI TLS extension. Two argument will be passed to it: `servername`,
    and `cb`. `SNICallback` should invoke `cb(null, ctx)`, where `ctx` is a
    SecureContext instance.
    (You can use `tls.createSecureContext(...)` to get proper
    SecureContext). If `SNICallback` wasn't provided - default callback with
    high-level API will be used (see below).

  - `sessionTimeout`: An integer specifying the seconds after which TLS
    session identifiers and TLS session tickets created by the server are
    timed out. See [SSL_CTX_set_timeout] for more details.

  - `ticketKeys`: A 48-byte `Buffer` instance consisting of 16-byte prefix,
    16-byte hmac key, 16-byte AES key. You could use it to accept tls session
    tickets on multiple instances of tls server.

    NOTE: Automatically shared between `cluster` 模块 workers.

  - `sessionIdContext`: A string containing an opaque identifier for session
    resumption. If `requestCert` is `true`, the 默认是 MD5 hash value
    generated from command-line. Otherwise, the 默认是 not provided.

  - `secureProtocol`: The SSL method to use, e.g. `SSLv3_method` to force
    SSL version 3. The possible values depend on your installation of
    OpenSSL and are defined in the constant [SSL_METHODS][].

  - `secureOptions`: Set server options. For example, to disable the SSLv3
    protocol set the `SSL_OP_NO_SSLv3` flag. See [SSL_CTX_set_options]
    for all available options.

Here is a simple example echo server:

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

      // This is necessary only if the client uses the self-signed certificate.
      ca: [ fs.readFileSync('client-cert.pem') ]
    };

    var server = tls.createServer(options, function(socket) {
      console.log('server connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      socket.write("welcome!\n");
      socket.setEncoding('utf8');
      socket.pipe(socket);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });

Or

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('server.pfx'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

    };

    var server = tls.createServer(options, function(socket) {
      console.log('server connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      socket.write("welcome!\n");
      socket.setEncoding('utf8');
      socket.pipe(socket);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });
You can test this server by connecting to it with `openssl s_client`:


    openssl s_client -connect 127.0.0.1:8000


## tls.connect(options[, callback])
## tls.connect(port[, host][, options][, callback])

Creates a new client connection to the given `port` and `host` (old API) or
`options.port` and `options.host`. (If `host` is omitted, it defaults to
`localhost`.) `options` should be an object which specifies:

  - `host`: Host the client should connect to

  - `port`: Port the client should connect to

  - `socket`: Establish secure connection on a given socket rather than
    creating a new socket. If this option is specified, `host` and `port`
    are ignored.

  - `path`: Creates unix socket connection to path. If this option is
    specified, `host` and `port` are ignored.

  - `pfx`: A string or `Buffer` containing the private key, certificate and
    CA certs of the client in PFX or PKCS12 format.

  - `key`: A string or `Buffer` containing the private key of the client in
    PEM format. (Could be an array of keys).

  - `passphrase`: A string of passphrase for the private key or pfx.

  - `cert`: A string or `Buffer` containing the certificate key of the client in
    PEM format. (Could be an array of certs).

  - `ca`: An array of strings or `Buffer`s of trusted certificates in PEM
    format. If this is omitted several well known "root" CAs will be used,
    like VeriSign. These are used to authorize connections.

  - `rejectUnauthorized`: If `true`, the server certificate is verified against
    the list of supplied CAs. An `'error'` event is emitted if verification
    fails; `err.code` contains the OpenSSL error code. Default: `true`.

  - `NPNProtocols`: An array of strings or `Buffer`s containing supported NPN
    protocols. `Buffer`s should have following format: `0x05hello0x05world`,
    where first byte is next protocol name's length. (Passing array should
    usually be much simpler: `['hello', 'world']`.)

  - `servername`: Servername for SNI (Server Name Indication) TLS extension.

  - `secureProtocol`: The SSL method to use, e.g. `SSLv3_method` to force
    SSL version 3. The possible values depend on your installation of
    OpenSSL and are defined in the constant [SSL_METHODS][].

  - `session`: A `Buffer` instance, containing TLS session.

The `callback` parameter will be added as a listener for the
['secureConnect'][] event.

`tls.connect()` returns a [tls.TLSSocket][] object.

Here is an example of a client of echo server as described previously:

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      // These are necessary only if using the client certificate authentication
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-cert.pem'),

      // This is necessary only if the server uses the self-signed certificate
      ca: [ fs.readFileSync('server-cert.pem') ]
    };

    var socket = tls.connect(8000, options, function() {
      console.log('client connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(socket);
      process.stdin.resume();
    });
    socket.setEncoding('utf8');
    socket.on('data', function(data) {
      console.log(data);
    });
    socket.on('end', function() {
      server.close();
    });

Or

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('client.pfx')
    };

    var socket = tls.connect(8000, options, function() {
      console.log('client connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(socket);
      process.stdin.resume();
    });
    socket.setEncoding('utf8');
    socket.on('data', function(data) {
      console.log(data);
    });
    socket.on('end', function() {
      server.close();
    });

## Class: tls.TLSSocket

Wrapper for instance of [net.Socket][], replaces internal socket read/write
routines to perform transparent encryption/decryption of incoming/outgoing data.

## new tls.TLSSocket(socket, options)

Construct a new TLSSocket object from existing TCP socket.

`socket` is an instance of [net.Socket][]

`options` is an object that might contain following properties:

  - `secureContext`: An optional TLS context object from
     `tls.createSecureContext( ... )`

  - `isServer`: If true - TLS socket will be instantiated in server-mode

  - `server`: An optional [net.Server][] instance

  - `requestCert`: Optional, see [tls.createSecurePair][]

  - `rejectUnauthorized`: Optional, see [tls.createSecurePair][]

  - `NPNProtocols`: Optional, see [tls.createServer][]

  - `SNICallback`: Optional, see [tls.createServer][]

  - `session`: Optional, a `Buffer` instance, containing TLS session

  - `requestOCSP`: Optional, if `true` - OCSP status request extension would
    be added to client hello, and `OCSPResponse` event will be emitted on socket
    before establishing secure communication


## tls.createSecureContext(details)

Creates a credentials object, with the optional details being a
dictionary with keys:

* `pfx` : A string or buffer holding the PFX or PKCS12 encoded private
  key, certificate and CA certificates
* `key` : A string holding the PEM encoded private key
* `passphrase` : A string of passphrase for the private key or pfx
* `cert` : A string holding the PEM encoded certificate
* `ca` : Either a string or list of strings of PEM encoded CA
  certificates to trust.
* `crl` : Either a string or list of strings of PEM encoded CRLs
  (Certificate Revocation List)
* `ciphers`: A string describing the 密码（cipher） to use or exclude.
  Consult
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>
  for details on the format.
* `honorCipherOrder` : When choosing a cipher, use the server's preferences
  instead of the client preferences. For further details see `tls` 模块
  documentation.

If no 'ca' details are given, then node.js will use the default
publicly trusted list of CAs as given in
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>.


## tls.createSecurePair([context][, isServer][, requestCert][, rejectUnauthorized])

Creates a new secure pair object with two streams, one of which reads/writes
encrypted data, and one reads/writes cleartext data.
Generally the encrypted one is piped to/from an incoming encrypted data stream,
and the cleartext one is used as a replacement for the initial encrypted stream.

 - `credentials`: A secure context object from tls.createSecureContext( ... )

 - `isServer`: A boolean indicating whether this tls connection should be
   opened as a server or a client.

 - `requestCert`: A boolean indicating whether a server should request a
   certificate from a connecting client. Only applies to server connections.

 - `rejectUnauthorized`: A boolean indicating whether a server should
   automatically reject clients with invalid certificates. Only applies to
   servers with `requestCert` enabled.

`tls.createSecurePair()` returns a SecurePair object with `cleartext` and
`encrypted` stream properties.

NOTE: `cleartext` has the same APIs as [tls.TLSSocket][]

## Class: SecurePair

Returned by tls.createSecurePair.

### Event: 'secure'

The event is emitted from the SecurePair once the pair has successfully
established a secure connection.

Similarly to the checking for the server 'secureConnection' event,
pair.cleartext.authorized should be checked to confirm whether the certificate
used properly authorized.

## Class: tls.Server

This class is a subclass of `net.Server` and has the same methods on it.
Instead of accepting just raw TCP connections, this accepts encrypted
connections using TLS or SSL.

### Event: 'secureConnection'

`function (tlsSocket) {}`

This event is emitted after a new connection has been successfully
handshaked. The argument is an instance of [tls.TLSSocket][]. It has all the
common stream methods and events.

`socket.authorized` is a boolean value which indicates if the
client has verified by one of the supplied certificate authorities for the
server. If `socket.authorized` is false, then
`socket.authorizationError` is set to describe how authorization
failed. Implied but worth mentioning: depending on the settings of the TLS
server, you unauthorized connections may be accepted.
`socket.npnProtocol` is a string containing selected NPN protocol.
`socket.servername` is a string containing servername requested with
SNI.


### Event: 'clientError'

`function (exception, tlsSocket) { }`

When a client connection emits an 'error' event before secure connection is
established - it will be forwarded here.

`tlsSocket` is the [tls.TLSSocket][] that the error originated from.


### Event: 'newSession'

`function (sessionId, sessionData, callback) { }`

Emitted on creation of TLS session. May be used to store sessions in external
storage. `callback` must be invoked eventually, otherwise no data will be
sent or received from secure connection.

NOTE: adding this event listener will have an effect only on connections
established after addition of event listener.


### Event: 'resumeSession'

`function (sessionId, callback) { }`

Emitted when client wants to resume previous TLS session. Event listener may
perform lookup in external storage using given `sessionId`, and invoke
`callback(null, sessionData)` once finished. If session can't be resumed
(i.e. doesn't exist in storage) one may call `callback(null, null)`. Calling
`callback(err)` will terminate incoming connection and destroy socket.

NOTE: adding this event listener will have an effect only on connections
established after addition of event listener.


### Event: 'OCSPRequest'

`function (certificate, issuer, callback) { }`

Emitted when the client sends a certificate status request. You could parse
server's current certificate to obtain OCSP url and certificate id, and after
obtaining OCSP response invoke `callback(null, resp)`, where `resp` is a
`Buffer` instance. Both `certificate` and `issuer` are a `Buffer`
DER-representations of the primary and issuer's certificates. They could be used
to obtain OCSP certificate id and OCSP endpoint url.

Alternatively, `callback(null, null)` could be called, meaning that there is no
OCSP response.

Calling `callback(err)` will result in a `socket.destroy(err)` call.

Typical flow:

1. Client connects to server and sends `OCSPRequest` to it (via status info
   extension in ClientHello.)
2. Server receives request and invokes `OCSPRequest` event listener if present
3. Server grabs OCSP url from either `certificate` or `issuer` and performs an
   [OCSP request] to the CA
4. Server receives `OCSPResponse` from CA and sends it back to client via
   `callback` argument
5. Client validates the response and either destroys socket or performs a
   handshake.

NOTE: `issuer` could be null, if the certificate is self-signed or if the issuer
is not in the root certificates list. (You could provide an issuer via `ca`
option.)

NOTE: adding this event listener will have an effect only on connections
established after addition of event listener.

NOTE: you may want to use some npm 模块 like [asn1.js] to parse the
certificates.


### server.listen(port[, host][, callback])

Begin accepting connections on the specified `port` and `host`.  If the
`host` is omitted, the server will accept connections directed to any
IPv4 address (`INADDR_ANY`).

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound.

See `net.Server` for more information.


### server.close()

Stops the server from accepting new connections. This function is
asynchronous, the server is finally closed when the server emits a `'close'`
event.

### server.address()

Returns the bound address, the address family name and port of the
server as reported by the operating system.  See [net.Server.address()][] for
more information.

### server.addContext(hostname, context)

Add secure context that will be used if client request's SNI hostname is
matching passed `hostname` (wildcards can be used). `context` can contain
`key`, `cert`, `ca` and/or any other properties from `tls.createSecureContext`
`options` argument.

### server.maxConnections

Set this property to reject connections when the server's connection count
gets high.

### server.connections

The number of concurrent connections on the server.


## Class: CryptoStream

    Stability: 0 - Deprecated. Use tls.TLSSocket instead.

This is an encrypted stream.

### cryptoStream.bytesWritten

A proxy to the underlying socket's bytesWritten accessor, this will return
the total bytes written to the socket, *including the TLS overhead*.

## Class: tls.TLSSocket

This is a wrapped version of [net.Socket][] that does transparent encryption
of written data and all required TLS negotiation.

This instance implements a duplex [Stream][] interfaces.  It has all the
common stream methods and events.

### Event: 'secureConnect'

This event is emitted after a new connection has been successfully handshaked.
The listener will be called no matter if the server's certificate was
authorized or not. It is up to the user to test `tlsSocket.authorized`
to see if the server certificate was signed by one of the specified CAs.
If `tlsSocket.authorized === false` then the error can be found in
`tlsSocket.authorizationError`. Also if NPN was used - you can check
`tlsSocket.npnProtocol` for negotiated protocol.

### Event: 'OCSPResponse'

`function (response) { }`

This event will be emitted if `requestOCSP` option was set. `response` is a
buffer object, containing server's OCSP response.

Traditionally, the `response` is a signed object from the server's CA that
contains information about server's certificate revocation status.

### tlsSocket.encrypted

Static boolean value, always `true`. May be used to distinguish TLS sockets
from regular ones.

### tlsSocket.authorized

A boolean that is `true` if the peer certificate was signed by one of the
specified CAs, otherwise `false`

### tlsSocket.authorizationError

The reason why the peer's certificate has not been verified. This property
becomes available only when `tlsSocket.authorized === false`.

### tlsSocket.getPeerCertificate([ detailed ])

Returns an object representing the peer's certificate. The returned object has
some properties corresponding to the field of the certificate. If `detailed`
argument is `true` - the full chain with `issuer` property will be returned,
if `false` - only the top certificate without `issuer` property.

例子：

    { subject:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      issuerInfo:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      issuer:
       { ... another certificate ... },
      raw: < RAW DER buffer >,
      valid_from: 'Nov 11 09:52:22 2009 GMT',
      valid_to: 'Nov  6 09:52:22 2029 GMT',
      fingerprint: '2A:7A:C2:DD:E5:F9:CC:53:72:35:99:7A:02:5A:71:38:52:EC:8A:DF',
      serialNumber: 'B9B0D332A1AA5635' }

If the peer does not provide a certificate, it returns `null` or an empty
object.

### tlsSocket.getCipher()
Returns an object representing the cipher name and the SSL/TLS
protocol version of the current connection.

例子：
{ name: 'AES256-SHA', version: 'TLSv1/SSLv3' }

See SSL_CIPHER_get_name() and SSL_CIPHER_get_version() in
http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_CIPHERS for more
information.

### tlsSocket.renegotiate(options, callback)

Initiate TLS 重新协商 process. The `options` may contain the following
fields: `rejectUnauthorized`, `requestCert` (See [tls.createServer][]
for details). `callback(err)` will be executed with `null` as `err`,
once the 重新协商 is successfully completed.

NOTE: Can be used to request peer's certificate after the secure connection
has been established.

ANOTHER NOTE: When running as the server, socket will be destroyed
with an error after `handshakeTimeout` timeout.

### tlsSocket.setMaxSendFragment(size)

Set maximum TLS fragment size (default and maximum value is: `16384`, minimum
is: `512`). Returns `true` on success, `false` otherwise.

Smaller fragment size decreases buffering latency on the client: large
fragments are buffered by the TLS layer until the entire fragment is received
and its integrity is verified; large fragments can span multiple roundtrips,
and their processing can be delayed due to packet loss or reordering. However,
smaller fragments add extra TLS framing bytes and CPU overhead, which may
decrease overall server throughput.

### tlsSocket.getSession()

Return ASN.1 encoded TLS session or `undefined` if none was negotiated. Could
be used to speed up handshake establishment when reconnecting to the server.

### tlsSocket.getTLSTicket()

NOTE: Works only with client TLS sockets. Useful only for debugging, for
session reuse provide `session` option to `tls.connect`.

Return TLS session ticket or `undefined` if none was negotiated.

### tlsSocket.address()

Returns the bound address, the address family name and port of the
underlying socket as reported by the operating system. Returns an
object with three properties, e.g.
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### tlsSocket.remoteAddress

The string representation of the remote IP address. For example,
`'74.125.127.100'` or `'2001:4860:a005::68'`.

### tlsSocket.remoteFamily

The string representation of the remote IP family. `'IPv4'` or `'IPv6'`.

### tlsSocket.remotePort

The numeric representation of the remote port. For example, `443`.

### tlsSocket.localAddress

The string representation of the local IP address.

### tlsSocket.localPort

The numeric representation of the local port.

[OpenSSL cipher list format documentation]: http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT
[BEAST attacks]: http://blog.ivanristic.com/2011/10/mitigating-the-beast-attack-on-tls.html
[tls.createServer]: #tls_tls_createserver_options_secureconnectionlistener
[tls.createSecurePair]: #tls_tls_createsecurepair_credentials_isserver_requestcert_rejectunauthorized
[tls.TLSSocket]: #tls_class_tls_tlssocket
[net.Server]: net.html#net_class_net_server
[net.Socket]: net.html#net_class_net_socket
[net.Server.address()]: net.html#net_server_address
['secureConnect']: #tls_event_secureconnect
[secureConnection]: #tls_event_secureconnection
[Stream]: stream.html#stream_stream
[SSL_METHODS]: http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_PROTOCOL_METHODS
[tls.Server]: #tls_class_tls_server
[SSL_CTX_set_timeout]: http://www.openssl.org/docs/ssl/SSL_CTX_set_timeout.html
[RFC 4492]: http://www.rfc-editor.org/rfc/rfc4492.txt
[Forward secrecy]: http://en.wikipedia.org/wiki/Perfect_forward_secrecy
[DHE]: https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange
[ECDHE]: https://en.wikipedia.org/wiki/Elliptic_curve_Diffie%E2%80%93Hellman
[asn1.js]: http://npmjs.org/package/asn1.js
[OCSP request]: http://en.wikipedia.org/wiki/OCSP_stapling
[SSL_CTX_set_options]: https://www.openssl.org/docs/ssl/SSL_CTX_set_options.html
[CVE-2014-3566]: https://access.redhat.com/articles/1232123
