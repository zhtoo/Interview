### windowSoftInputMode

AndroidManifest.xml文件中界面对应的<activity>里加入

	<activity android:name="xxx.xxx.xxx"
            android:windowSoftInputMode="XXX"
            />

	
值|作用
---|---
stateUnspecified|软键盘的状态 (是否它是隐藏或可见 )没有被指定。系统将选择一个合适的状态或依赖于主题的设置。这个是为了软件盘行为默认的设置。
stateUnchanged|软键盘被保持无论它上次是什么状态，是否可见或隐藏，当主窗口出现在前面时。
stateHidden|当用户选择该 Activity时，软键盘被隐藏——也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个 Activity。
stateAlwaysHidden|软键盘总是被隐藏的，当该 Activity主窗口获取焦点时。
stateVisible|软键盘是可见的，当那个是正常合适的时 (当用户导航到 Activity主窗口时 )。
stateAlwaysVisible|当用户选择这个 Activity时，软键盘是可见的——也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个Activity。
adjustUnspecified|它不被指定是否该 Activity主 窗口调整大小以便留出软键盘的空间，或是否窗口上的内容得到屏幕上当前的焦点是可见的。系统将自动选择这些模式中一种主要依赖于是否窗口的内容有任何布局 视图能够滚动他们的内容。如果有这样的一个视图，这个窗口将调整大小，这样的假设可以使滚动窗口的内容在一个较小的区域中可见的。这个是主窗口默认的行为 设置。
adjustResize|该 Activity主窗口总是被调整屏幕的大小以便留出软键盘的空间
adjustPan|该 Activity主窗口并不调整屏幕的大小以便留出软键盘的空间。相反，当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分。这个通常是不期望比调整大小，因为用户可能关闭软键盘以便获得与被覆盖内容的交互操作。




 

