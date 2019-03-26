### VideoView源码分析及重新封装



VideoView是Android提供播放视频的一个控件，继承于SurfaceView，自己维护了一个MediaPlayer，同时人性化的提供了对视频操作的类MediaControl（google自己写的样式。额，不推荐直接使用，可以自己继承MediaController，对其样式进行修改。）

### 类注释 
（参照google翻译，原文请自行阅读源码）

显示视频文件。 VideoView类可以从各种来源（例如资源或内容提供商）加载图像，负责从视频计算其测量值，以便可以在任何布局管理器中使用，并提供各种显示选项，如缩放和着色。

进入后台时，VideoView不会保持其完整状态。

特别是，它不会恢复当前播放状态，播放位置，通过{@link #addSubtitleSource addSubtitleSource（）}添加的任何字幕轨道。 应用程序应在{@link android.app.Activity＃onSaveInstanceState}和{@link android.app.Activity＃onRestoreInstanceState}中自行保存和恢复这些内容。

另请注意，恢复VideoView时，音频会话ID（来自{@link #getAudioSessionId}）可能会更改其先前返回的值。

默认情况下，VideoView使用{@link AudioManager＃AUDIOFOCUS_GAIN}请求获取音频焦点。 使用{@link #setAudioFocusRequest（int）}更改此行为。

播放期间使用的默认{@link AudioAttributes}使用{@link AudioAttributes＃USAGE_MEDIA}和内容类型{@link AudioAttributes＃CONTENT_TYPE_MOVIE}，使用{@link #setAudioAttributes（AudioAttributes）}来修改它们。

大概意思是：
1. 可以在任意ViewGroup中使用。
2. VideoView不会保存视频的状态，需要自行在onSaveInstanceState和onRestoreInstanceState进行视频的状态保存。
3. 默认的音频属性是AudioAttributes.USAGE_MEDIA和AudioAttributes.CONTENT_TYPE_MOVIE，可以通过setAudioAttributes（）方法来修改。


### 源码分析

#### 构造

 	public VideoView(Context context) {
        this(context, null);
    }

    public VideoView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VideoView(Context context, AttributeSet attrs, int defStyleAttr) {
        this(context, attrs, defStyleAttr, 0);
    }

    public VideoView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
		//... 具体代码往下看。
    }

重点看VideoView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes)。

	public VideoView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
		//初始化视频的宽高
        mVideoWidth = 0;
        mVideoHeight = 0;
		//初始化音频管理器
        mAudioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
		//初始化音频流的信息。
        mAudioAttributes = new AudioAttributes.Builder().setUsage(AudioAttributes.USAGE_MEDIA)
                .setContentType(AudioAttributes.CONTENT_TYPE_MOVIE).build();
		//自行查阅SurfaceView，这个类讲起来又会很长
        getHolder().addCallback(mSHCallback);
        getHolder().setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
		
        setFocusable(true);
        setFocusableInTouchMode(true);
        requestFocus();
		//当前状态：空闲
        mCurrentState = STATE_IDLE;
		//标记状态：空闲
        mTargetState = STATE_IDLE;
    }

状态：
	// all possible internal states //所有可能的内部状态，这里是指视频播放的状态。
	
    private static final int STATE_ERROR = -1;//错误状态
    private static final int STATE_IDLE = 0;//空闲
    private static final int STATE_PREPARING = 1;//准备中
    private static final int STATE_PREPARED = 2;//准备完成
    private static final int STATE_PLAYING = 3;//播放
    private static final int STATE_PAUSED = 4;//暂停
    private static final int STATE_PLAYBACK_COMPLETED = 5;//已完成

#### onMeasure（测量）

 	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = getDefaultSize(mVideoWidth, widthMeasureSpec);
        int height = getDefaultSize(mVideoHeight, heightMeasureSpec);
        if (mVideoWidth > 0 && mVideoHeight > 0) {

            int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
            int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
            int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
            int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

            if (widthSpecMode == MeasureSpec.EXACTLY && heightSpecMode == MeasureSpec.EXACTLY) {
              
                width = widthSpecSize;
                height = heightSpecSize;

				//为了兼容性，根据宽高比调整大小
                if ( mVideoWidth * height  < width * mVideoHeight ) {
                    width = height * mVideoWidth / mVideoHeight;
                } else if ( mVideoWidth * height  > width * mVideoHeight ) {
                    height = width * mVideoHeight / mVideoWidth;
                }
            } else if (widthSpecMode == MeasureSpec.EXACTLY) {
                width = widthSpecSize;
                height = width * mVideoHeight / mVideoWidth;
                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
                    height = heightSpecSize;
                }
            } else if (heightSpecMode == MeasureSpec.EXACTLY) {
                height = heightSpecSize;
                width = height * mVideoWidth / mVideoHeight;
                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) { 
                    width = widthSpecSize;
                }
            } else {
                width = mVideoWidth;
                height = mVideoHeight;
                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
                    height = heightSpecSize;
                    width = height * mVideoWidth / mVideoHeight;
                }
                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) {
                    width = widthSpecSize;
                    height = width * mVideoHeight / mVideoWidth;
                }
            }
        } else {
			//没有尺寸，只需采用给定的尺寸
        }
        setMeasuredDimension(width, height);
    }

这里做的其实就是尺寸自适应，保证视频按照原先宽高比播放。（源码就不分析了，对自定义控件不熟悉的童鞋可以看看别人的实现思路。）


#### setVideoURI

 	public void setVideoURI(Uri uri, Map<String, String> headers) {
        mUri = uri;
        mHeaders = headers;
        mSeekWhenPrepared = 0;//当准备完成时，跳转到视频的某个时刻
		//重点看这个方法
        openVideo();
		//当某些内容发生变化而导致此视图的布局无效时，请调用此方法。 这将安排视图树的布局传递。 当视图层次结构当前处于布局传递中时，不应调用此方法（{@link #isInLayout（）}。如果发生布局，则可以在当前布局传递结束时接受请求（然后布局将再次运行） ）或在绘制当前帧并发生下一个布局之后。
        requestLayout();
		//重绘
        invalidate();
    }

####  openVideo
 	private void openVideo() {
        if (mUri == null || mSurfaceHolder == null) {
           //非空判断
            return;
        }
   		//释放mMediaPlayer，但是不清除视频播放的状态
        release(false);

        if (mAudioFocusType != AudioManager.AUDIOFOCUS_NONE) {
            //请求获取自动获取焦点
            mAudioManager.requestAudioFocus(null, mAudioAttributes, mAudioFocusType, 0);
        }

        try {
			//初始化MediaPlayer
            mMediaPlayer = new MediaPlayer();
         
            final Context context = getContext();
            final SubtitleController controller = new SubtitleController(
                    context, mMediaPlayer.getMediaTimeProvider(), mMediaPlayer);
            controller.registerRenderer(new WebVttRenderer(context));
            controller.registerRenderer(new TtmlRenderer(context));
            controller.registerRenderer(new Cea708CaptionRenderer(context));
            controller.registerRenderer(new ClosedCaptionRenderer(context));
            mMediaPlayer.setSubtitleAnchor(controller, this);

            if (mAudioSession != 0) {
                mMediaPlayer.setAudioSessionId(mAudioSession);
            } else {
                mAudioSession = mMediaPlayer.getAudioSessionId();
            }
            mMediaPlayer.setOnPreparedListener(mPreparedListener);
            mMediaPlayer.setOnVideoSizeChangedListener(mSizeChangedListener);
            mMediaPlayer.setOnCompletionListener(mCompletionListener);
            mMediaPlayer.setOnErrorListener(mErrorListener);
            mMediaPlayer.setOnInfoListener(mInfoListener);
            mMediaPlayer.setOnBufferingUpdateListener(mBufferingUpdateListener);
            mCurrentBufferPercentage = 0;
            mMediaPlayer.setDataSource(mContext, mUri, mHeaders);
            mMediaPlayer.setDisplay(mSurfaceHolder);
            mMediaPlayer.setAudioAttributes(mAudioAttributes);
            mMediaPlayer.setScreenOnWhilePlaying(true);
            mMediaPlayer.prepareAsync();

            for (Pair<InputStream, MediaFormat> pending: mPendingSubtitleTracks) {
                try {
                    mMediaPlayer.addSubtitleSource(pending.first, pending.second);
                } catch (IllegalStateException e) {
                    mInfoListener.onInfo(
                            mMediaPlayer, MediaPlayer.MEDIA_INFO_UNSUPPORTED_SUBTITLE, 0);
                }
            }

            // we don't set the target state here either, but preserve the
            // target state that was there before.
            mCurrentState = STATE_PREPARING;
            attachMediaController();
        } catch (IOException ex) {
            Log.w(TAG, "Unable to open content: " + mUri, ex);
            mCurrentState = STATE_ERROR;
            mTargetState = STATE_ERROR;
            mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
            return;
        } catch (IllegalArgumentException ex) {
            Log.w(TAG, "Unable to open content: " + mUri, ex);
            mCurrentState = STATE_ERROR;
            mTargetState = STATE_ERROR;
            mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
            return;
        } finally {
            mPendingSubtitleTracks.clear();
        }
    }
