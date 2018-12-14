# debug模式和Attach模式的断点调试

## debug步骤：
### 1、设置断点（点击红点位置添加或取消断点）
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_01.png)
### 2、点击debug模式运行
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_02.png)
### 3、查看调试面板
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_03.png)
## 一、简单调试
### 1. step over（F8）：一步步执行程序代码
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_11.png)

点击调试按钮或按快捷键F8，程序会执行下一行代码。 

### 2. step into(F7)：执行方法里面的代码 
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_12.png)
点击调试按钮或按快捷键F7，如果某行执行的代码是某一个方法，点击此按钮就是进入方法里面，执行该方法里面的代码。
### 3. force step into (alt+shift+F7):执行所有过程的代码
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_23.png)
点击调试按钮或按快捷键alt+shift+F7，将会执行调用方法的具体实现代码，也是是程序执行的所有过程，这个用来研究源码使用非常方便。

### 4. step out(shift+F8) ：如果有下一个断点，执行下一个断点，然后继续执行
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_14.png)
点击调试按钮或按快捷键shift+F8。如果我们的一个流程当中，包括调用的方法中，如果有断点，走到下一个断点，如果没有断点，而是在一个调用的方法当中，会跳出这个方法，继续走。

### 5. run to Cursor (alt+F9)：下个断点我们见
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_15.png)

点击调试按钮或按快捷键alt+F9，会很快执行到下一个断点的位置，而且可以进入任何调用的方法

## 二、高级调试

### 1. 跨断点调试
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_21.png)

如果我们设置了多个断点，此时我们需要直接跳转到下一个断点，那么直接点击此按钮

### 2.观察变量
如果我们想观察1个或者几个变量的值的变化，如果我们在Variables显示面版中观察如果我这里有太多太多的自定义变量和系统变量了，那么就难观察了，我们可以做如下操作：
点击Watches,点击＋号，然后输入变量的名称回车就OK了，而且会有历史记录

如果变量名比较长我们可以选择［Variables］中的变量名然后点击［右键］，选择［Add to Watches],然后就会出现在Watches面板中。

### 3.设置变量的值
在程序中有很多的条件语句和循环语句，调试也是比较耗时的，我们可以通过快速设置变量的值来加快调试速度，我们可以做如下操作：
选择［Variables］中的变量名然后点击［右键］，选择［Set Value..]或者选择之后直接F2

### 4.查看断点
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_24.png)
点击之后我们可以看到所有的断点，以及位置代码,也可以设置一些属性

### 5.停止调试
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_25.png)

要注意的是这里的［停止调试］不是让程序停止，而是跳过所有调试

到这里我们的Android Studio的断点调试和高级调试就完毕了。

## Attach模式调试
应用场景：APK已经运行在普通模式（非Debug）的情况下，突然需要Debug，而又不想重新运行浪费时间。 
 
### 1、先设置断点，再正常运行APK 
### 2、点击Attach调试
![](https://github.com/zhtoo/Interview/blob/master/AndroidStudio/picture/debug_32.png)
或者像下面一样打开也是可以的

即运行Run->Attach debugger to Android process 

attach process到指定进程，条件触发之后就可以直接进入调试模式。