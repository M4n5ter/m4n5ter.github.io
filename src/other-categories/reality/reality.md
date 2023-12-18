# REALITY 的实现原理

[REALITY](https://github.com/XTLS/REALITY) 是一个基于 [GO](https://github.com/golang/go/commits/master/src/crypto/tls) 最新版本 TLS 库的协议，它能够在不需要域名的情况下，使用他人网站的域名与自己的客户端进行 TLS 握手，并在中间人对 REALITY 服务端进行探测时，向中间人展示他人网站真正的证书，即对中间人表现为流量转发，中间人得到的是他人真正的证书，而不是伪造的证书。

## TLS 1.3 是如何握手的

想知道 REALITY 是如何工作的，首先需要了解 TLS 1.3 是如何握手的。下面是 RFC 8446 中关于 TLS 1.3 握手的流程图：

```text
RFC 8446                           TLS                       August 2018

   Figure 1 below shows the basic full TLS handshake:

       Client                                           Server

Key  ^ ClientHello
Exch | + key_share*
     | + signature_algorithms*
     | + psk_key_exchange_modes*
     v + pre_shared_key*       -------->
                                                  ServerHello  ^ Key
                                                 + key_share*  | Exch
                                            + pre_shared_key*  v
                                        {EncryptedExtensions}  ^  Server
                                        {CertificateRequest*}  v  Params
                                               {Certificate*}  ^
                                         {CertificateVerify*}  | Auth
                                                   {Finished}  v
                               <--------  [Application Data*]
     ^ {Certificate*}
Auth | {CertificateVerify*}
     v {Finished}              -------->
       [Application Data]      <------->  [Application Data]

              +  Indicates noteworthy extensions sent in the
                 previously noted message.

              *  Indicates optional or situation-dependent
                 messages/extensions that are not always sent.

              {} Indicates messages protected using keys
                 derived from a [sender]_handshake_traffic_secret.

              [] Indicates messages protected using keys
                 derived from [sender]_application_traffic_secret_N.

               Figure 1: Message Flow for Full TLS Handshake
```

可以发现 TLS 1.3 握手的过程是 1 RTT（即一个来回），相比 TLS 1.2 减少了一个 RTT。而握手过程分 3 个重要阶段：

1. 密钥交换（Key Exchange）
2. 服务端参数（Server Parameters）
3. 认证（Authentication）

### 密钥交换

建立共享的密钥材料并选择加密参数。此阶段之后的所有内容都将被加密

因此，如果存在中间人，中间人能得到的信息非常有限，最主要的就是 SNI。

### 服务端参数

建立其他握手参数（例如，客户端是否经过认证，应用层协议支持等）。

### 认证

认证服务器（以及可选的客户端认证），并提供密钥确认和握手完整性。

而 TLS 1.3 的握手过程中，密钥交换阶段是最重要的。在密钥交换阶段，客户端发送 ClientHello 消息，该消息包含一个随机数（ClientHello.random）；它提供的协议版本；一系列对称密码/HKDF哈希对；**"key_share"** 扩展中的一组 Diffie-Hellman 密钥共享，或者 "pre_shared_key" 扩展中的一组预共享密钥标签，或者两者都有；可能还有其他扩展。为了与中间件兼容，可能还存在其他字段和/或消息。

服务器处理 ClientHello 并确定连接的适当加密参数。然后，它用自己的 ServerHello 进行响应，该响应指示协商的连接参数。 **ClientHello 和 ServerHello 的组合决定了共享密钥。如果正在使用 (EC)DHE 密钥建立，那么 ServerHello 将包含一个 "key_share" 扩展，其中包含服务器的临时 Diffie-Hellman 共享**；服务器的共享必须与客户端的某个共享在同一组。如果正在使用 PSK 密钥建立，那么 ServerHello 将包含一个 "pre_shared_key" 扩展，指示选择了客户端提供的哪个 PSK。请注意，实现可以同时使用 (EC)DHE 和 PSK，在这种情况下，两个扩展都将被提供。

## REALITY 是如何工作的

### CLIENT

REALITY 客户端需要预先在配置中存储 REALITY 服务端的公钥，并指定与服务端约定好伪装的域名，以及自己的 **shortId**，该 shortId 应该在服务端的配置中存在。

与正常客户端的区别是，REALITY 客户端会在 ClientHello 中携带 sessionId，虽然 sessionId 在 TLS 1.3 中已经不再需要，并可以为空，但为了向后兼容，sessionId 是可以存在的，REALITY 客户端会将 shortId 藏在 sessionId 中，并通过预先在配置文件中设定的 REALITY 服务器的 PUBLIC KEY 与 REALITY 客户端自己的私钥进行 DHKE 得到共享密钥，并使用 AEAD 加密算法对 sessionId 进行加密。

### Server

在服务端，REALITY 与正常 GO TLS 库的区别在于，REALITY 会检查 ClientHello，并且在满足指定条件时才能与 REALITY 服务端本身握手，否则流量将会被导入指定的他人的域名。

1. 它会查看客户端指定的 **SNI** 是否在自己预先配置的列表中，如果在列表中，那么客户端才能够继续与 REALITY 服务端本身进行握手。
2. 它会使用与客户端加密 sessionId 同样的方式来解密 ClientHello 中携带的 sessionId，如果解密失败，那么说明客户端不是 REALITY 客户端，REALITY 服务端本身将会将流量导入指定的他人的域名。
3. 它会检查客户端发送的 **shortId** 是否在服务端的配置文件中给出，只有客户端的 **shortId** 在服务端配置中被包含，客户端才能够继续与 REALITY 服务端本身进行握手。

重要的时，通过 DH 密钥交换算法，REALITY 服务端能够精准识别当前进行握手的是否是受信任的 REALITY 客户端，而不是中间人，因为中间人无法得到 REALITY 客户端的私钥（客户端的密钥对是临时生成的），也无法得到 REALITY 服务端的私钥，因此中间人无法得到 DH 密钥交换算法的共享密钥，也就无法解密 sessionId，这里的安全性是由 TLS 1.3 本身的设计保证的，而 REALITY 服务端只是利用了这个特性，并与客户端提前约定好相关配置（域名，服务端公钥，shortId 等）来甄别受信任的客户端与其他人。

#### Server 的证书

REALITY 服务端本身的证书是可以自签的（在实现中是通过生成一份临时自签证书，并将域名指定为与客户端约定好的他人的域名），因为 REALITY 服务端本身的证书并不会被中间人使用，中间人只会得到他人网站的证书，而不是 REALITY 服务端本身的证书。