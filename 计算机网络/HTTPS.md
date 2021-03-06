# HTTPS

说完了HTTP，自然要聊聊它的进阶版HTTPS了。

HTTPS，全名“超文本传输安全协议”，即Hypertext Transfer Protocol Secure。
HTTP协议传输的数据都是未加密的，也就是明文，这传点个人隐私啥的肯定很不安全啊，拦截篡改都很容易。那么为了解决这个问题，就出现了HTTPS。HTTPS经由超文本传输协议（HTTP）进行通信，但利用SSL/TLS来加密数据包。

## 工作流程

![](http://www.runoob.com/wp-content/uploads/2017/05/201208201734403507.png)

1.客户端发起HTTPS请求

就是用户在浏览器里输入一个https网址，然后连接到server的443端口。

2.服务端的配置

采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面。这套证书其实就是一对公钥和私钥。

3.传送证书

证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等等。

4.客户端解析证书

部分工作是有客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值。然后用证书对该随机值进行加密。

5.传送加密信息

传送的是用证书加密后的随机值，目的就是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。

6.服务端加密信息

服务端用私钥解密后，得到了客户端传过来的随机值(私钥)，然后把内容通过该值进行对称加密。所谓对称加密就是，将信息和私钥通过某种算法混合在一起，这样除非知道私钥，不然无法获取内容，而正好客户端和服务端都知道这个私钥

7.传输加密后的信息

8.客户端解密信息

客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容。

HTTPS一般使用的加密与HASH算法如下：

+ 非对称加密算法：RSA，DSA/DSS
+ 对称加密算法：AES，RC4，3DES
+ HASH算法：MD5，SHA1，SHA256