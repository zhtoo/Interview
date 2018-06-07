## 收录开发过程中遇到的偏僻问题

### 应用签名（仅对AndroidStudio介绍）

#### 1. 利用AndroidStudio生成xxx.jks文件
	
- 打开AndroidStudio，点击 Build ---> Generate Signed APK... ---> Create new...
- 在新的弹框界面中 Key store path 后面的输入框中输入你想在哪里生成.jks文件（建议以.jks文件格式生成，如果是点击后面的‘...’选择路径的话，File name 后面的格式就只会有.jks）
- Password 密码  Confirm 再次输入密码
- Alias 别名（随便取一个即可）
- Alias下面需要再输入密码，可以和之前一样，也可以不一样，一般是设置一样，容易记
- Validity（years）证书有效时间 （这个默认25年，可以改，不该也行，按照需求来设置，一般APP也活不过25年）
- First and Last Name ：输入姓名，写自己的也可以，反正别人也不会看
- Organizational Unit ：公司名称/单位名称
- Organizational：所在的组织名称
- City or Locality ：市 
- State or Province ：省
- Countty Code（XX）：国家代码 （中国 CN）

#### 2. 获取xxx.jks文件中的签名
- 确保你的电脑安装了jdk，没有先去安装
- 打开角度看安装目录，一般在C:\Program Files\Java
- win + r ---> cnd --->在输入框输入 cd C:\Program Files\Java\jdk1.8.0_161\bin（jdk中bin的路径，这个除了cd ，其他的都由自己实际的路径为主） ，回车
- 找到你之前生成XXX.jks复制到bin目录下（避免文件找不到，或者密钥库文件不存在的错误）
- 在cnd命令窗口输入keytool -list -v -keystore xxx.jks（xxx.jks可以替换成你签名文件所在的正式路径） ，回车
- 输入密钥库口令 ，这里输入你之前设置的密码即可（不是别名的密码，输入的密码是不可感知的，也就是界面不会更具你的输入显示***或者其他的，所以在回车后直接输入密码即可，不要误以为没有输入，这段话针对小白的，是大神的话，你还来看个这个？）
- 回车之后你就会看到相应的数据，一般签名是指：证书指纹中的MD5或者SHA1。

#### 3.开发踩过的坑：
- 微信开发平台需要的签名是MD5

## 签名中的MD5和SHA1

首先要分清楚MD(Message Digest 信息摘要)5(第五代)和SHA1（Secure  Hash  Algorithm 安全哈希算法）并不是加密算法，应该归类为HASH（哈希）算法或者称之为摘要算法(Digest Algorithm)，即将无限制长度的字符串转换成固定长度(MD5是16个字符16*8=128bits，SHA1是20个字节=160bits 20*8bits的长度)。得到的是摘要，输入是无限制的字符。

虽然说现在有碰撞算法可以找到两个字符使用MD5生成为一样的MD5摘要字符，但是并不能说MD5是可逆的.即便是彩虹表也只是类似一个大数据库，在里面找相同的罢了。因为两个不同的字符串经过MD5算法之后，有可能生成相同的MD5或者SHA1值。这就像是X+Y=1024，X、Y的组合有许多种，并不能100%准确的指出，XY到底是两个什么值。

但是MD5和SHA1并不是加密算法。没有所谓的公钥私钥。而常见的加密算法有RSA，它属于是非对称的加密算法。公钥位于客户端，私钥位于服务器端。私钥是只有加密者才能拥有的。需要严加保护。【一个公钥对应一个私钥。】（有待考证）。一般来说生成私钥是需要用户自定义密码的，例如java中进行私钥生成的过程中，需要开发者自行生成私钥。公钥能够正确的解密出加密前的内容。

SHA1即安全哈希算法（Secure Hash Algorithm），用于签名；RSA是目前最有影响力的公钥加密算法。
说到这就的提到公钥和私钥：公钥、私钥分居客户端和服务器端，分别用于加密和解密。同时，私钥还用于签名，公钥还用于验证签名。

[参考地址](https://blog.csdn.net/tutuboke/article/details/53689266)