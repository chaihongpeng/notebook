CSR
证书签名请求文件
C证书 S校验 R请求request

CRT证书

key私钥


获取key-->生成CSR-->获取到CRT

https://slproweb.com/products/Win32OpenSSL.html


生成私钥
openssl genrsa -des3 -out <xxx.key私钥的生成位置> <默认2048位，指定生成的位数>
生成的是加密算法，输入的密码是可以反推回去

生成公钥
openssl req -new -key <之前生成的key的位置> -out <yyy.csr输出位置>
生成待签名证书

openssl req -text -in <之前生成csr的位置>



=====================================================================================================
用户发送请求后-->路由器-->ISP供应商上--->电信/联通-->城市出口-->ISP供应商
$性能上SPDY对网络传输性能做了优化
网络传输过程中，中间的节点是可以随意拦截你传输的数据，并返回伪造信息
如果服务商被黑客入侵，你的请求就会在传输过程中泄密，并被伪造返回值

为了不让拦截者看懂你的消息，就需要将传输数据加密

最重要的环节是客户端把盐发给服务器

文件传输的过程使用的是对称加密算法
des des3  AES   加密过程中需要盐
加密解密使用的盐一样，https使用的就是对称加密算法

秘钥的传递是要使用非对称加密，秘钥有两个
公钥，私钥
公钥进行加密，只有私钥才可以解开得到明文
私钥对明文加密，同样也可以使用公钥进行解密

先有私钥，根据私钥进行加密得到公钥，私钥只是一段特别长的字符串

私钥只有一个，存在服务器端



服务器的私钥生成服务器的公钥，此时其他人都没有数据
服务器要把公钥发给服务器
但是数据传输可以被窃取，黑客窃取了公钥，并把公钥返回给客户端
浏览器通过已经泄露的加密出数据发给服务器
黑客继续获取到公钥加密的数据，但是看不到数据里的内容，只能把请求转给服务器端
	如果黑客通过公钥伪造一份假请求
	服务器返回数据给拦截者，公钥可以解开，但是无论黑客怎么做，都无法伪造客户端通过公钥可以解开的有效数据
	
	但是黑客可以伪造一对公私钥，截取真正的公钥把假公钥发给客户端，理论上黑客作为中间商就这样破解了用户的信息
	
	结论：只靠非对称加密也是不行的，因为浏览器无法判断公钥是不是真的由服务器下发的，还是被拦截的替换

第三套算法，hash摘要算法
	数字签名，有效数据通过摘要算法生成 签名
	摘要就是签名
	(hash私钥，数据)-->签名
	
	算法是固定的，每次发送传递的是数据和签名


​	




CA机构插入加密


服务器拿着Server公钥去CA用CA私钥加密，生成证书
黑客拿到证书后，解密证书拿到Server公钥
黑客更改公钥，用自己的私钥加密出证书

客户端自带公钥，CA的公钥是公开的，CA将公钥内置到操作系统里，CA的公钥是提前内置的，CA的证书里面有域名，域名不对无法访问，申请CA有很复杂的过程，不会出现黑客去其他CA机构生成同域名伪造证书

CA机构提供一个公钥，服务器给CA的公钥生成证书，CA将证书给代理，代理给客户端
到443端口去建立握手
client --> server hi
server --> client hi
客户端向服务端协商加密算法
服务器下发对称加密的秘钥



证书由CA的私钥生成的，他需要公钥去解密
crt pem都是公钥
key私钥
csr请求证书的证书


公钥加密，私钥 签名数据 生成证书
证书就是公钥


自建CA
注:<>内代表可选路径，<>内同名路径请保持全文一致

生成CA私钥
openssl genrsa -out <myca.key>

生成CA公钥，也就是签名
openssl req -new -key <myca.key> -out <myca.csr>

certmgr.msc查看根证书

生成CA根证书
openssl x509 -req -in <myca.csr> -extensions v3_ca -signkey <myca.key> -out <myca.crt>

对服务器端证书签名
openssl x509 -days 365 -req -in <server.csr> -extensions v3_req -CAkey <myca.key> -CA <myca.crt> -CAcreateserial -out server.crt


key私钥 crt公钥

//生产CA私钥
openssl genrsa -passout pass:111111 -des3 -out ca.key 4096
//生成CA公钥
openssl req -passin pass:111111 -new -x509 -days 365 -key ca.key -out ca.crt -subj "/CN=localhost"
//生成服务器私钥
openssl genrsa -passout pass:111111 -des3 -out server.key 4096
//生成服务器请求证书的证书
openssl req -passin pass:111111 -new -key server.key -out server.csr -subj "/CN=localhost"
//生成公钥
openssl x509 -req -passin pass:111111 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

openssl rsa -passin pass:111111 -in server.key -out server.key
openssl genrsa -passout pass:111111 -des3 -out client.key 4096
openssl req -passin pass:111111 -new -key client.key -out client.csr -subj "/CN=localhost"
openssl x509 -passin pass:111111 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
openssl rsa -passin pass:111111 -in client.key -out client.key
openssl pkcs8 -topk8 -nocrypt -in client.key -out client.pem
openssl pkcs8 -topk8 -nocrypt -in server.key -out server.pem


req:主要用于生成和处理PKCS#10证书请求
	-out:指定输出文件名。


根CA通常不会直接为服务端或客户端证书签名, 根CA仅用于创建一个或多个中间CA

# 文件后缀

- `key` 一般指pem类型的私钥
- `csr` 证书请求文件
- `crt` 证书文件, 可以是pem类型
- `pem` pem格式的证书文件

# 创建CA私钥及CA证书

```sh
openssl -x509 -new -newkey rsa:2048 -nodes -keyout ./ca.key -out ./ca.crt -config ca.conf -extensions v3_req -days 3650
```

- `-newkey` 创建新的请求证书, 并创建私钥
- `-x509` 创建自签名证书文件, 而非证书请求文件
- `-key` 指定私钥输入文件
- `-keyout` 指定自动创建私钥时私钥的存放位置
- `-out` 证书请求或自签名证书的输出文件
- `-nodes` 默认自动创建私钥要求加密并提示输入加密密码, 指定该选项禁止对私钥文件加密
- `-days` 自签名证书的有效期限, 需要和`-x509`一起使用

# openssl-command

## rsa

### 用途

用于处理rsa密钥, 格式转换和打印信息

### 选项说明

- `-inform` PEM|NET|DER, 输入文件的格式
- `-outform` PEM|NET|DER, 输出文件格式
- `-in` 输入的RSA密钥文件, 默认为标准输入
- `-out` RSA密钥输出文件, 默认为标准输出
- `-passin` 指定私钥包含口令的存放方式, 如: `-passin file:pwd.txt`, `-passin pass:123456`

## genrsa

```shell
openssl genrsa -out server.key 2048
```

### 用途

