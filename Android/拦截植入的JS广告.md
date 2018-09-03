网页上的广告一般是 在网页植入一段js代码，要想屏蔽广告只需要将这些js屏蔽掉即可。

### WebViewClient的几个回调函数

1. public boolean **shouldOverrideUrlLoading**(WebView view, String url) ：
在点击请求的是链接是才会调用，重写此方法返回true表明点击网页里面的链接还是在当前的webview里跳转，不跳到浏览器那边。
 
2. public void **onReceivedSslError**(WebView view, SslErrorHandler handler, android.net.http.SslError error)：
重写此方法可以让webview处理https请求。

3. public boolean **shouldOverrideKeyEvent** (WebView view, KeyEvent event)：
重写此方法才能够处理在浏览器中的按键事件。

4. public void **onLoadResource**(WebView view, String url) ：
在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。

5. public void **onPageStarted**(WebView view, String url, Bitmap favicon) ：
在页面加载开始时调用。

6. public void **onPageFinished**(WebView view, String url) ：
在页面加载结束时调用。


拦截广告就是拦截加载广告的js，上面的onLoadResource似乎是很合适的函数，只要判断onLoadResource的参数url是否是加载广告js的即可，如果不是广告相关的url正常加载，如果是则不加载。但是在使用onLoadResource之后才发现根本不行。

这里引用WebViewClient另外一个回调函数：

public WebResourceResponse **shouldInterceptRequest**(WebView view, String url)

shouldInterceptRequest有两种重载。

public WebResourceResponse shouldInterceptRequest (WebView view, String url) 从API 11开始引入，API 21弃用

public WebResourceResponse shouldInterceptRequest (WebView view, WebResourceRequest request) 从API 21开始引入


### 使用shouldInterceptRequest (WebView view, String url)完成对webview广告的拦截。

#### 拦截广告资源URL

在Webview加载资源时会回调shouldInterceptRequest函数，我们可以通过重写shouldInterceptRequest函数实现对webview的资源请求进行处理。进行处理后返回数据。如果主程序返回的数据为null，WebView会自行请求网络加载资源。不是shouldInterceptRequest函数返回null就能屏蔽掉请求！正确的屏蔽请求的方式：

		@Override
	    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
	        url = url.toLowerCase();
			 if (!ADFilterTool.hasAd(context, url)) {
				return super.shouldInterceptRequest(view, url);//正常加载
			 }else{
				return new WebResourceResponse(null,null,null);//含有广告资源屏蔽请求
			}
	}


### 屏蔽广告的NoAdWebViewClient类：
 只需使用webview.setWebViewClient(NoAdWebViewClient webclient)即可屏蔽指定webview的广告。

NoAdWebViewClient 屏蔽广告

	import android.content.Context;
	import android.util.Log;
	import android.webkit.WebResourceResponse;
	import android.webkit.WebView;
	import android.webkit.WebViewClient;
	 
	public class NoAdWebViewClient extends WebViewClient {
	    private  String homeurl;
	    private Context context;
	 
	    public NoAdWebViewClient(Context context,String homeurl) {
	        this.context = context;
	        this.homeurl = homeurl;
	    }
	    @Override
	    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
	        url = url.toLowerCase();
	        if(!url.contains(homeurl)){
	            if (!ADFilterTool.hasAd(context, url)) {
	                return super.shouldInterceptRequest(view, url);
	            }else{
	                return new WebResourceResponse(null,null,null);
	            }
	        }else{
	            return super.shouldInterceptRequest(view, url);
	        }
	 
	 
	    }
	}


### 判断URL是否含广告的ADFilterTool类：该类通过判断url是否包含在广告拦截库中

	import android.content.Context;
	import android.content.res.Resources;
	import android.util.Log;
	 
	public class ADFilterTool {
	    public static boolean hasAd(Context context, String url) {
	        Resources res = context.getResources();
	        String[] adUrls = res.getStringArray(R.array.adBlockUrl);
	        for (String adUrl : adUrls) {
	            if (url.contains(adUrl)) {
	                return true;
	            }
	        }
	        return false;
	    }
	}

### 广告Url资源文件（广告拦截库可自行百度更新）：AdUrlString.Xml

所谓广告拦截库，实际上是请求广告资源的url合集，网络上有大量的广告拦截库，读者可以定期更新一下文件来实现对广告的高效过滤。本文屏蔽的方式比较粗暴，凡是含有广告资源的域名统统禁止。要想实现更精准的过滤，访友你可以使用通配符匹配url的方式进行拦截，现在PC端的浏览器正是这样做的。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <string-array name="adBlockUrl">
	        <item>ubmcmm.baidustatic.com</item>
	        <item>cpro2.baidustatic.com</item>
	        <item>cpro.baidustatic.com</item>
	        <item>s.lianmeng.360.cn</item>
	        <item>nsclick.baidu.com</item>
	        <item>pos.baidu.com</item>
	        <item>cbjs.baidu.com</item>
	        <item>cpro.baidu.com</item>
	        <item>images.sohu.com/cs/jsfile/js/c.js</item>
	        <item>union.sogou.com/</item>
	        <item>sogou.com/</item>
	        <item>a.baidu.com</item>
	        <item>c.baidu.com</item>
	 
	    </string-array>
	</resources>
 
PS：这中拦截也可以根实际情况来定义，比如在APP中H5的地址是固定的几个，我们就以这些地址为判断条件，相同通行，不同拦截。