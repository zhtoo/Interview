## 谷歌强推 AndroidX ,你还在应Support？（转）

链接：http://www.apkbus.com/blog-822715-81875.html
作者：littleRed

AndroidX 是 Google 2018 IO 大会推出的新扩展库，主要是对 Android支持库做了重大改进。与支持库一样，AndroidX 与 Android 操作系统分开提供，并与各个 Android 版本向后兼容，可以说 AndroidX 就是为了替换 Android 支持库而设计的。 

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_1.png)

### AndroidX 是什么？
- AndroidX 是 Android 团队用于在Jetpack中开发、测试、打包和发布库以及对其进行版本控制的开源项目。[摘自官方]
- AndroidX 完全取代了支持库，不仅提供同等的功能，而且提供了新的库。
- AndroidX 会将原始支持库 API 软件包映射到 androidx 命名空间。只有软件包和 Maven 工件名称发生了变化；类、方法和字段名称没有改变。
- 与支持库不同，AndroidX 软件包会单独维护和更新。androidx 软件包使用严格的语义版本控制，从版本 1.0.0 开始，可以单独更新项目中的 AndroidX 库。
- 所有新支持库的开发工作都将在 AndroidX 库中进行，这包括维护原始支持库工件和引入新的 Jetpack 组件。

### AndroidX 的变化

常见依赖库映射
![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_2.png)

更多详细依赖库变化，可查阅官方文档
(https://developer.android.com/jetpack/androidx/migrate#artifact_mappings)
或下载这些映射的 CSV 格式
(https://developer.android.com/topic/libraries/support-library/downloads/androidx-artifact-mapping.csv)文件。

常见类映射
![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_3.png)

更多详细支持类映射变化，可查阅官方文档(https://developer.android.com/jetpack/androidx/migrate#artifact_mappings)或下载这些映射的CSV 格式(https://developer.android.com/topic/libraries/support-library/downloads/androidx-class-mapping.csv)文件。
### 为什么要迁移 AndroidX？
下面是 Google 官方描述


    Existing packages, such as the Android Support Library, are being refactored into AndroidX.
    Although Support Library versions 27 and lower are still available on Google Maven,
    all new development will be included in only AndroidX versions 1.0.0 and higher.

大致意思是：现有的软件包，如 Android 支持库，正在被重构为 Androidx。尽管在 Google Maven 上仍然提供支持库版本 27 及更低版本，但所有新开发将只包含在 Androidx 1.0.0 及更高版本中。
### AndroidX 迁移步骤？
**1. 更新 Android Studio 与 Gradle 版本**

- 将 Android studio 升级到 3.2 及以上；
- Gradle 插件版本改为 4.6 及以上；
- compileSdkVersion 版本升级到 28 及以上；
- buildToolsVersion 版本改为 28.0.2 及以上。

**2. 迁移 AndroidX 配置**

- 在项目的gradle.properties文件里添加如下配置：
    `android.useAndroidX=trueandroid.enableJetifier=true`

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_4.png)

备注：enableJetifier 如果取值为 false, 表示不迁移依赖包到 androidx，但在使用依赖包中的内容时可能会出现问题，当然了，如果你的项目中没有使用任何三方依赖，那么，此项可以设置为 false。


**3.修改依赖库**


修改项目app 目录下的 build.gradle 依赖库，具体可以参照 AndroidX 变化中的依赖库映射。

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_5.png)

**4. 依赖类重新导包**

将原来 import 的 android. 包删除，重新 import 新的 androidx. 包

    import android.support.v7.app.AppCompatActivity; 
    → import androidx.appcompat.app.AppCompatActivity;

**5. 一键迁移 AndroidX 库**

AS 3.2 及以上版本提供了更加方便快捷的方法一键迁移到 AndroidX。选择菜单上的 ReFactor —— Migrate to AndroidX… 即可。（如果迁移失败，就需要重复上面 1，2，3，4 步手动去修改迁移）

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_6.png)

备注：如果你的项目 compileSdkVersion 低于 28，点击 Refactor to AndroidX… 会提示：

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_7.png)

### Q&A
**同一个项目中 Android Support 和 AndroidX 可以共存吗？**

不可以共存。需要将依赖修改为Android Suppor或AndroidX中任一种。

**执行 Migrate to AndroidX 之后就完成 AndroidX 迁移了？**

不一定。部分控件的包名/路径名转换的有问题，所以还需要我们手动调整（包括修改xml布局文件和.java/.kt 文件）。

**DataBinding 中的错误（重名 id 错误）？**

在 AndroidStudio3.2 + androidx 环境下，对错误的检查和处理更为严格。如果同一个xml布局文件中存在同名id，在之前的版本中，我们可以正常编译和运行，但是，在新的环境下， 必然会报错，错误信息如下：

![](https://github.com/zhtoo/Interview/blob/master/picture/MigrateToandroidX/MigrateToandroidX_8.png)

**attr.xml 中重复的属性名称会报错？**

在迁移到 androidX 之前，我们为自定义控件编写自定义属性时，可以与android已有的属性重名，但是，在AndroidX环境下不行了，如果存在重名的情况，必然会报错——会提示你重复定义。

错误示例

    <declare-styleable name="RoundImageView">...
    <!-在迁移到androidx之前，这样写虽然不规范，但是能用，不报错->
    <attr name="textSize" format="Integer" />...</declare-styleable>

正确示例

    …<!-迁移到androidX之后，必须使用android:xxx 属性，不能定义android已有的属性->…
 
**Glide 中的注解不兼容 androidX？**

迁移到 androidX 之后，Glide 中使用的

android.support.annotation.CheckResult 

和 android.support.annotation.Non 这两个注解无法迁移。

之前有用户在 Glide 中提过 issue: 

https://github.com/bumptech/glide/issues/3185

在上述 issue 中有用户表示，将 Glide 升级到 4.8.0 之后，可以正常迁移。但是，我这边并不行。然后，我先升级了 Glide , 又在 gralde 文件中增加了 support.annotation ，这样才能正常编译通过。貌似在后续 Glide 5.x 版本中会完成对 androidx 的完全兼容。

**规范包名（即文件夹名）?**

这里所说的包名，指的是项目中的文件夹名称。在之前版本中，我们命名包名时可能会出现大写字母，虽然这并不符合 Java 命名规范，但起码能正常编译和运行。然而，升级到 AndroidStudio3.2 + androidX 环境后，必须严格遵守命名规范，否则，可能报错，从而导致不能正常编译和运行。
