## Handler消息处理

### Handler流程

Zygote启动ActivityThread.main()

> ActivityThread.main() 

 	public static void main(String[] args) {
     	//...
        Looper.prepareMainLooper();
       	//...
		Looper.loop();
    }


> Looper.prepareMainLooper()

	public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
           	//...
            sMainLooper = myLooper();
        }
    }

 	private static void prepare(boolean quitAllowed) {
        sThreadLocal.set(new Looper(quitAllowed));
    }

	private Looper(boolean quitAllowed) {
		//创建一个新的MessageQueue（消息队列）
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }

	/**
     * 返回与当前线程关联的Looper对象。 如果调用线程未与Looper关联，则返回null。
     */
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

> MessageQueue类

	MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }

> Looper.loop()

      public static void loop() {

        final Looper me = myLooper();
        final MessageQueue queue = me.mQueue;

        for (;;) {
            Message msg = queue.next();
			//...
            try {
                msg.target.dispatchMessage(msg);
            } finally {
            	//...
            } 
            msg.recycleUnchecked();
        }
    }

> Handler

 	public Handler() {
        this(null, false);
    }
	//创建一个Handler对象
	public Handler(Callback callback, boolean async) {
        mLooper = Looper.myLooper();
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
	
	//举例:发送一个消息
	public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }

	public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        return enqueueMessage(queue, msg, uptimeMillis);
    }

	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
		//将handler放入Message中
        msg.target = this;
		//将消息放进消息队列中
        return queue.enqueueMessage(msg, uptimeMillis);
    }

	public void dispatchMessage(Message msg) {
        handleMessage(msg); 
    }

> MessageQueue.enqueueMessage()

	boolean enqueueMessage(Message msg, long when) {
        synchronized (this) {
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;//Message--->prev
                    p = p.next;//
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            } 
        }
        return true;
    }

Q&A
#### 如何知道Looper是在 Zygote启动ActivityThread.main()执行初始化的？
不知道，自己百度

#### 从Handler的创建和发送流程中是否能分析到全部的流程？
不知道，自己百度
#### Zygote是如何启动ActivityThread.main()？
不知道，自己百度

### 待处理，有兴趣的可以看看

怎么查看启动activity的调用信息？

直接在onCreate（）里面抛出一个方法就行。

下面是启动App的过程
 
	at com.zht.common.base.BaseActivity.onCreate(BaseActivity.java:32)
    at android.app.Activity.performCreate(Activity.java:7372)
    at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1218)
    at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3147)
    at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3302)
    at android.app.ActivityThread.-wrap12(Unknown Source:0)
    at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1891)
    at android.os.Handler.dispatchMessage(Handler.java:108)
    at android.os.Looper.loop(Looper.java:166)
    at android.app.ActivityThread.main(ActivityThread.java:7425)
    at java.lang.reflect.Method.invoke(Native Method)
    at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:245)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:921)


下面是启动Activity--->startActivity--->Activity的过程,

	at com.zht.common.base.BaseActivity.onCreate(BaseActivity.java:32)
	at android.app.Activity.performCreate(Activity.java:7372)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1218)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3147)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3302) 
	at android.app.ActivityThread.-wrap12(Unknown Source:0) 
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1891) 
	at android.os.Handler.dispatchMessage(Handler.java:108) 
	at android.os.Looper.loop(Looper.java:166) 
	at android.app.ActivityThread.main(ActivityThread.java:7425) 
	at java.lang.reflect.Method.invoke(Native Method) 
	at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:245) 
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:921) 
 

好吧一样。

然后来看看具体流程

> ZygoteInit.main

	public static void main(String argv[]) {
		//创建一个ZygoteServer服务
        ZygoteServer zygoteServer = new ZygoteServer();
		//标记Zygote开始。这样可以确保线程创建将引发错误。
		//确保在启动Zygote时，没有线程被创建。
        ZygoteHooks.startZygoteNoThreadCreation();

        try {
			//设置id：
            Os.setpgid(0, 0);
        } catch (ErrnoException ex) {
            throw new RuntimeException("Failed to setpgid(0,0)", ex);
        }
		//创建一个Runnable
        final Runnable caller;
        try {
            if (!"1".equals(SystemProperties.get("sys.boot_completed"))) {
                MetricsLogger.histogram(null, "boot_zygote_init",
                        (int) SystemClock.elapsedRealtime());
            }
			
			
            String bootTimeTag = Process.is64Bit() ? "Zygote64Timing" :"Zygote32Timing";

            TimingsTraceLog bootTimingsTraceLog = new TimingsTraceLog(bootTimeTag,
                    Trace.TRACE_TAG_DALVIK);

            bootTimingsTraceLog.traceBegin("ZygoteInit");

            RuntimeInit.enableDdms();

            boolean startSystemServer = false;

            String socketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            if (abiList == null) {
                throw new RuntimeException("No ABI list supplied.");
            }

            zygoteServer.registerServerSocket(socketName);
          
            if (!enableLazyPreload) {
                bootTimingsTraceLog.traceBegin("ZygotePreload");
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_START,
                    SystemClock.uptimeMillis());
                preload(bootTimingsTraceLog);
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_END,
                    SystemClock.uptimeMillis());
                bootTimingsTraceLog.traceEnd(); 
            } else {
                Zygote.resetNicePriority();
            }
         
            bootTimingsTraceLog.traceBegin("PostZygoteInitGC");
            gcAndFinalize();
            bootTimingsTraceLog.traceEnd(); 

            bootTimingsTraceLog.traceEnd(); 

            Trace.setTracingEnabled(false, 0);

            Zygote.nativeUnmountStorageOnInit();

            Seccomp.setPolicy();

            ZygoteHooks.stopZygoteNoThreadCreation();

            if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);

                if (r != null) {
                    r.run();
                    return;
                }
            }

            Log.i(TAG, "Accepting command socket connections");

            caller = zygoteServer.runSelectLoop(abiList);
        } catch (Throwable ex) {
            Log.e(TAG, "System zygote died with exception", ex);
            throw ex;
        } finally {
            zygoteServer.closeServerSocket();
        }

        if (caller != null) {
            caller.run();
        }
    }

> Zygote$MethodAndArgsCaller.run

