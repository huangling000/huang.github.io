用我的语言给你描述一下http和https：

首先要明白http是用于传输数据的协议，但是http中的数据是明文传输的，很容易通过一些简单的方式比如抓包将数据泄露，为了能够对传输数据进行加密，就有了https，他是在http的基础上使用了ssl/tsl协议对数据进行加密。

理解整个加密过程首先需要理解对称加密与非对称加密。

对称加密是客户端服务端使用同一个密钥对信息进行加密解密，算法公开，只要拥有密钥就能对信息进行加密解密，速度快，适用于大量数据传输，但缺点是需要考虑保证密钥的安全性。

非对称加密是，公钥能够对私钥加密的数据解密，密钥能够对公钥加密的数据解密，服务器可以将公钥发出去让客户端解密自己发送的数据，但客户端发回的通过公钥加密的数据只有服务端能够用私钥解密。非对称加密的优点是加密性能好，私钥不会进行传播，保证只有服务器能够解读密文，缺点是，效率较慢，比较适用于传输数据量小的数据。

因此https选择用对称加密的方式进行客户端与服务端之间的数据传输，用非对称加密的方式传输对称加密的密钥。

即客户端发起https请求，服务端会向客户端发送一个公钥。客户端会首先验证公钥的合法性（公钥私钥是在一个权威的网站上进行申请的），合法的话客户端会生成一个随机数作为一个后面使用的进行客户端与服务端之间传输使用的密钥。客户端用公钥对生成的密钥进行加密，发送给服务端，服务端对信息进行解密就拿到了密钥，这样服务端与客户端就可以通过这个密钥进行通信了。