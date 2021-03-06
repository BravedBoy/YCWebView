#### 问题汇总目录介绍
- 4.0.0 WebView进化史介绍
- 4.0.1 提前初始化WebView必要性
- 4.0.2 x5加载office资源
- 4.0.3 WebView播放视频问题
- 4.0.4 无法获取webView的正确高度
- 4.0.5 使用scheme协议打开链接风险
- 4.0.6 如何处理加载错误
- 4.0.7 webView防止内存泄漏
- 4.0.8 关于js注入时机修改
- 4.0.9 视频/图片宽度超过屏幕
- 4.1.0 如何保证js安全性
- 4.1.1 如何代码开启硬件加速
- 4.1.2 WebView设置Cookie
- 4.1.3 开启硬件加速导致的闪烁问题
- 4.1.4 webView加载网页不显示图片
- 4.1.5 绕过证书校验漏洞
- 4.1.6 allowFileAccess漏洞
- 4.1.7 WebView嵌套ScrollView问题
- 4.1.8 WebView中图片点击放大
- 4.1.9 页面滑动期间不渲染/执行
- 4.2.0 被运营商劫持和注入问题
- 4.2.1 解决资源加载缓慢问题
- 4.2.2 判断是否已经滚动到页面底端
- 4.2.3 使用loadData加载html乱码
- 4.2.4 WebView下载进度无法监听
- 4.2.5 webView出现302/303重定向
- 4.2.6 webView出现302/303白屏
- 4.2.8 onReceiveError问题
- 4.2.9 loadUrl在19以上超过2097152个字符失效
- 4.3.0 WebViewJavascriptBridge: WARNING
- 4.3.1 Android与js传递数据大小有限制
- 4.3.2 多次调用callHandler部分回调函数未被调用
- 4.3.3 字符串转义bug探讨
- 4.3.8 Javascript调用原生方法会偶现失败
- 4.3.9 dispatchMessage运行主线程问题
- 4.4.0 怎么实现WebView免流方案
- 4.4.1 Channel is unrecoverably broken and will be disposed!
- 4.4.2 定制js的alert,confirm和prompt对话框
- 4.4.3 x5长按图片如何操作
- 4.4 4 x5长按文字内容如何自定义弹窗
- 4.4.5 webView.goBack()会刷新页面吗
- 4.4.6 mWebView.scrollTo(0, 0)回顶部失效
- 4.4.7 部分手机监听滑动顶部或底部失效
- 4.4.8 prompt的一个坑导致js挂掉
- 4.4.9 webView背景设置为透明无效探索
- 4.5.0 如何屏蔽掉WebView中长按事件
- 4.5.1 WeView出现OOM影响主进程如何避免
- 4.5.2 WebView域控制不严格漏洞
- 4.5.3 下载文件时的路径穿越问题
- 4.5.4 WebView中http和https混合使用问题
- 4.5.5 调用系统EMAIL发送邮件崩溃
- 4.5.7 WebView访问部分网页崩溃问题


### 4.0.0 WebView进化史介绍
- 进化史如下所示
    - 从Android4.4系统开始，Chromium内核取代了Webkit内核。
    - 从Android5.0系统开始，WebView移植成了一个独立的apk，可以不依赖系统而独立存在和更新。
    - 从Android7.0 系统开始，如果用户手机里安装了 Chrome ， 系统优先选择 Chrome 为应用提供 WebView 渲染。
    - 从Android8.0系统开始，默认开启WebView多进程模式，即WebView运行在独立的沙盒进程中。


### 4.0.1 提前初始化WebView必要性
- 第一次打开Web面 ，使用WebView加载页面的时候特别慢，第二次打开就能明显的感觉到速度有提升，为什么？
    - 是因为在你第一次加载页面的时候 WebView 内核并没有初始化 ，所以在第一次加载页面的时候需要耗时去初始化WebView内核 。
    - 提前初始化WebView内核 ，例如如下把它放到了Application里面去初始化 , 在页面里可以直接使用该WebView，这种方法可以比较有效的减少WebView在App中的首次打开时间。当用户访问页面时，不需要初始化WebView的时间。
    - 但是这样也有不好的地方，额外的内存消耗。页面间跳转需要清空上一个页面的痕迹，更容易内存泄露。


### 4.0.2 x5加载office资源
- 关于加载word，pdf，xls等文档文件注意事项：Tbs不支持加载网络的文件，需要先把文件下载到本地，然后再加载出来
- 还有一点要注意，在onDestroy方法中调用此方法mTbsReaderView.onStop()，否则第二次打开无法浏览。更多可以看FileReaderView类代码！



### 4.0.3 WebView播放视频问题
- 1、此次的方案用到WebView，而且其中会有视频嵌套，在默认的WebView中直接播放视频会有问题， 而且不同的SDK版本情况还不一样，网上搜索了下解决方案，在此记录下. webView.getSettings.setPluginState(PluginState.ON);webView.setWebChromeClient(new WebChromeClient());
- 2、然后在webView的Activity配置里面加上： android:hardwareAccelerated="true"
- 3、以上可以正常播放视频了，但是webview的页面都finish了居然还能听 到视频播放的声音， 于是又查了下发现webview的onResume方法可以继续播放，onPause可以暂停播放， 但是这两个方法都是在Added in API level 11添加的，所以需要用反射来完成。
- 4、停止播放：在页面的onPause方法中使用：webView.getClass().getMethod("onPause").invoke(webView, (Object[])null);
- 5、继续播放：在页面的onResume方法中使用：webView.getClass().getMethod("onResume").invoke(webView,(Object[])null);这样就可以控制视频的暂停和继续播放了。


### 4.0.4 无法获取webView的正确高度
- 偶发情况，获取不到webView的内容高度
    - 其中htmlString是一个HTML格式的字符串。
    ```
    webView.loadData(htmlString, "text/html", "utf-8");
    webView.setWebViewClient(new WebViewClient() {
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            Log.d("yc", view.getContentheight() + "");
        }
    });
    ```
    - 这是因为onPageFinished回调指的WebView已经完成从网络读取的字节数，这一点。在点onPageFinished被激发的页面可能还没有被解析。
- 第一种解决办法：提供onPageFinished（）一些延迟
    ```
    webView.setWebViewClient(new WebViewClient() {
        @Override
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            webView.postDelayed(new Runnable() {
                @Override
                public void run() {
                    int contentHeight = webView.getContentHeight();
                    int viewHeight = webView.getHeight();
                }
            }, 500);
        }
    });
    ```
- 第二种解决办法：使用js获取内容高度，具体可以看这篇文章：https://www.jianshu.com/p/ad22b2649fba



### 4.0.5 使用scheme协议打开链接风险
- 常见的用法是在APP获取到来自网页的数据后，重新生成一个intent，然后发送给别的组件使用这些数据。比如使用Webview相关的Activity来加载一个来自网页的url，如果此url来自url scheme中的参数，如：yc://ycbjie:8888/from?load_url=http://www.taobao.com。
    - 如果在APP中，没有检查获取到的load_url的值，攻击者可以构造钓鱼网站，诱导用户点击加载，就可以盗取用户信息。
    - 这个时候，别人非法篡改参数，于是将scheme协议改成yc://ycbjie:8888/from?load_url=http://www.doubi.com。这个时候点击进去即可进入钓鱼链接地址。
- 使用建议
    - APP中任何接收外部输入数据的地方都是潜在的攻击点，过滤检查来自网页的参数。
    - 不要通过网页传输敏感信息，有的网站为了引导已经登录的用户到APP上使用，会使用脚本动态的生成URL Scheme的参数，其中包括了用户名、密码或者登录态token等敏感信息，让用户打开APP直接就登录了。恶意应用也可以注册相同的URL Sechme来截取这些敏感信息。Android系统会让用户选择使用哪个应用打开链接，但是如果用户不注意，就会使用恶意应用打开，导致敏感信息泄露或者其他风险。
- 解决办法
    - 在内嵌的WebView中应该限制允许打开的WebView的域名，并设置运行访问的白名单。或者当用户打开外部链接前给用户强烈而明显的提示。具体操作可以看5.0.8 如何设置白名单操作方式。


### 4.0.6 如何处理加载错误(Http、SSL、Resource)
- 对于WebView加载一个网页过程中所产生的错误回调，大致有三种
    ```
    /**
     * 只有在主页面加载出现错误时，才会回调这个方法。这正是展示加载错误页面最合适的方法。
     * 然而，如果不管三七二十一直接展示错误页面的话，那很有可能会误判，给用户造成经常加载页面失败的错觉。
     * 由于不同的WebView实现可能不一样，所以我们首先需要排除几种误判的例子：
     *      1.加载失败的url跟WebView里的url不是同一个url，排除；
     *      2.errorCode=-1，表明是ERROR_UNKNOWN的错误，为了保证不误判，排除
     *      3failingUrl=null&errorCode=-12，由于错误的url是空而不是ERROR_BAD_URL，排除
     * @param webView                                           webView
     * @param errorCode                                         errorCode
     * @param description                                       description
     * @param failingUrl                                        failingUrl
     */
    @Override
    public void onReceivedError(WebView webView, int errorCode,
                                String description, String failingUrl) {
        super.onReceivedError(webView, errorCode, description, failingUrl);
        // -12 == EventHandle.ERROR_BAD_URL, a hide return code inside android.net.http package
        if ((failingUrl != null && !failingUrl.equals(webView.getUrl())
                && !failingUrl.equals(webView.getOriginalUrl())) /* not subresource error*/
                || (failingUrl == null && errorCode != -12) /*not bad url*/
                || errorCode == -1) { //当 errorCode = -1 且错误信息为 net::ERR_CACHE_MISS
            return;
        }
        if (!TextUtils.isEmpty(failingUrl)) {
            if (failingUrl.equals(webView.getUrl())) {
                //做自己的错误操作，比如自定义错误页面
            }
        }
    }

    /**
     * 只有在主页面加载出现错误时，才会回调这个方法。这正是展示加载错误页面最合适的方法。
     * 然而，如果不管三七二十一直接展示错误页面的话，那很有可能会误判，给用户造成经常加载页面失败的错觉。
     * 由于不同的WebView实现可能不一样，所以我们首先需要排除几种误判的例子：
     *      1.加载失败的url跟WebView里的url不是同一个url，排除；
     *      2.errorCode=-1，表明是ERROR_UNKNOWN的错误，为了保证不误判，排除
     *      3failingUrl=null&errorCode=-12，由于错误的url是空而不是ERROR_BAD_URL，排除
     * @param webView                                           webView
     * @param webResourceRequest                                webResourceRequest
     * @param webResourceError                                  webResourceError
     */
    @Override
    public void onReceivedError(WebView webView, WebResourceRequest webResourceRequest,
                                WebResourceError webResourceError) {
        super.onReceivedError(webView, webResourceRequest, webResourceError);
    }

    /**
     * 任何HTTP请求产生的错误都会回调这个方法，包括主页面的html文档请求，iframe、图片等资源请求。
     * 在这个回调中，由于混杂了很多请求，不适合用来展示加载错误的页面，而适合做监控报警。
     * 当某个URL，或者某个资源收到大量报警时，说明页面或资源可能存在问题，这时候可以让相关运营及时响应修改。
     * @param webView                                           webView
     * @param webResourceRequest                                webResourceRequest
     * @param webResourceResponse                               webResourceResponse
     */
    @Override
    public void onReceivedHttpError(WebView webView, WebResourceRequest webResourceRequest,
                                    WebResourceResponse webResourceResponse) {
        super.onReceivedHttpError(webView, webResourceRequest, webResourceResponse);
    }

    /**
     * 任何HTTPS请求，遇到SSL错误时都会回调这个方法。
     * 比较正确的做法是让用户选择是否信任这个网站，这时候可以弹出信任选择框供用户选择（大部分正规浏览器是这么做的）。
     * 有时候，针对自己的网站，可以让一些特定的网站，不管其证书是否存在问题，都让用户信任它。
     * 坑：有时候部分手机打开页面报错，绝招：让自己网站的所有二级域都是可信任的。
     * @param webView                                           webView
     * @param sslErrorHandler                                   sslErrorHandler
     * @param sslError                                          sslError
     */
    @Override
    public void onReceivedSslError(WebView webView, SslErrorHandler sslErrorHandler, SslError sslError) {
        super.onReceivedSslError(webView, sslErrorHandler, sslError);
        //判断网站是否是可信任的，与自己网站host作比较
        if (WebViewUtils.isYCHost(webView.getUrl())) {
            //如果是自己的网站，则继续使用SSL证书
            sslErrorHandler.proceed();
        } else {
            super.onReceivedSslError(webView, sslErrorHandler, sslError);
        }
    }
    ```



### 4.0.7 webView防止内存泄漏
- https://my.oschina.net/zhibuji/blog/100580




### 4.0.9 视频/图片宽度超过屏幕
- 视频播放宽度或者图片宽度比webView设置的宽度大，超过屏幕：这个时候可以设置ws.setLoadWithOverviewMode(false);
- 另外一种让图片不超出屏幕范围的方法，可以用的是css
    ```
    <script type="text/javascript">
       var tables = document.getElementsByTagName("img");  //找到table标签
         for(var i = 0; i<tables.length; i++){  // 逐个改变
                tables[i].style.width = "100%";  // 宽度改为100%
                 tables[i].style.height = "auto";
         }
    </script>
    ```
- 通过webView的setting属性设置
    ```
    // 网页内容的宽度是否可大于WebView控件的宽度
    ws.setLoadWithOverviewMode(false);
    ```


### 4.1.0 如何保证js安全性
- Android和js如何通信
    - 为了与Web页面实现动态交互，Android应用程序允许WebView通过WebView.addJavascriptInterface接口向Web页面注入Java对象，页面Javascript脚本可直接引用该对象并调用该对象的方法。
    - 这类应用程序一般都会有类似如下的代码：
        ```
        webView.addJavascriptInterface(javaObj, "jsObj");
        ```
    - 此段代码将javaObj对象暴露给js脚本，可以通过jsObj对象对其进行引用，调用javaObj的方法。结合Java的反射机制可以通过js脚本执行任意Java代码，相关代码如下：
        - 当受影响的应用程序执行到上述脚本的时候，就会执行someCmd指定的命令。
        ```
        <script>
        　　function execute(cmdArgs) {
            　　return jsobj.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec(cmdArgs);
        　　}
        
        　　execute(someCmd);
        </script>
        ```
- addJavascriptInterface任何命令执行漏洞
    - 在webView中使用js与html进行交互是一个不错的方式，但是，在Android4.2(16，包含4.2)及以下版本中，如果使用addJavascriptInterface，则会存在被注入js接口的漏洞；在4.2之后，由于Google增加了@JavascriptInterface，该漏洞得以解决。
- @JavascriptInterface注解做了什么操作
    - 之前，任何Public的函数都可以在JS代码中访问，而Java对象继承关系会导致很多Public的函数都可以在JS中访问，其中一个重要的函数就是getClass()。然后JS可以通过反射来访问其他一些内容。通过引入 @JavascriptInterface注解，则在JS中只能访问 @JavascriptInterface注解的函数。这样就可以增强安全性。
- 在4.2之前，存在漏洞，解决方案如下所示。移除android 4.2之前的默认接口
    ```
    removeJavascriptInterface(“searchBoxJavaBridge_”)
    removeJavascriptInterface(“accessibility”)
    removeJavascriptInterface(“accessibilityTraversal”)
    ```



### 4.1.1 如何代码开启硬件加速
- 开启软硬件加速这个性能提升还是很明显的，但是会耗费更大的内存 。直接调用代码api即可完成，webView.setOpenLayerType(true);


### 4.1.2 WebView设置Cookie
- h5页面为何要设置cookie，主要是避免网页重复登录，作用是记录用户登录信息，下次进去不需要重复登录。
- 代码里怎么设置Cookie，如下所示
    ```
    /**
     * 同步cookie
     *
     * @param url               地址
     * @param cookieList        需要添加的Cookie值,以键值对的方式:key=value
     */
    private void syncCookie (Context context , String url, ArrayList<String> cookieList) {
        //初始化
        CookieSyncManager.createInstance(context);
        //获取对象
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.setAcceptCookie(true);
        //移除
        cookieManager.removeSessionCookie();
        //添加
        if (cookieList != null && cookieList.size() > 0) {
            for (String cookie : cookieList) {
                cookieManager.setCookie(url, cookie);
            }
        }
        String cookies = cookieManager.getCookie(url);
        X5LogUtils.d("cookies-------"+cookies);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            cookieManager.flush();
        } else {
            CookieSyncManager.getInstance().sync();
        }
    }
    ```
- 在android里面在调用webView.loadUrl(url)之前一句调用此方法就可以给WebView设置Cookie 
    - 注:这里一定要注意一点，在调用设置Cookie之后不能再设置，否则设置Cookie无效。该处需要校验，为何？？？
    ```
    webView.getSettings().setBuiltInZoomControls(true);  
    webView.getSettings().setJavaScriptEnabled(true);  
    ```
- 还有跨域问题： 域A: test1.yc.com 域B: test2.yc.com
    - 那么在域A生产一个可以使域A和域B都能访问的Cookie就需要将Cookie的domain设置为.yc.com；
    - 如果要在域A生产一个令域A不能访问而域能访问的Cookie就要将Cookie设置为test2.yc.com。
- Cookie的过期机制
    - 可以设置Cookie的生效时间字段名为： expires 或 max-age。
        - expires：过期的时间点
        - max-age：生效的持续时间，单位为秒。
    - 若将Cookie的 max-age 设置为负数，或者 expires 字段设置为过期时间点，数据库更新后这条Cookie将从数据库中被删除。如果将Cookie的 max-age 和 expires 字段设置为正常的过期日期，则到期后再数据库更新时会删除该条数据。
- 下面列出几个有用的接口：
    - 获取某个url下的所有Cookie：CookieManager.getInstance().getCookie(url)
    - 判断WebView是否接受Cookie：CookieManager.getInstance().acceptCookie()
    - 清除Session Cookie：CookieManager.getInstance().removeSessionCookies(ValueCallback<Boolean> callback)
    - 清除所有Cookie：CookieManager.getInstance().removeAllCookies(ValueCallback<Boolean> callback)
    - Cookie持久化：CookieManager.getInstance().flush()
    - 针对某个主机设置Cookie：CookieManager.getInstance().setCookie(String url, String value)



### 4.1.3 开启硬件加速导致的闪烁问题
- 在应用开启硬件加速后，WebView可能在加载过程出现闪烁现象。
- 解决方案：为WebView关闭硬件加速功能。webView.setLayerType(View.LAYER_TYPE_SOFTWARE,null);




### 4.1.4 webView加载网页不显示图片
- webView从Lollipop(5.0)开始webView默认不允许混合模式, https当中不能加载http资源, 而开发的时候可能使用的是https的链接, 但是链接中的图片可能是http的, 所以需要设置开启。
    ```
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.LOLLIPOP) {
            mWebView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
    mWebView.getSettings().setBlockNetworkImage(false);
    ```



### 4.1.5 绕过证书校验漏洞
- webViewClient中有onReceivedError方法，当出现证书校验错误时，我们可以在该方法中使用handler.proceed()来忽略证书校验继续加载网页，或者使用默认的handler.cancel()来终端加载。
    - 因为我们使用了handler.proceed()，由此产生了该“绕过证书校验漏洞”。如果确定所有页面都能满足证书校验，则不必要使用handler.proceed()
    ```
    @SuppressLint("NewApi")
    @Override
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        //handler.proceed();// 接受证书
        super.onReceivedSslError(view, handler, error);
    }
    ```



### 4.1.6 allowFileAccess漏洞
- 如果webView.getSettings().setAllowFileAccess(boolean)设置为true，则会面临该问题；该漏洞是通过WebView对Javascript的延时执行和html文件替换产生的。
    - 解决方案是禁止WebView页面打开本地文件，即：webView.getSettings().setAllowFileAccess(false);
    - 或者更直接的禁止使用JavaScript：webView.getSettings().setJavaScriptEnabled(false);



### 4.1.7 WebView嵌套ScrollView问题
- 问题描述
    - 当 WebView 嵌套在 ScrollView 里面的时候，如果 WebView 先加载了一个高度很高的网页，然后加载了一个高度很低的网页，就会造成 WebView 的高度无法自适应，底部出现大量空白的情况出现。
- 解决办法
    - 可以参考这篇博客：https://blog.csdn.net/self_study/article/details/54378978


### 4.1.8 WebView中图片点击放大
- 首先载入js
    ```
    //将js对象与java对象进行映射
    webView.addJavascriptInterface(new ImageJavascriptInterface(context), "imagelistener");
    ```
- html加载完成之后，添加监听图片的点击js函数，这个可以在onPageFinished方法中操作
    ```
    @Override
    public void onPageFinished(WebView view, String url) {
        X5LogUtils.i("-------onPageFinished-------"+url);
        //html加载完成之后，添加监听图片的点击js函数
        //addImageClickListener();
        addImageArrayClickListener(webView);
    }
    ```
- 具体看addImageArrayClickListener的实现方法。
    ```
    /**
     * android与js交互：
     * 首先我们拿到html中加载图片的标签img.
     * 然后取出其对应的src属性
     * 循环遍历设置图片的点击事件
     * 将src作为参数传给java代码
     * 这个循环将所图片放入数组，当js调用本地方法时传入。
     * 当然如果采用方式一获取图片的话，本地方法可以不需要传入这个数组
     * 通过js代码找到标签为img的代码块，设置点击的监听方法与本地的openImage方法进行连接
     * @param webView                       webview
     */
    private void addImageArrayClickListener(WebView webView) {
        webView.loadUrl("javascript:(function(){" +
                "var objs = document.getElementsByTagName(\"img\"); " +
                "var array=new Array(); " +
                "for(var j=0;j<objs.length;j++){" +
                "    array[j]=objs[j].src; " +
                "}"+
                "for(var i=0;i<objs.length;i++)  " +
                "{"
                + "    objs[i].onclick=function()  " +
                "    {  "
                + "        window.imagelistener.openImage(this.src,array);  " +
                "    }  " +
                "}" +
                "})()");
    }
    ```
- 最后看看js的通信接口做了什么
    ```
    public class ImageJavascriptInterface {
    
        private Context context;
        private String[] imageUrls;
    
        public ImageJavascriptInterface(Context context,String[] imageUrls) {
            this.context = context;
            this.imageUrls = imageUrls;
        }
    
        public ImageJavascriptInterface(Context context) {
            this.context = context;
        }
    
        /**
         * 接口返回的方式
         */
        @android.webkit.JavascriptInterface
        public void openImage(String img , String[] imageUrls) {
            Intent intent = new Intent();
            intent.putExtra("imageUrls", imageUrls);
            intent.putExtra("curImageUrl", img);
    //        intent.setClass(context, PhotoBrowserActivity.class);
            context.startActivity(intent);
            for (int i = 0; i < imageUrls.length; i++) {
                Log.e("图片地址"+i,imageUrls[i].toString());
            }
        }
    }
    ```


### 4.1.9 页面滑动期间不渲染/执行
- 在有些需求中会有一些吸顶的元素，例如导航条，购买按钮等；当页面滚动超出元素高度后，元素吸附在屏幕顶部。在WebView中成了难题：在页面滚动期间，Scroll Event不触发。不仅如此，WebView在滚动期间还有各种限定：
    - setTimeout和setInterval不触发。
    - GIF动画不播放。
    - 很多回调会延迟到页面停止滚动之后。
    - background-position: fixed不支持。
- 这些限制让WebView在滚动期间很难有较好的体验。这些限制大部分是不可突破的，但至少对于吸顶功能还是可以做一些支持，解决方法：
    - 在Android上，监听touchMove事件可以在滑动期间做元素的position切换（惯性运动期间就无效了）。
- 参考美团技术文章



### 4.2.0 被运营商劫持和注入问题
- 由于WebView加载的页面代码是从服务器动态获取的，这些代码将会很容易被中间环节所窃取或者修改，其中最主要的问题出自地方运营商和一些WiFi。监测到的问题包括：
    - 无视通信规则强制缓存页面。
    - header被篡改。
    - 页面被注入广告。
    - 页面被重定向。
    - 页面被重定向并重新iframe到新页面，框架嵌入广告。
    - HTTPS请求被拦截。
    - DNS劫持。
- 针对页面注入的行为，有一些解决方案：
    - 1.使用CSP（Content Security Policy）
    - 2.HTTPS。
        - HTTPS可以防止页面被劫持或者注入，然而其副作用也是明显的，网络传输的性能和成功率都会下降，而且HTTPS的页面会要求页面内所有引用的资源也是HTTPS的，对于大型网站其迁移成本并不算低。HTTPS的一个问题在于：一旦底层想要篡改或者劫持，会导致整个链接失效，页面无法展示。这会带来一个问题：本来页面只是会被注入广告，而且广告会被CSP拦截，而采用了HTTPS后，整个网页由于受到劫持完全无法展示。
        - 对于安全要求不高的静态页面，就需要权衡HTTPS带来的利与弊了。
    - 3.App使用Socket代理请求
        - 如果HTTP请求容易被拦截，那么让App将其转换为一个Socket请求，并代理WebView的访问也是一个办法。
        - 通常不法运营商或者WiFi都只能拦截HTTP（S）请求，对于自定义的包内容则无法拦截，因此可以基本解决注入和劫持的问题。
        - Socket代理请求也存在问题：
        - 首先，使用客户端代理的页面HTML请求将丧失边下载边解析的能力；根据前面所述，浏览器在HTML收到部分内容后就立刻开始解析，并加载解析出来的外链、图片等，执行内联的脚本……而目前WebView对外并没有暴露这种流式的HTML接口，只能由客户端完全下载好HTML后，注入到WebView中。因此其性能将会受到影响。
        - 其次，其技术问题也是较多的，例如对跳转的处理，对缓存的处理，对CDN的处理等等……稍不留神就会埋下若干大坑。
        - 此外还有一些其他的办法，例如页面的MD5检测，页面静态页打包下载等等方式，具体如何选择还要根据具体的场景抉择。



### 4.2.1 解决资源加载缓慢问题
- 在资源预加载方面，其实也有很多种方式，下面主要列举了一些：
    - 第一种方式是使用 WebView 自身的缓存机制：如果我们在 APP 里面访问一个页面，短时间内再次访问这个页面的时候，就会感觉到第二次打开的时候顺畅很多，加载速度比第一次的时间要短，这个就是因为 WebView 自身内部会做一些缓存，只要打开过的资源，他都会试着缓存到本地，第二次需要访问的时候他直接从本地读取，但是这个读取其实是不太稳定的东西，关掉之后，或者说这种缓存失效之后，系统会自动把它清除，我们没办法进行控制。基于这个 WebView 自身的缓存，有一种资源预加载的方案就是，我们在应用启动的时候可以开一个像素的 WebView ，事先去访问一下我们常用的资源，后续打开页面的时候如果再用到这些资源他就可以从本地获取到，页面加载的时间会短一些。
    - 第二种方案是，自己去构建和管理缓存：把这些需要预加载的资源放在 APP 里面，可能是预先放进去的，也可能是后续下载的，问题在于前端这些页面怎么去缓存，两个方案，第一种是前端可以在 H5 打包的时候把里面的资源 URL 进行替换，这样可以直接访问本地的地址；第二种是客户端可以拦截这些网页发出的所有请求做替换。
    - 具体可以看美团的技术文章：[美团大众点评 Hybrid 化建设](https://mp.weixin.qq.com/s/rNGD6SotKoO8frmxIU8-xw?)



### 4.2.2 判断是否已经滚动到页面底端
- getScrollY()方法返回的是当前可见区域的顶端距整个页面顶端的距离,也就是当前内容滚动的距离.
- getHeight()或者getBottom()方法都返回当前WebView 这个容器的高度
- getContentHeight 返回的是整个html的高度,但并不等同于当前整个页面的高度,因为WebView有缩放功能，所以当前整个页面的高度实际上应该是原始html 的高度再乘上缩放比例. 因此,更正后的结果,准确的判断方法应该是：
    ```
    if(WebView.getContentHeight*WebView.getScale() == (webview.getHeight()+WebView.getScrollY())){
        //已经处于底端
    }
    ```


### 4.2.3 使用loadData加载html乱码
- 可以通过使用 WebView.loadData(String data, String mimeType, String encoding)) 方法来加载一整个 HTML 页面的一小段内容，第一个就是我们需要 WebView 展示的内容，第二个是我们告诉 WebView 我们展示内容的类型，一般，第三个是字节码，但是使用的时候，这里会有一些坑
    - 明明已经指定了编码格式为 UTF-8，加载却还会出现乱码……
    ```
    String html = new String("<h3>我是loadData() 的标题</h3><p>&nbsp&nbsp我是他的内容</p>");
    webView.loadData(html, "text/html", "UTF-8");
    ```
- 使用loadData()或 loadDataWithBaseURL()加载一段HTML代码片段
    - data:是要加载的数据类型，但在数据里面不能出现英文字符：'#', '%', '\' , '?' 这四个字符，如果有的话可以用 %23, %25, %27, %3f，这些字符来替换，在平时测试时，你的数据时，你的数据里含有这些字符，但不会出问题，当出问题时，你可以替换下。
        * %，会报找不到页面错误，页面全是乱码。乱码样式见符件。
        * #，会让你的goBack失效，但canGoBAck是可以使用的。于是就会产生返回按钮生效，但不能返回的情况。
        * \ 和? 我在转换时，会报错，因为它会把\当作转义符来使用，如果用两级转义，也不生效，我是对它无语了。
    - 我们在使用loadData时，就意味着需要把所有的非法字符全部转换掉，这样就会给运行速度带来很大的影响，因为在使用时，在页面stytle中会使用很多%号。页面的数据越多，运行的速度就会越慢。
    - data中，有人会遇到中文乱码问题，解决办法：参数传"utf-8"，页面的编码格式也必须是utf-8，这样编码统一就不会乱了。别的编码我也没有试过。
- 解决办法
    - 
    ```
    String html = new String("<h3>我是loadData() 的标题</h3><p>&nbsp&nbsp我是他的内容</p>");
    webView.loadData(html, "text/html;charset=UTF-8", "null");
    ```


### 4.2.4 WebView下载进度无法监听
- https://www.jianshu.com/p/6e38e1ef203a



### 4.2.5 webView出现302/303重定向
- 专业叙述
    - 302重定向又称之为302代表暂时性转移
- 网络解释
    - 重定向是网页制作中的一个知识，几个例子跟你说明，假设你现在所处的位置是一个论坛的登录页面，你填写了帐号，密码，点击登陆，如果你的帐号密码正确，就自动跳转到论坛的首页，不正确就返回登录页；这里的自动跳转，就是重定向的意思。或者可以说，重定向就是，在网页上设置一个约束条件，条件满足，就自动转入到其它网页、网址 。比如，你输入一个网站链接，一般可以直接进入网站，如果出现错误，则又跳转到另外一个网页。
- 举个例子
    - 叙述下这种问题的情况，就是WebView首先加载A链接，然后在WebView上点击一个B链接进行加载，B链接会自动跳转到C链接，这个时候调用WebView的goback方法，会返回到加载B链接，但是B链接又会跳转到C链接，从而导致没法返回到A链接界面（当然也有朋友说快速的按两次返回键－也就是连续触发了两次goback可以返回到A链接，但并不是所有用户都懂这个，而且操作上也很恶心。），这就是重定向问题。
- 实现WebView的滑动监听和优雅处理回退栈问题
    - WebView能否知道某个url是不是301/302呢？当然知道，WebView能够拿到url的请求信息和响应信息，根据header里的code很轻松就可以实现，事实正是如此，交给WebView来处理重定向(return false)，这时候按返回键，是可以正常地回到重定向之前的那个页面的。（PS：从上面的章节可知，WebView在5.0以后是一个独立的apk，可以单独升级，新版本的WebView实现肯定处理了重定向问题）
    - 但是，业务对url拦截有需求，肯定不能把所有的情况都交给系统WebView处理。为了解决url拦截问题，本文引入了另一种思想——通过用户的touch事件来判断重定向。具体可以看项目lib中的ScrollWebView！


### 4.2.6 webView出现302/303白屏
- 


### 4.2.8 onReceiveError问题
- https://www.jianshu.com/p/fcebd23cbebb



### 4.2.9 loadUrl在19以上超过2097152个字符失效
- 修改代码如下所示
    ```
    /**
    * loadUrl方法在19以上超过2097152个字符失效
    */
    private static final int URL_MAX_CHARACTER_NUM=2097152;
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT &&
            javascriptCommand.length()>=URL_MAX_CHARACTER_NUM) {
        this.evaluateJavascript(javascriptCommand,null);
    }else {
        this.loadUrl(javascriptCommand);
    }
    ```


### 4.3.0 WebViewJavascriptBridge: WARNING
- 有时候会出现这种报错日志：WebViewJavascriptBridge: WARNING: javascript handler threw.", source: (1) 
    - 发生这种错误的原因。在作者库的library下的assets中有个js文件，错误是在那里面出现的。
    - 这里总结如下：两边互传数据最好做的是严格意义上的json字符串。最后一点，也是最重要的一点。
        ```
        try {
            handler(message.data, responseCallback);
        } catch (exception) {
            if (typeof console != 'undefined') {
                console.log("WebViewJavascriptBridge: WARNING: javascript handler threw.", message, exception);
            }
        }
        ```
    - 上面的错误，是发生在WebViewJavascriptBridge.js的这里。。这里try catch发生错误。handler是js端给到的处理函数，也就是在js端的这个处理函数里发生任何错误，都会出现这个错误提示。导致大家无法获取准确的错误。
- 出现这个错误。
    - 首先检查js端的代码。js端发生错误，就报上面的错误。所以首要是去找js端的错误。



### 4.3.1 Android与js传递数据大小有限制



### 4.3.2 多次调用callHandler部分回调函数未被调用
- https://github.com/lzyzsd/JsBridge/issues/178
- https://github.com/lzyzsd/JsBridge/issues/170


### 4.3.3 字符串转义bug探讨
- 字符串转义bug。很多时候侨接不成功，这些问题本质上的原因都是通过js bridge传递数据转义有误导致。 
- 此bug会导致严重问题，如果传递的数据转义发生错误时，将导致不可用，像WebViewJavascriptBridge: WARNING: javascript handler threw.", source: (1) 这种错误很多时候都是因为js收到的数据和期望不符导致的异常（当然有些也有可能是js hanlder 处理不当抛出的）。这是一个偶现的致命bug 。要彻底解决这个问题最根本的方法就是不应该去转义，因为在传递数据格式未限定的情况下，只要转义，正常的数据字符串中都有可能匹配到转义规则（而这些字符串本身是不需要转义），这将会导致对于一部分数据能够正常转义，而一部分数据不能，这样的bug很难测试。 如果非要转义，就必须得限定jsbridge数据传递的格式，比如必须以json形式传递（不能直接传递string、bool等基础类型），这样才可以应用固定的转义规则解析。



### 4.3.8 Javascript调用原生方法会偶现失败
- 在测试过程中发现，失败的时机往往是webview调用 onPageFinished 前后，具体的表现是js调用native方法时 shouldOverrideUrlLoading（包括两种重载）没有被触发，所以端上没有去刷新js调用的message queue. 至于为什么没有就调用shouldOverrideUrlLoading，这是因为js和webview通信机制有问题，通过改变iframe src属性的这种方式并不能保证shouldOverrideUrlLoading每次都会被调用，这也是一些其它android jsbridge 会出现此问题的原因。



### 4.3.9 dispatchMessage运行主线程问题
- 先看一下案例代码
    ```
    // 必须要找主线程才会将数据传递出去 --- 划重点
    if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
        X5LogUtils.d("分发message--------------"+javascriptCommand);
        //this.loadUrl(javascriptCommand);
        //开始执行js中_handleMessageFromNative方法
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT &&
                javascriptCommand.length()>=URL_MAX_CHARACTER_NUM) {
            this.evaluateJavascript(javascriptCommand,null);
        }else {
            this.loadUrl(javascriptCommand);
        }
    }
    ```
- 可以看到，只有当是主线程时，才会执行分发的js，那么如果不是主线程呢？那自然就不会被分发，如果要确保这段代码没问题，那么就必须保证dispatchMessage是在主线程中被调用，而callHandler最终会调用dispatchMessage，所以就得保证callHandler必须在主线程调用。如果强制让用户去遵守这个规则是不靠谱的，况且有些时候，用户也不知道自己是否在主线程，示例代码中有在 onPageFinished时调用callHandler，不错onPageFinished在大多数情况下都会在主线程中被回调，但是我查了一圈，从没看到任何官方文档有说onPageFinished会在主线程中被回调。所以，那些出现的callHanlder不能调用bug的魅族手机，最好先去检查一下是否在主线程调用的。 所以，库中是应该保证callHandler无论是在哪个线程发起的调用, 最终的js都能在主线程被执行（因为webview要执行 s代码只能在主线程）。



### 4.4.0 怎么实现WebView免流方案
- 市面上常见的免流应用，原理无非就是走“特殊通道”，让这一部分的流量不计入运营商的流量统计平台中。Android中要实现这种“特殊通道”，有几种方案。
    - vpn。目前运营商貌似没有采用这种方案，但确实是可行的。
    - 全局代理。把所有的流量中转到代理服务器中，代理服务器再根据流量判断是否属于免流流量。
    - IP直连。走这个IP的所有流量，服务器判断是否免流。
- 对于上面提到的几种方案，native页面所有的请求都是应用层发起的，实际上都比较好实现，但WebView的页面和资源请求是通过JNI发起的，想要拦截请求的话，需要一些功夫。
    - 网罗网上的所有方案，目前觉得可行的有两种，分别是全局代理和拦截WebViewClient.shouldInterceptRequest()。
    - 更多参考[如何设计一个优雅健壮的Android WebView](https://blog.klmobile.app/2018/02/27/design-an-elegant-and-powerful-android-webview-part-two/)



### 4.4.1 Channel is unrecoverably broken and will be disposed!
- 程序崩溃，但是不报错原因，在网上翻了好久，网上好多说是jni错误，但是有种情况也会出现这种错误，就是bitmap.recycle()调用不对
- 把bitmap注释掉，不报错。



### 4.4.2 定制js的alert,confirm和prompt对话框
- https://www.iteye.com/blog/gundumw100-1158719
- https://blog.csdn.net/u012246458/article/details/53665597


### 4.4.3 x5长按图片如何操作
- x5支持长按事件监听，代码如下所示：
    ```
    webView.setOnLongClickListener(new View.OnLongClickListener() {
        @Override
        public boolean onLongClick(View v) {
            return handleLongImage();
        }
    });
    
    /**
     * 长按图片事件处理
     */
    private boolean handleLongImage() {
        final WebView.HitTestResult hitTestResult = webView.getHitTestResult();
        // 如果是图片类型或者是带有图片链接的类型
        if (hitTestResult.getType() == WebView.HitTestResult.IMAGE_TYPE ||
                hitTestResult.getType() == WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE) {
            // 弹出保存图片的对话框
            new AlertDialog.Builder(EightActivity.this)
                    .setItems(new String[]{"查看大图", "保存图片到相册"},
                            new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            String picUrl = hitTestResult.getExtra();
                            //获取图片
                            Log.e("picUrl", picUrl);
                            switch (which) {
                                case 0:
    
                                    break;
                                case 1:
                                    break;
                                default:
                                    break;
                            }
                        }
                    })
                    .show();
            return true;
        }
        return false;
    }
    ```


### 4.4 4 x5长按文字内容如何自定义弹窗
- ActionMode的使用特别的简单，主要用到两个方法，startActionMode和ActionMode.Callback()，startActionMode:开启我们的菜单
    ```
    ActionMode.Callback mCallback=new ActionMode.Callback(){
        /**
         * 创建菜单的样式，返回true说明创建成功
         * @param actionMode
         * @param menu
         * @return
         */
        @Override
        public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
            MenuInflater menuInflater = actionMode.getMenuInflater();
            menuInflater.inflate(R.menu.action_mode,menu);
            return true;
        }
    
        @Override
        public boolean onPrepareActionMode(ActionMode actionMode, Menu menu) {
            return false;
        }
    
        /**
         * 当ActionMode的条目被点击的时候，调用这个方法
         * @param actionMode
         * @param menuItem
         * @return
         */
        @Override
        public boolean onActionItemClicked(ActionMode actionMode, MenuItem menuItem) {
            return false;
        }
    
        /**
         * 当ActionMode被销毁的时候调用
         * @param actionMode
         */
        @Override
        public void onDestroyActionMode(ActionMode actionMode) {
            if(actionMode!=null){
                actionMode.finish();
            }
        }
    };
    ```
- 实现自定义ActionMode，重写startActionMode方法，拦截我们的ActionMode对象，然后对此进行一些处理就可以了，直接上代码
    ```
    @Override
    public ActionMode startActionMode(ActionMode.Callback callback) {
        ActionMode actionMode = super.startActionMode(callback);
        return resolveMode(actionMode);
    }
    
    @Override
    public ActionMode startActionMode(ActionMode.Callback callback, int type) {
        ActionMode actionMode = super.startActionMode(callback, type);
        return resolveMode(actionMode);
    }
    
    public ActionMode resolveMode(ActionMode actionMode) {
        if(actionMode!=null){
            final Menu menu = actionMode.getMenu();
            menu.clear();
            for (int i = 0; i < title.length; i++) {
                menu.add(title[i]);
            }
            for (int i = 0; i < title.length; i++) {
                MenuItem item = menu.getItem(i);
                item.setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(MenuItem menuItem) {
    
                        String title = menuItem.getTitle().toString();
                        getSelectedData(title); //获取选中的h5页面的文本
                        releaseActionMode();
                        return true;
                    }
                });
            }
            this.mActionMode = actionMode;
        }
        return actionMode;
    }
    ```
- 当点击ActionMode的item的之后，将我们的actionMode finish掉
    ```
    public void releaseActionMode() {
        if (mActionMode != null) {
            mActionMode.finish();
            mActionMode = null;
        }
    }
    ```
- 获取h5页面的文本信息，需要使用到js方法来帮助我们实现这些功能，然后在通过js和java交互回传我们的文本内容（js和java如何交互，这里就不多说了......）
    ```
    /**
     * 点击的时候，获取网页中选择的文本，回掉到原生中的js接口
     * @param title 传入点击的item文本，一起通过js返回给原生接口
     */
    private void getSelectedData(String title) {
    
        String js = "(function getSelectedText() {" +
                "var txt;" +
                "var title = \"" + title + "\";" +
                "if (window.getSelection) {" +
                "txt = window.getSelection().toString();" +
                "} else if (window.document.getSelection) {" +
                "txt = window.document.getSelection().toString();" +
                "} else if (window.document.selection) {" +
                "txt = window.document.selection.createRange().text;" +
                "}" +
                "ActionModeJavaScript.callback(txt,title);" +           //回调java方法将js获取的结果传递过去
                "})()";
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {  //android系统4.4以上的时候调用js方法用这个
            evaluateJavascript("javascript:" + js, null);
        } else {
            loadUrl("javascript:" + js);
        }
    }
    ```


### 4.4.5 webView.goBack()会刷新页面吗


### 4.4.6 mWebView.scrollTo(0, 0)回顶部失效
- 思考一下，为何会失效



### 4.4.7 部分手机监听滑动顶部或底部失效
- 先来看一下如何监听webView滑动到顶部和底部的逻辑代码，可以说网上绝大多数的都是这样
    ```
    /**
     * 判断是否在顶部
     * @return                              true表示在顶部
     */
    private boolean isTop() {
        return mWebView.getScrollY() <= 0;
    }
    
    /**
     * 判断是否在底部
     * @return                              true表示在底部
     */
    private boolean isBottom() {
        //return  Math.abs(webContent - webNow)<1
        return mWebView.getHeight() + mWebView.getScrollY() >= mWebView.getContentHeight() * mWebView.getScale();
    }
    ```
- 出现问题，部分手机
    - 在部分设备上会有问题，这些安卓手机可以隐藏掉下面的返回键,home键和菜单键的.也可以显示.可能因为如此，webView就不知道它的底部具体在哪里了.Math.abs(webContent - webNow)的值有可能等于1甚至大于1。
- 建议添加下面逻辑
    - 要完整实现网页加载完毕，监听网页滚动到底部才触发某些事件的话，那就可以设个标识，在onPageFinished的时候设为true，然后判断if(isLoadFinish &&  scrollY !=0) 再回调自定义listener的onPageEnd，再加上加载完毕的标识为true，这样就可以实现网页加载完毕后，滚动到底部的监听了。




### 4.4.8 prompt的一个坑导致js挂掉
- 使用onJsPrompt实现js通信，在js调用​window.alert​，​window.confirm​，​window.prompt​时，​会调用WebChromeClient​对应方法，可以此为入口，作为消息传递通道，​通常会选Prompt作为入口，在App中就是onJsPrompt作为jsbridge的调用入口。
- 从表现上来看，onJsPrompt必须执行完毕，prompt函数才会返回，否则js线程会一直阻塞在这里。实际使用中确实会发生这种情况，尤其是APP中有很多线程的场景下，怀疑是这么一种场景：
    - 第一步：js线程在执行prompt时被挂起，
    - 第二部 ：UI线程被调度，恰好销毁了Webview，调用了 （webview的detroy），detroy之后，导致 onJsPrompt不会被回调，prompt一直等着，js线程就一直阻塞，导致所有webview打不开，一旦出现可能需要杀进程才能解决。
- 解决方案
    - 使用onJsPrompt实现js通信，建议使用handler处理消息，避免某些方法因为做了耗时操作导致js等待状态；还需要注意销毁webView的过程，在onStop设置setJavaScriptEnabled(false)，然后销毁。



### 4.4.9 webView背景设置为透明无效探索
- webView是一个使用方便、功能强大的控件，但由于webView的背景颜色默认是白色，在一些场合下会显得很突兀（比如背景是黑色，app中的夜间模式）。
- 首先看一下，网上的解决方案
    ```
    android:layerType="software"（没效果）
    mWebView.setBackgroundColor(0);（没效果）
    mWebView.setBackgroundDrawable(R.color.transparent);（没效果）
    ```
- 最后解决方案如下所示
    - 首先检查配置文件里application设置android:hardwareAccelerated=”false”，自己尝试后必须这样设置才行；
    - 在loadUrl后设置mWebView.setBackgroundColor(0);mWebView.getBackground().setAlpha(1); // 设置填充透明度 范围：0-255
    - 检查xml布局文件里的WebView的父层布局，也要设置背景为透明的；（之前也因为这个问题没发现，绕了很大一个圈）



### 4.5.0 如何屏蔽掉WebView中长按事件
- webView长按时将会调用系统的复制控件，具体可以看一下案例。案例中提到，你可以自定义长按逻辑，也可以屏蔽长按事件。具体屏蔽逻辑该如何操作呢？
    ```
    mWebView.setOnLongClickListener(new OnLongClickListener() {  
        @Override 
        public boolean onLongClick(View v) {  
          return true;  
        }  
    });
    ```


### 4.5.1 WeView出现OOM影响主进程如何避免
- WeView出现OOM，我在实际开发中没有遇到，倒是有点可惜。这个是看网上的，后期有待求证……
- 问题描述：由于WebView默认运行在应用进程中，如果WebView加载的数据过大（例如加载大图片），就可能导致OOM问题，从而影响应用主进程。
- 解决方案：为了避免WebView影响主进程，可以尝试将WebView所在的Activity运行在独立进程中。这样即使WebView出现了OOM问题，应用主进程也不会受到影响。具体做法也很简单，只要在AndroidManifest文件中为相应的Activity设置process属性即可。为Activity设置了process属性，意思就是让这个Activity运行在名为:remote的私有进程中。
- 需要注意，这种方式可能会有进程通信方面的问题，因此需要根据实际情况决定是否需要使用。



### 4.5.2 WebView域控制不严格漏洞
- 由于应用中的WebView没有对file:///类型的url做限制，可能导致外部攻击者利用Javascript代码读取本地隐私数据。
- 解决方案：
    - 1.如果WebView不需要使用file协议，直接禁用所有与file协议相关的功能即可。需要注意，即使禁用了File协议，也不影响对assets和resources资源的加载。它们的url格式分别为：file:///android_asset、file:///android_res。
    ```
    webSettings.setAllowFileAccess(false);
    webSettings.setAllowFileAccessFromFileURLs(false);
    webSettings.setAllowUniversalAccessFromFileURLs(false);
    ```
    - 2.如果WebView需要使用file协议，则应该禁用file协议的Javascript功能。具体方法为：在调用loadUrl方法前，以及在shouldOverrideUrlLoading方法中判断url的scheme是否为file。如果是file协议，就禁用Javascript，否则启用Javascript。
    ```
    //WebSettings
    webSettings.setAllowFileAccess(true);
    webSettings.setAllowFileAccessFromFileURLs(false);
    webSettings.setAllowUniversalAccessFromFileURLs(false);
     
    //调用loadUrl前
    if("file".equals(Uri.parse(url).getScheme())){//判断是否为file协议
        webView.getSettings().setJavaScriptEnabled(false);
    }else{
        webView.getSettings().setJavaScriptEnabled(true);
    }
    webView.loadUrl(url);
     
    //WebViewClient中做的操作
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        if("file".equals(request.getUrl().getScheme())){//判断是否为file协议
            view.getSettings().setJavaScriptEnabled(false);
        }else{
            view.getSettings().setJavaScriptEnabled(true);
        }
        return false;
    }
    
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if("file".equals(Uri.parse(url).getScheme())){//判断是否为file协议
            view.getSettings().setJavaScriptEnabled(false);
        }else{
            view.getSettings().setJavaScriptEnabled(true);
        }
        return false;
    }
    ```

### 4.5.3 下载文件时的路径穿越问题
- 下载文件时，如果文件名中包含“../”这样的字符，并且WebView并未对文件名进行过滤，就会出现文件路径穿越问题。攻击者可以借助这种方式将可执行文件写入一些特定的位置。
- 解决方案：在下载文件时对文件名进行判断，过滤“../”这样的字符。


### 4.5.4 WebView中http和https混合使用问题
- 在Android 5.0及以上，WebView可能在加载混合使用http和https的网页时出现异常。比如在一个https的安全网页中加载使用http协议的资源将会失败。
- 解决方案：在Android 5.0后利用WebSettings设置WebView支持http和https混合内容模式。
    ```
    if(Build.VERSION.SDK_INT>=21){
        //方式1
        webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_COMPATIBILITY_MODE);
    }
    
    //或者
    if(Build.VERSION.SDK_INT>=21){
        webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
    ```
- 需要注意，MIXED_CONTENT_ALWAYS_ALLOW这个模式是不安全的，建议先使用MIXED_CONTENT_COMPATIBILITY_MODE模式。这个模式会尝试以安全的方式加载部分http资源，另一部分http资源则不会被加载。资源是否能被加载的判断依据可能会随着版本的不同而改变，因此需要根据实际情况决定是否采用这一模式。


### 4.5.5 调用系统EMAIL发送邮件崩溃
- 崩溃日志：ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.SENDTO dat=mailto:xxxxxxxxxxxx@xxx.xxx }
- 在调用系统EMAIL发从邮件时，如果手机没有能接受SENDTO和mailto的应用，将会出现如下崩溃，特别是一些国行手机，这些手机里面没有安卓原生GMAIL。


### 4.5.7 WebView组件访问部分网页崩溃问题
- 在测试WebView组件时发现总是出现崩溃现像：提示：ERR_CLEARTEXT_NOT_PERMITTED
- 当时以为是权限问题，查找自己的AndroidManifest文件发现已经申请INTERNET权限了。看了网上的一些大佬的文章才知道，原来由于 Android P （9.0）限制了明文流量的网络请求，非加密的流量请求都会被系统禁止掉，所以如果访问没有https协议的网站默认不不可以访问的。
- 解决方案如下所示
    - 1.只访问带有https协议的网站。
    - 2.在AndroidManifest文件设置明文通信属性。
    ```
      ...
      <!--权限-->
    <uses-permission android:name="android.permission.INTERNET"/>
        <application
           ...
           <!--默认为false，即不允许未加密的网络流量通信-->
            android:cleartextTrafficPermitted="true"
          ...
    ```
    - 还有另一种操作，如下所示
    ```
    //在res文件夹下添加xml文件夹下添加network_security_config.xml文件
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config xmlns:android="http://schemas.android.com/apk/res/android">
        <debug-overrides>
            <trust-anchors>
                <!-- Trust user added CAs while debuggable only -->
                <certificates src="user" />
            </trust-anchors>
        </debug-overrides>
        <base-config cleartextTrafficPermitted="true" />
    </network-security-config>
    
    
    //在AndroidManifest.xml文件中添加属性
    //在application 中 添加android:networkSecurityConfig="@xml/network_security_config"
    
    <application
        android:name=".base.BaseApplication"
        android:networkSecurityConfig="@xml/network_security_config"
        android:theme="@style/AppTheme">
    ```




### 4.9.9 掘金问题反馈记录
- 使用JsBridge遇到的坑
    - 由于JsBridge采用 json字符串，客户端传给前端数据中/进行了转义，导致前端收到数据后解析不出来。二，当前端给Native端发消息时，如果发送的消息频率过快，导致队列清空 shouldurl不回调，最终callback不回调，客户端也就收不到消息了。









