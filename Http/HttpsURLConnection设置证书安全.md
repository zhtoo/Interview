### HttpsURLConnection设置证书安全

#### 先上代码

     
	//新建一个自己的主机名称验证方法
	HostnameVerifier hostnameVerifier = new HostnameVerifier() {
        public boolean verify(String urlHostName, SSLSession session) {
			//打印日志，可省略此处代码
            System.out.println("Warning: URL Host: " + urlHostName + " vs. "
                    + session.getPeerHost());
            //直接返回true
            return true;
        }
    };

    private static void trustAllHttpsCertificates() throws Exception {
        javax.net.ssl.TrustManager[] trustAllCerts = new javax.net.ssl.TrustManager[1];
        javax.net.ssl.TrustManager tm = new MyTrustManager();
        trustAllCerts[0] = tm;
        javax.net.ssl.SSLContext sc = javax.net.ssl.SSLContext
                .getInstance("SSL");
        sc.init(null, trustAllCerts, null);
        javax.net.ssl.HttpsURLConnection.setDefaultSSLSocketFactory(sc
                .getSocketFactory());
    }

    static class MyTrustManager implements javax.net.ssl.TrustManager,
            javax.net.ssl.X509TrustManager {
        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
            return null;
        }

        public boolean isServerTrusted(
                java.security.cert.X509Certificate[] certs) {
            return true;
        }

        public boolean isClientTrusted(
                java.security.cert.X509Certificate[] certs) {
            return true;
        }

        public void checkServerTrusted(
                java.security.cert.X509Certificate[] certs, String authType)
                throws java.security.cert.CertificateException {
            return;
        }

        public void checkClientTrusted(
                java.security.cert.X509Certificate[] certs, String authType)
                throws java.security.cert.CertificateException {
            return;
        }
    }

### 使用

     	//这里的代码在new URL(mData)使用
		trustAllHttpsCertificates();
     	HttpsURLConnection.setDefaultHostnameVerifier(hostnameVerifier);

### 原因

https通信过程
客户端在使用HTTPS方式与Web服务器通信时有以下几个步骤，如图所示。

1. 客户使用https的URL访问Web服务器，要求与Web服务器建立SSL连接。
1. Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端。
1. 客户端的浏览器与Web服务器开始协商SSL连接的安全等级，也就是信息加密的等级。
1. 客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。
1. Web服务器利用自己的私钥解密出会话密钥。
1. Web服务器利用会话密钥加密与客户端之间的通信。

 ![](https://github.com/zhtoo/Interview/blob/master/picture/HttpsURLConnection设置证书安全.png)


	 