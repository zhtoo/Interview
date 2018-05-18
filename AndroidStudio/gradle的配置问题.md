##  gradle的配置问题
参考地址：

[Gradle发行说明的Android插件](https://developer.android.com/studio/releases/gradle-plugin#revisions)

[配置构建](https://developer.android.com/studio/build/)
###  Instant Run

###### 启用 Instant Run（即时运行）

稳定版 Instant Run 是默认开启的（不像测试版需要手动开启），但 Gradle plugin 必须升级到 2.0 才可以。如果你在用 Instant Run 的过程中碰到了 问题（更新：已碰到，确实还有 bug），可以手动关闭 Settings -> Build, Execution, Deployment -> Instant Run。

###### 启用 Built-in shrinker（内置压缩）
Gradle plugin 2.0 新增了 built-in code shrinker（内置代码压缩），用来取代 ProGuard（一般用在 debug build），它只会压缩（shrink）代码而不会混淆（obfuscate），能够进一步加快 Instant Run 的运行速度。

开启方法是添加 useProguard false 到 build.gradle 文件里，即：

	android {
		...
	    buildTypes {
	        debug {
	            minifyEnabled true
	            useProguard false
	            proguardFiles getDefaultProguardFile('proguard-android.txt')
	        }
	        release {
	            minifyEnabled true
	            useProguard true
	            proguardFiles getDefaultProguardFile('proguard-android.txt')
	        }
	    }
		...
	}

这样在 debug build 时只会压缩代码，在 release build 时才压缩与混淆代码。
useProguard true 是默认设置，可以不用添加。

###### 启用 Resource Shrinking（资源压缩）
这是一个很早就加入的功能，但之前一直有bug，2.0 做了修复。它的作用是移除没有使用的资源（Resources，包括第三方库里的），必须配合 ProGuard（或其他代码压缩工具）使用，启用方法是：

	android {
		...
	    buildTypes {
	        release {
	            minifyEnabled true
	            shrinkResources true  // 启用 Resource Shrinking
	        }
	    }
		...
	}


###支持 Incremental compilation（增量编译）
	compileOptions {
    	incremental=true|false
	}
默认是关闭，开启能够加快编译速度，但可能导致 R class 无法正确的重新编译。

###  lint配置 

lintOptions：lint检查，用与检测app可能出现的问题。 

    android {
	    lintOptions {
	        // true--关闭lint报告的分析进度
	        quiet true
	        // true--错误发生后停止gradle构建
	        abortOnError false
	        // true--只报告error
	        ignoreWarnings true
	        // true--忽略有错误的文件的全/绝对路径(默认是true)
	        //absolutePaths true
	        // true--检查所有问题点，包含其他默认关闭项
	        checkAllWarnings true
	        // true--所有warning当做error
	        warningsAsErrors true
	        // 关闭指定问题检查
	        disable 'TypographyFractions','TypographyQuotes'
	        // 打开指定问题检查
	        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'
	        // 仅检查指定问题
	        check 'NewApi', 'InlinedApi'
	        // true--error输出文件不包含源码行号
	        noLines true
	        // true--显示错误的所有发生位置，不截取
	        showAll true
	        // 回退lint设置(默认规则)
	        lintConfig file("default-lint.xml")
	        // true--生成txt格式报告(默认false)
	        textReport true
	        // 重定向输出；可以是文件或'stdout'
	        textOutput 'stdout'
	        // true--生成XML格式报告
	        xmlReport false
	        // 指定xml报告文档(默认lint-results.xml)
	        xmlOutput file("lint-report.xml")
	        // true--生成HTML报告(带问题解释，源码位置，等)
	        htmlReport true
	        // html报告可选路径(构建器默认是lint-results.html )
	        htmlOutput file("lint-report.html")
	        //  true--所有正式版构建执行规则生成崩溃的lint检查，如果有崩溃问题将停止构建
	        checkReleaseBuilds true
	        // 在发布版本编译时检查(即使不包含lint目标)，指定问题的规则生成崩溃
	        fatal 'NewApi', 'InlineApi'
	        // 指定问题的规则生成错误
	        error 'Wakelock', 'TextViewEdits'
	        // 指定问题的规则生成警告
	        warning 'ResourceAsColor'
	        // 忽略指定问题的规则(同关闭检查)
	        ignore 'TypographyQuotes'
	    }
	}

当你出现：多语言未翻译问题的时候

可以在lint.xml中加入

    <issue
	id="MissingTtanslation"
	severity="ignore"
	/>

或者，按照studio的提示来

打开项目目录下的build.gradle文件（这就是gradle的配置文件，gradle就是编译工具了），

然后按照提示一股脑把新参数全部加到android里面。

    android {
		lintOptions{
			//关闭在打包Release版本的时候进行检测
			checkReleaseBuilds false
			//检测到错误不停止打包
			abortOnError false
		}
	}


这样打包就不成问题了。checkReleaseBuilds就是在打包Release版本的时候进行检测，这里就直接关掉了，也可以打开，这样报错还会显示出来。关键的就是abortOnError一定要设为false，这样即使有报错也不会停止打包了。