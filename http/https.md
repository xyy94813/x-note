# HTTPS

由于 HTTP 的数据传输是公开透明的，所以如何保证 HTTP 安全可靠的传输就是 **超文本传输安全协议(HyperText Transfer Protocol Secure)** 存在的意义。

通常意义上的 HTTPS 指的是 HTTP + SSL/TLS，是采用 SSL/TLS 对 HTTP 数据包进行加密的方案。
除此之外还有 S-HTTP。
只不过 S-HTTP 诞生之时 HTTPS 已经普遍推广，因而没有被广泛支持。

> SSL (Secure Socket Layer) 是一种安全协议。
> IETF 将 SSL 进行标准化，1999 年公布第一版 TLS（Transport Layer Security）标准文件。

## 对称加密

使用同一个私钥进行加密和解密就是对称加密。

常见的对称加密算法有：

- DES（Data Encryption Standard）
- AES（Advanced Encryption Standard）

对称加密的效率要高于非对称加密。

但是私钥的传输不安全容易破解，且管理困难。（想象你要存储每一个互联网服务器的私钥。）

## 非对称加密

非对称加密，加解密时分别使用两个不同的密钥，公钥和私钥。
使用公钥进行加密，只能使用私钥进行解密；使用私钥进行加密，只能使用公钥进行解密。

常见的非对称加密算法：

- RSA
- Elgamal

非对称加密解决了对称加密的密钥管理和传输问题。

但是相对于对称加密的效率要低的多。

## 请求 HTTPS URL 的过程

1. 用户连接到 Web 站点，该 Web 站点受服务器证书所保护。
2. 服务器进行响应，并自动传送网站的数字证书给用户，用于鉴别站点安全性。
3. 浏览器检查证书是否可信赖。
4. 浏览器客户端使用公钥加密了一个随机对称密钥 K，用以跟网站之间所有的通讯过程进行加密。
5. 服务器用自己的私匙解密出私钥 K，并用私钥解密 HTTP Request 数据包。
6. 服务器使用私钥 K 对 Response 进行加密。
7. 浏览器使用私钥 K 解密 Response。

所以 HTTPS 同时使用了对称加密和非对称加密。

## 数字证书

浏览器如何鉴别证书是否可信赖？
如果代理服务器串改了证书内容？

数字证书一般由证书签证机关（CA）签发，全球只有少数几家 CA 机构。
站点需要向 CA 提交相应材料后进行注册，因此站点的验证信息都存贮在 CA 机构的服务器中。
浏览器通过向机构进行认证而确认证书是否可靠。

数字证书有三种：

- [域名验证（DV）证书](https://www.trustauth.cn/baike/23080.html)
- [组织验证（OV）证书](https://www.trustauth.cn/baike/23173.html)
- [扩展验证（EV）证书](https://www.trustauth.cn/baike/23189.html)
