# HTTPS

>1,客户端发送一个 ClientHello 消息到服务器端，消息中同时包含了它的 Transport Layer Security (TLS) 版本，可用的加密算法和压缩算法。
>2,服务器端向客户端返回一个 ServerHello 消息，消息中包含了服务器端的 TLS 版本，服务器所选择的加密和压缩算法，以及数字证书认证机构（Certificate Authority，缩写 CA）签发的服务器公开证书，证书中包含了公钥。客户端会使用这个公钥加密接下来的握手过程，直到协商生成一个新的对称密钥。证书中还包含了该证书所应用的域名范围（Common Name，简称 CN），用于客户端验证身份。
>3,客户端根据自己的信任 CA 列表，验证服务器端的证书是否可信。如果认为可信（具体的验证过程在下一节讲解），客户端会生成一串伪随机数，使用服务器的公钥加密它。这串随机数会被用于生成新的对称密钥
>4,服务器端使用自己的私钥解密上面提到的随机数，然后使用这串随机数生成自己的对称主密钥
>5,客户端发送一个 Finished 消息给服务器端，使用对称密钥加密这次通讯的一个散列值
>6,服务器端生成自己的 hash 值，然后解密客户端发送来的信息，检查这两个值是否对应。如果对应，>7,就向客户端发送一个 Finished 消息，也使用协商好的对称密钥加密


C---client hello (client TLS version,可用的加密算法，压缩算法)--->S
C<---server hello (server TLS version,CA证书(公钥)，服务器选择的加密算法和压缩算法)---S
C---     校验证书可信，公钥加密随机数，随机数用于生成对称秘钥    --->S
C---                 finish（使用了对称秘钥）                 --->S
C<---                finish（使用了对称秘钥）                  ---S  //协商结束


>1,从现在开始，接下来整个 TLS 会话都使用对称秘钥进行加密，传输应用层（HTTP）内容
从上面的过程可以看到，TLS 的完整过程需要三个算法（协议），密钥交互算法，对称加密算法，和消息认证算法（TLS 的传输会使用 MAC(message authentication code) 进行完整性检查）。
>2,我们以 Github 网站使用的 TLS 为例，使用浏览器可以看到它使用的加密为 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256。其中密钥交互算法是 ECDHE_RSA，对称加密算法是 AES_128_GCM，消息认证（MAC）算法为 SHA256。

* [https及相关简介](https://hit-alibaba.github.io/interview/basic/network/HTTPS.html)