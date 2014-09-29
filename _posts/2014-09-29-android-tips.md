---
layout: post
title: "Android tips"
description: ""
category: 
tags: []
---
## 4.0开发笔记

### 1 Cookie
Webview中的Cookie是一个麻烦的东西，一直没找到一个系统的介绍。在FM中微博登录用到了cookie，在4.0上能登录，但是2.3上却不行
后来发现是下面这句话造成的
CookieManager.getInstance().getCookie(".douban.com");
换成下面这种方式就可以了
Uri uri = Uri.parst(url);
CookieManager.getInstance().getCookie(uri.getHost());

但是4.0和2.3上的表现又不一致，shouldOverrideUrlLoading在2.3上最后一次跳转不会进去；
在2.3上要比4.0上多调用一次onPageStart，而且在2.3上是在onPageStart中可以取到cookie，在4.0中
是在onPageEnd可以取到cookie，最后修改方式如下：

    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
    	super.onPageStarted(view, url, favicon);
    	NLog.v(TAG, "onPageStarted() url=" + url);
    	mLoadingView.setVisibility(View.VISIBLE);
    	checkCookie(url);
	}

	@Override
	public void onPageFinished(WebView view, String url) {
    	super.onPageFinished(view, url);
	    NLog.d(TAG, "onPageFinished url=" + url);
	    mLoadingView.setVisibility(View.GONE);
	    if (checkUrl(url)) {
	        checkCookie(url);
	        FMSinaAuth.this.loginSina(url);
	    }
	}

	private void checkCookie(String url) {
    try {
        Uri uri = Uri.parse(url);
        String host = uri.getHost();
        if (host.contains(".douban.com")) {
            String cookie = CookieManager.getInstance().getCookie(host);
            if (!TextUtils.isEmpty(cookie)) {
                mCookie = cookie;
            }
        }
        NLog.v(TAG, "checkCookie() host:" + host + ", hasCookie=" + CookieManager.getInstance().hasCookies() +
    	            ", mCookie= " + mCookie + ", cookie=" + CookieManager.getInstance().getCookie(".douban.com"));
    	} catch (Exception e) {
        	e.printStackTrace();
    	}
	}

### 2 res-auto
Added support for custom views with custom attributes in libraries.Layouts using custom attributes must use the namespace URI,
http://schemas.android.com/apk/res-auto instead of the URI that includes the app package name.
This URI is replaced with the app specific one at build time.
(http://stackoverflow.com/questions/10448006/xml-namespace-declaration-auto-substitute-package-name)


### 3 播放注意事项
在activity onCreate中设置setVolumeControlStream(AudioManager.STREAM_MUSIC);


### 4时间函数

    /*  Try to use String.format() as little as possible, because it creates a
     *  new Formatter every time you call it, which is very inefficient.
     *  Reusing an existing Formatter more than tripled the speed of
     *  makeTimeString().
     *  This Formatter/StringBuilder are also used by makeAlbumSongsLabel()
     */
    private static StringBuilder sFormatBuilder = new StringBuilder();
    private static Formatter sFormatter = new Formatter(sFormatBuilder, Locale.getDefault());
    private static final Object[] sTimeArgs = new Object[5];

    public static String makeTimeString(Context context, long secs) {
        String durationformat = context.getString(
                secs < 3600 ? R.string.durationformatshort : R.string.durationformatlong);

        /* Provide multiple arguments so the format can be changed easily
         * by modifying the xml.
         */
        sFormatBuilder.setLength(0);

        final Object[] timeArgs = sTimeArgs;
        timeArgs[0] = secs / 3600;
        timeArgs[1] = secs / 60;
        timeArgs[2] = (secs / 60) % 60;
        timeArgs[3] = secs;
        timeArgs[4] = secs % 60;

        return sFormatter.format(durationformat, timeArgs).toString();
    }
    
### 5 style
android中v7 Theme的style以@style引用；android framework中的以android：引用

	<style name="CustomActionBarStyle" parent="android:Widget.Holo.Light.ActionBar">
    	    <item name="android:background">@color/actionbar_background</item>
        	<item name="android:titleTextStyle">@style/CustomActionBarTitleTextStyle</item>
	</style>
	<style name="CustomActionBarStyle"
           parent="@style/Widget.AppCompat.Light.ActionBar.Solid.Inverse">
           <item name="android:background">@color/actionbar_background</item>
	</style>

### 6 animator

http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html

http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html

### 7 service
service中的bindService和unbindService要对应，不然会出现service leak。

典型的MediaPlayService为：

先startService（service状态为start）

bindService（操控service，获取service状态）

停止service：

所有client unbind并且调用stopSelf

unbindService并不会调用ServiceConnection中的onServiceDisconnected。

onServiceDisconnected is only called in extreme situations (crashed / killed).

bindService时需要调用startService否则unbindService时Service就会destroy

### 8 URI & URL
To encode a URI, invoke any of the multiple-parameter constructors of this class. These constructors accept your original strings and encode them into their raw form.

To decode a URI, invoke the single-string constructor, and then use the appropriate accessor methods to get the decoded components.

URL:A Uniform Resource Locator that identifies the location of an Internet resource as specified by RFC 1738.

onWindowsFocusChange (还是需要在addOnGlobalLayoutListener中判断）
在该函数调用时getWidth不会等于0

### 9 CircleProgressBarView 平滑移动处理
优化CircleProgressBarView，使之可以平滑移动，主要做了以下两点：

* 画thumbnail，通过将canvas translate到thumnail的中心点然后restore，防止thumbnail setBound出现误差；

* 通过在onDraw中设置invalidate实现平滑移动，PlayActivity中的handler只刷新text view的时间显示

### 10 Animation
http://www.satyakomatineni.com/akc/display?url=displaynoteimpurl&ownerUserId=satya&reportId=2898

### 11 Fragment
通过getChildFragmentManager实现nest fragments，并且通过在Fragment的onDestroyView时设置setAdapter(null)使得view pager中的Fragment也detach掉

### 12 ListView setItemChecked不起作用的问题
需要item实现了checkable接口
http://www.marvinlabs.com/2010/10/29/custom-listview-ability-check-items/

### 13 阻止Activity布局被softinputmethod影响需设置
android:windowSoftInputMode="stateUnspecified|adjustPan"

### 14 custom view
自定义View的事件响应

一般来说有Drag和Flinging，通过GestureDetector.OnGestureListener中的onScroll和onFling实现。在该Listener中onDown需要返回true，否则后续的消
息就不会进到OnGestureListener中。

ViewDragHelper用作ViewGroup中的事件处理。

Scroll和OverScroll是处理滑动很重要的函数。OverScroll相比较Scroll多了对边界的判断。
通过Scroll和OverScroll计算运动的路径。

得到位置信息后就要把它画出来了，draw有两种方法：

* 在得到位置后调用postInvalidate，在onDraw中计算scroll offset

* 通过ValueAnimator的addUpdateListener函数，在AnimatorUpdateListener中实时改变object的属性

两种方式第二种更好，减少了不必要的invalidate，并且也更平滑，只是ValueAnimator是在API 11中才加进来的。

LyricView实现原理

LyricView从TextView继承，通过setScroller将Scroller传入系统，在draw时会调用computeScroll，所以在SimpleGestrueListener中 onScroll只需调用scrollTo即可。

onFling中由于该Scroller变化对应的时TextView中的scrollY，所以起始值需要传getScrollY

### 15 在有实体menu按键的手机上显示menu icon
	private void forceShowActionBarOverflowMenu() {
        try {
            ViewConfiguration config = ViewConfiguration.get(this);
            Field menuKeyField = ViewConfiguration.class.getDeclaredField("sHasPermanentMenuKey");
            if (menuKeyField != null) {
                menuKeyField.setAccessible(true);
                menuKeyField.setBoolean(config, false);
            }
        } catch (Exception ignored) {

        }
    }

### 16 mac java multiple versions

http://apple.stackexchange.com/questions/57986/multiple-java-versions-support-on-os-x-and-java-home-location

### 17 清除singlechoice mode的选中状态

	protected void clearChecked() {
        listView.setChoiceMode(AbsListView.CHOICE_MODE_NONE);
        listView.setAdapter(adapter);
        listView.post(new Runnable() {
            @Override
            public void run() {
                listView.setChoiceMode(AbsListView.CHOICE_MODE_SINGLE);
                listView.setAdapter(adapter);
            }
        });
    }

### 18 list view item has ImageButton

Remove the focusable attribute from the Button would solve this problem. You could do that either in a layout xml file or java source code.

And one more tip, if you are using ImageButton instead of Button,you need setFocusable in your java code to make that work,

because the constructor of ImageButton would enabe this attribute after inflate from xml file.

### 19 MediaPlayer状态控制

 prepare也会调用onPrepare回调
 
 prepareAsync后进入Preparing状态，其是中间状态，在Preparing调用任何函数的效果都是undefined
 
 pause底层engine是异步的，所以立即调用isPlaying可能返回的还是true

### 20 可以使用menu orderCategory调整menu item顺序

### 21 修改ActionBar右上角icon

http://stackoverflow.com/questions/22046903/changing-the-android-overflow-menu-icon-programmatically/22106474#22106474

### 22 没有root的情况下如何adb pull /data/data/package/下的数据

	#!/usr/bin/env bash
	PACKAGE_NAME=com.your.package
	DB_NAME=data.db
	rm -rf ${DB_NAME}
	adb shell "run-as ${PACKAGE_NAME} chmod 666 /data/data/${PACKAGE_NAME}/databases/${DB_NAME}"
	adb pull /data/data/${PACKAGE_NAME}/databases/${DB_NAME} /tmp/
	adb shell "run-as ${PACKAGE_NAME} chmod 600 /data/data/${PACKAGE_NAME}/databases/${DB_NAME}"
	sqlite3 /tmp/${DB_NAME

### 23 pathPattern

	<data android:pathPattern=".*\\.mytype"/>
	<data android:pathPattern=".*\\..*\\.mytype"/>
	<data android:pathPattern=".*\\..*\\..*\\.mytype"/>
	<data android:pathPattern=".*\\..*\\..*\\..*\\.mytype"/>


### 24 notification重装后不起作用
报Permission Denial: Accessing service ComponentInfo{com.douban.radio/com.douban.radio.mediaplayer.MediaPlaybackService} from pid=-1, uid=10168 that is not exported from uid 10169

出现在应用重装后，widget／notification不响应的问题

https://code.google.com/p/android/issues/detail?id=61850

修改PendingIntent.getService将PendingIntent.FLAG_UPDATE_CURRENT换成PendingIntent.FLAG_CANCEL_CURRENT
{% include JB/setup %}
