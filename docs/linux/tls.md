# HTTPS, SSL, TLS

传输层安全性协议(英语: Transport Layer Security, 缩写: TLS)及其前身安全套接层（英语：Secure Socket Layer，缩写：SSL）是一种安全协议，目的是为互联网通信提供安全及数据完整性保障。

网景公司（Netscape）在 1994 年推出 HTTPS 协议，以 SSL 进行加密，这是 SSL 的起源。

IETF 将 SSL 进行标准化，1999年公布TLS 1.0 标准文件（RFC 2246）。随后又公布 TLS1.1（RFC 4366,2006年）、TLS 1.2（RFC 5246，2008年）和TLS 1.3（RFC 8466,2018年）

> 目前推荐使用的是 TLS1.2 和 TLS1.3

## 协议过程

SSL/TLS协议的基本思路是采用公钥加密法，也就是说，客户端先向服务器端索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

但是，这里有两个问题。

（1）如何保证公钥不被篡改？

> 解决方法：将公钥放在数字证书中。只要证书是可信的，公钥就是可信的。

（2）公钥加密计算量太大，如何减少耗用的时间？

> 解决方法：每一次对话（session），客户端和服务器端都生成一个"对话密钥"（session key），用它来加密信息。由于"对话密钥"是对称加密，所以运算速度非常快，而服务器公钥只用于加密"对话密钥"本身，这样就减少了加密运算的消耗时间。

因此，SSL/TLS协议的基本过程是这样的：

（1） 客户端向服务器端索要并验证公钥。

（2） 双方协商生成"对话密钥"。

（3） 双方采用"对话密钥"进行加密通信。

上面过程的前两步，又称为"握手阶段"（handshake）。

## 握手过程

假定客户端叫做爱丽丝，服务器叫做鲍勃，整个握手过程可以用下图说明

> 爱丽丝（Alice）和鲍勃（Bob）是广泛地代入密码学和物理学领域的通用角色。

![](../images/tls-handshake.png)

```
第一步，爱丽丝给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法。

第二步，鲍勃确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random）。

第三步，爱丽丝确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给鲍勃。

第四步，鲍勃使用自己的私钥，获取爱丽丝发来的随机数（即Premaster secret）。

第五步，爱丽丝和鲍勃根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"（session key），用来加密接下来的整个对话过程。
```

```
（1）生成对话密钥一共需要三个随机数。其中两个随机数是明文，最后一个是密文，三个一起随机性更强

（2）握手之后的对话使用"对话密钥"加密（对称加密），服务器的公钥和私钥只用于加密和解密"对话密钥"（非对称加密），无其他作用。

（3）服务器公钥放在服务器的数字证书之中。
``

## 生成证书

### openssl

```bash
# 生成 CA 私钥，无加密
openssl genrsa -out ca.key 2048
# 生成 CA 私钥使用 AES256 加密输出，后续的步骤需要输入密码
openssl genrsa -aes256 -passout pass:123456 -out ca.key 2048
# 生成 CSR (Certificate Signing Request) 证书签名请求文件，过程会要求输入证书相应信息
openssl req -new -key ca.key  -out ca.csr
# x509 命令生成 CA 根证书
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt

# 生成服务器私钥
openssl genrsa -out server.key 2048
# 生成服务器公钥，如果使用数字证书，这步不需要
openssl rsa -in server.key -pubout -out server.pem
# 服务器端需要向 CA 机构申请签名证书，在申请签名证书之前依然是创建自己的 CSR 文件
openssl req -new -key server.key -out server.csr
# 向自己的 CA 机构申请证书，签名过程需要 CA 的证书和私钥参与，最终颁发一个带有 CA 签名的证书
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt

# 查看证书内容
openssl x509 -text -in server.crt

# 如果需要验证客户端，则需要生成客户端证书
# 生成客户端私钥
openssl genrsa -out client.key 2048
# 生成客户端公钥
openssl rsa -in client.key -pubout -out client.pem
# client 端 CSR
openssl req -new -key client.key -out client.csr
# client 端 CA 签名
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in client.csr -out client.crt

# 把证书和密钥合并到一个 pem 文件，方便成对引用，管理
cat server.key server.crt > server.pem
```

证书默认有效期为 365 天，可另外指定

```
$ openssl help
Standard commands
asn1parse         ca                ciphers           cms
crl               crl2pkcs7         dgst              dhparam
dsa               dsaparam          ec                ecparam
enc               engine            errstr            gendsa
genpkey           genrsa            help              list
nseq              ocsp              passwd            pkcs12
pkcs7             pkcs8             pkey              pkeyparam
pkeyutl           prime             rand              rehash
req               rsa               rsautl            s_client
s_server          s_time            sess_id           smime
speed             spkac             srp               storeutl
ts                verify            version           x509

Message Digest commands (see the `dgst' command for more details)
blake2b512        blake2s256        gost              md4
md5               mdc2              rmd160            sha1
sha224            sha256            sha3-224          sha3-256
sha3-384          sha3-512          sha384            sha512
sha512-224        sha512-256        shake128          shake256
sm3

Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb
aes-256-cbc       aes-256-ecb       aria-128-cbc      aria-128-cfb
aria-128-cfb1     aria-128-cfb8     aria-128-ctr      aria-128-ecb
aria-128-ofb      aria-192-cbc      aria-192-cfb      aria-192-cfb1
aria-192-cfb8     aria-192-ctr      aria-192-ecb      aria-192-ofb
aria-256-cbc      aria-256-cfb      aria-256-cfb1     aria-256-cfb8
aria-256-ctr      aria-256-ecb      aria-256-ofb      base64
bf                bf-cbc            bf-cfb            bf-ecb
bf-ofb            camellia-128-cbc  camellia-128-ecb  camellia-192-cbc
camellia-192-ecb  camellia-256-cbc  camellia-256-ecb  cast
cast-cbc          cast5-cbc         cast5-cfb         cast5-ecb
cast5-ofb         des               des-cbc           des-cfb
des-ecb           des-ede           des-ede-cbc       des-ede-cfb
des-ede-ofb       des-ede3          des-ede3-cbc      des-ede3-cfb
des-ede3-ofb      des-ofb           des3              desx
idea              idea-cbc          idea-cfb          idea-ecb
idea-ofb          rc2               rc2-40-cbc        rc2-64-cbc
rc2-cbc           rc2-cfb           rc2-ecb           rc2-ofb
rc4               rc4-40            rc5               rc5-cbc
rc5-cfb           rc5-ecb           rc5-ofb           seed
seed-cbc          seed-cfb          seed-ecb          seed-ofb
sm4-cbc           sm4-cfb           sm4-ctr           sm4-ecb
sm4-ofb           zlib
```

## keytool

`openssl` 默认使用 `PEM` 格式（形如`-----BEGIN CERTIFICATE----- ... ...-----END CERTIFICATE-----`）存放证书和私钥，`nginx/node.js` 可以使用该格式启动服务。

但是对于 Java 体系，默认使用的是自己的密钥库（`KeyStore`），常见的有 `JKS`, `JCEKS`, `PKCS12` 三个类型

> 在使用 tomcat 或 Java 客户端或 android 时，一般都是使用 KeyStore   
> KeyStore 的特点是可以存储多个服务器密码，有密码保护，二进制

其中 JKS 和 JCEKS 属于 Java 体系独有规范，PKCS12 属于公共标准

openssl 能够生成 PKCS12 格式的密钥库

PKCS12 格式文件，可以包含多个证书/私钥对，指定多个受信任的 server（也可以不包含证书），每个 server 有一个 aliase name。我们来看最简单的只包含一个 alias 的文件生成：

```bash
# 导出时需要指定一个密码
openssl pkcs12 -export -in ca.crt -inkey ca.key -out ca.p12
```

另外，jdk 中包含一个 keytool 命令，可以完成整个的证书生成过程

- 生成自签名证书、服务端认证、客户端信任

```bash
# 生成密钥对，默认是 JKS 格式，注意要求输入名字时要输入域名，里面包含了自签名证书
keytool -genkey -alias server -keyalg RSA -keystore server.keystore

# 查看 keystore 文件
keytool -list -v -keystore server.keystore

# 导出证书，cer 格式是微软对 crt 格式的一个复制，赋予了某些特别操作，此处的证书是 der 二进制编码
keytool -exportcert -keystore server.keystore -alias server -file server.cer
# 添加 -rfc 选项导出 PEM ASCII 编码证书
keytool -exportcert -keystore server.keystore -alias server -rfc -file server.cer

# 查看证书
keytool -printcert -file server.cer

# 客户端导入信任证书（SSL 客户端使用）
keytool -importcert -keystore client_trust.keystore -file server.cer -alias client_trust_server
```

tomcat server.xml 配置 ssl

```xml
<!-- Define a SSL Coyote HTTP/1.1 Connector on port 8443 -->
<Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="${user.home}/.keystore" keystorePass="changeit"
           clientAuth="false" sslProtocol="TLS"/>
```

java 客户端请求前配置：

```java
System.setProperty("java.protocol.handler.pkgs", "com.sun.net.ssl.internal.www.protocol");
System.setProperty("java.protocol.handler.pkgs", "com.ibm.net.ssl.internal.www.protocol");

String trustStorePath = "~/client_trust.keystore";

String trustStorePassword = "123456";

System.setProperty("javax.net.ssl.trustStore", trustStorePath);
System.setProperty("javax.net.ssl.trustStorePassword", trustStorePassword);
```

tomcat 客户端请求前配置：

```xml
<Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="9000" maxThreads="8000"
           truststoreFile="${catalina.home}/conf/client_trust.keystore"
           URIEncoding="UTF-8"
           />
``

- CA 签名证书

```bash
# 通过 keystore 生成 CSR 文件，用于给提供给权威机构生成正式证书
keytool -certreq -keyalg RSA -alias server -file server.csr -keystore server.keystore
# 通过 openssl 生成 crt 证书
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt
# 导入 keystore
keytool -importcert -keystore server.keystore -file server.crt -alias server 
```

- JDK 导入自定义根证书

把 ca.crt 拷贝导 $JAVA_HOME/jre/lib/security 下，运行以下命令：

```bash
keytool -importcert -alias venustest -keystore cacerts -file venus.cer -storepass changeit
```

一般密码默认为 changeit，并选择信任该证书

> `$JAVA_HOME/jre/lib/security/cacerts` 是 jdk 的信任密钥库

```
$ keytool
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法
```

## 证书格式

### 证书格式

常见有两种：`crt` 和 `cer`

- crt 是 Unix 和类 Unix 的证书格式
- cer 是微软的，常见于 windows 系统

> crt 和 cer 内容是一样，唯一的区别只是 windows 会对不同后缀的证书的点击操作做出不同的响应

### 证书编码格式

常见也有两种 `pem` 和 `der`
- pem 是 ASCII Base64 编码
- der 是二进制编码

> cer 一般都是 der 编码

```bash
# 查看 pem 编码证书
openssl x509 -text -in server.crt
# 查看 der 编码证书
openssl x509 -inform der -text -in server.crt

# pem 转 der
openssl x509 -in server.crt -outform der -out server.der
# der 转 pem
openssl x509 -in server.crt -inform der -outform pem -out server.pem
```

> 证书文件后缀既可以使用编码后缀也可以使用格式后缀

### 证书存储格式

常见的也有两种 `pem` 和 `keystore`

> 存储多个证书和密钥，并加以密码保存

- pem 不仅仅是中编码格式，也是种最广泛的存储格式
- keystore 主要见于 Java 体系，其中又包含的常见格式有 JKS 和 pkcs12

> 推荐使用行业标准的 pkcs12 标准库

```bash
# PEM 转 pkcs12 格式 keystore，注意定义 -name 选项，这将作为 keystore 识别实体的参数
openssl pkcs12 -export -in server.crt -inkey server.key -passin pass:111111 -password pass:111111 -name server -out server.p12

# pkcs12 格式 keystore 转 jks 格式 keystore
keytool -importkeystore -scrkeystore server.p12 -destkeystore server.keystore \
-srcstoretype pkcs12 -deststoretype jks -srcalias server -destalias server \
-deststorepass 111111 -srcstorepass 111111

# pkcs12 转 pem
openssl pkcs12 -in server.p12 -out server.pem
# pkcs12 提取 pem 格式证书
openssl pkcs12 -in server.p12 -clcerts -nokeys -password pass:111111 -out server.crt
# pkcs12 提取 pem 格式密钥
openssl pkcs12 -in server.p12 -nocerts -password pass:111111 -passout pass:111111 -out server.key
```

> 除了涉及 JKS 格式的要用 jdk 自带的 keytool 工具，其它都可以使用 openssl 完成

## 配置证书

- [证书配置文件生成器](https://ssl-config.mozilla.org/)
- [配置文件模板](https://github.com/SSLMate/tlsconfigguide)

## 进一步的安全措施

以下措施可以进一步保证通信安全

### HTTP Strict Transport Security (HSTS)

- 访问网站时，用户很少直接在地址栏输入https://，总是通过点击链接，或者3xx重定向，从HTTP页面进入HTTPS页面。攻击者完全可以在用户发出HTTP请求时，劫持并篡改该请求。

- 另一种情况是恶意网站使用自签名证书，冒充另一个网站，这时浏览器会给出警告，但是许多用户会忽略警告继续访问。

"HTTP严格传输安全"（简称 HSTS）的作用，就是**强制浏览器只能发出HTTPS请求，并阻止用户接受不安全的证书**。

它在网站的响应头里面，加入一个强制性声明。比如:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

上面这段头信息有两个作用。

```
（1）在接下来的一年（即31536000秒）中，浏览器只要向example.com或其子域名发送HTTP请求时，必须采用HTTPS来发起连接。用户点击超链接或在地址栏输入http://www.example.com/，浏览器应当自动将http转写成https，然后直接向https://www.example.com/发送请求。

（2）在接下来的一年中，如果example.com服务器发送的证书无效，用户不能忽略浏览器警告，将无法继续访问该网站
```

HSTS 很大程度上解决了 SSL 剥离攻击。只要浏览器曾经与服务器建立过一次安全连接，之后浏览器会强制使用HTTPS，即使链接被换成了HTTP。

该方法的主要不足是，用户首次访问网站发出HTTP请求时，是不受HSTS保护的。

如果想要全面分析网站的安全程度，可以使用 Mozilla 的 [Observatory](https://developer.mozilla.org/en-US/observatory)

### Cookie

另一个需要注意的地方是，确保浏览器只在使用 HTTPS 时，才发送Cookie。

网站响应头里面，`Set-Cookie`字段加上`Secure`标志即可。

```
Set-Cookie: LSID=DQAAAK...Eaem_vYg; Secure
```

## Refrences

- [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
- [图解SSL/TLS协议](https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)
- [HTTPS性能优化技巧](https://blog.httpwatch.com/2009/01/15/https-performance-tuning/)
- [HTTPS证书生成原理和部署细节](https://www.barretlee.com/blog/2015/10/05/how-to-build-a-https-server/)
- [openssl生成SSL证书的流程](https://www.cnblogs.com/kabi/p/7472976.html)
- [不同格式证书导入keystore方法](https://blog.csdn.net/peterwanghao/article/details/1761728)
- [DER、CRT、CER、PEM格式的证书及转换](https://blog.csdn.net/xiangguiwang/article/details/76400805)
- [使用keytool 生成证书](https://www.cnblogs.com/littleatp/p/5922362.html)
- [tomcat ssl/tls 配置](https://doc.yonyoucloud.com/doc/wiki/project/tomcat/ssl-tls.html)