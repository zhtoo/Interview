## applicationId与包名区别

### applicationId的作用

每个Android应用都有一个唯一的应用ID（applicationId）。在Android设备和市场上，这个ID是你应用的唯一标识.若想在市场上更新应用,新应用的ID必须和原来apk的应用ID一致.所以一旦发布了应用,就不能再改变应用ID.

### applicationId与包名的定义方式

在Eclipse中没有applicationId这个概念，在Eclipse中applicationId即等同于包名。

###### 包名定义
在Android Studio中，这两个概念做个区分。包名的定义在清单文件中：

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    package="com.example.myapplication" > 

###### applicationId的定义
应用ID是在moudle层的build.gradle中定义，applicationId值即为应用ID,如下所示:

    android {  
    defaultConfig {  
        applicationId "com.example.myapplication"  
        minSdkVersion 15  
        targetSdkVersion 24  
        versionCode 1  
        versionName "1.0"  
    }  
    ...  
	}  

### 区别
在Android Studio中创建一个新项目时,applicationId默认是和项目的包名一致的。所以常常有开发者会将两者混淆，以为它们是一个概念。实际上,applicationId和包名是相互独立的。改变包名不会影响应用ID,反之亦然。

通常Android的applicationId与包名是绑定的,所以在Android API中,一些方法和参数从名称上看似乎它们返回的是包名，事实上它们返回的是applicationId值.例如,Context.getPackageName()方法返回的是applicationId,而不是包名。

applicationId的命名并不是随意的，它至少需要需遵循以下限制:

- applicationId至少包含两部分(也就是说至少有一个点,如com.example);
- 每部分必须以字母开头;
- 所有字符必须是字母数字或者下划线[a-zA-Z0-9_]

注意:
如果你使用了webview,请使用包名作为applicationId的前缀,否则,有可能会报错.

### applicationId作用

applicationId除了唯一标识了应用外，它在开发过程中还可以在一台测试机上实现同时装上测试版和发布版或者多个测试版本

###### 如何才能做到呢？

让测试版的applicationId与发布版不一致即可。

在buildTypes中修改开发版的applicationId。如下：

    android {  
    ...  
	    buildTypes {  
	        debug {  
	            applicationIdSuffix ".debug"   //等同于“com.example.myapplicationtest.debug”  
	        }  
    	}  
	}  

 
上面代码中，将debug版的应用ID在原来applicationId后追加“.debug”。applicationIdSuffix代表了默认的applicationId，即为defaultConfig中applicationId的值。所以，debug版的应用ID为："com.example.myapplicationtest.debug"。

另外，有时我们发布到市场的应用希望有不同的版本，比如：免费版和收费版。这就需要我们来构建不同的应用变体。那么我们可以在productFlavors中进行相应的配置，来生成不同的应用。如：

    android {
	    defaultConfig {
	        applicationId "com.example.myapplicationtest"
	    }
	    productFlavors {
	        free {
	            applicationIdSuffix ".free"
	        }
	        paid {
	            applicationIdSuffix ".paid"
	        }
	    }
	} 
在上面的代码中，我们用"free"表示免费版，用"paid"表示付费版。在productFlavors中，通过配置不同applicationId，最终生成不同的应用。最终这两种应用apk可同时存在于市场中。

###### 修改包名

默认情况下,包名与applicationId是相同的。当然，开发者也可以对包名进行修改.如果开发者想要修改包名的话,注意项目目录结构必须与AndroidManifest.xml中package属性的包名一致.

    <?xml version="1.0" encoding="utf-8"?>  
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    	package="com.example.myapplicationtest"  
    	android:versionCode="1"  
    	android:versionName="1.0" > 
 
### package值的作用:
1. 它为R.java文件提供了命名空间,例如R class的包名为com.example.myappcationtest.R
2. 决定manifest中声明的class的相对名称。如：
manifest中声明的<activity android:name=".MainActivity"> 的真实路径为：
com.example.myapplicationtest.ManiActivity。
如果开发者想修改包名,必须确保manifest中package值也做了同步修改。


### 补充
事实上编译完成之后applicationId会覆盖清单文件中的package值