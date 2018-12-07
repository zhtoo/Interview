### 持有WakeLock
获取WakeLock相关权限：

    <uses-permission android:name="android.permission.WAKE_LOCK" />

通过PowerManager获取WakeLock，在onResume以及onPause执行相应操作：
   
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        PowerManager powerManager = (PowerManager)getSystemService(POWER_SERVICE);
        if (powerManager != null) {
            mWakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK, "WakeLock");
        }
    }
 
    @Override
    protected void onResume() {
        super.onResume();
        if (mWakeLock != null) {
            mWakeLock.acquire();
        }
    }
 
    @Override
    protected void onPause() {
        super.onPause();
        if (mWakeLock != null) {
            mWakeLock.release();
        }
    }

WakeLock获取时相关的flag如下所示：
- PARTIAL_WAKE_LOCK :保持CPU 运转，屏幕和键盘灯有可能是关闭的。
- SCREEN_DIM_WAKE_LOCK ：保持CPU 运转，允许保持屏幕显示但有可能是灰的，允许关闭键盘灯
- SCREEN_BRIGHT_WAKE_LOCK ：保持CPU 运转，允许保持屏幕高亮显示，允许关闭键盘灯
- FULL_WAKE_LOCK ：保持CPU 运转，保持屏幕高亮显示，键盘灯也保持亮度

PS：现在官方已经不推荐使用这种方式保持亮屏了，推荐以下两种方式

### 在Window设置flag

    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

这种方式不需要申请权限，也是官方推荐的做法

### 在界面布局xml中顶层添加属性

在界面xml文件中的顶层布局添加属性：

    android:keepScreenOn="true"
     