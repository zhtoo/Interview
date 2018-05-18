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
- 回车之后你就会看到相应的数据，一般签名是指：证书指纹中的SHA1。