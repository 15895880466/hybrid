# Android侧webview与Js通信的方式
  Android 端webview与Js通信的方式很多，[Android：你要的WebView与 JS 交互方式 都在这里了](https://blog.csdn.net/carson_ho/article/details/64904691)
  该篇博客解释对比的十分清楚，简单再介绍下
  * 总体目录
  ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-29c6a46c81304f4f.png)
  ## 一、交互方式总结
  ### Js主动调用Native
  Js主动调Native主要有三种方式
  * 通过 WebView的addJavascriptInterface（）(@JavascriptInterface)
    > 该方法通过addJavascriptInterface（）将java对象映射到Js对象，js端直接调用即可，十分方便。但是该方法在Android4.2（17）之前有较大的安全漏洞，所以在17以后用添加注解@JavascriptInterface方式调用。
    > 优点：使用简单 
      缺点：API17之前有严重的安全漏洞
  * 通过 WebViewClient 的shouldOverrideUrlLoading ()拦截url
    > 1.js端通过修改iframe属性触发Android侧WebViewClient的回调方法shouldOverrideUrlLoading ()
      2.拦截、解析该 url 的协议
      3.如果检测到是预先约定好的协议，就调用相应方法 
    > 优点：对Api无要求，不存在安全漏洞，较为通用
      缺点：需要js与native侧协商格式，JS获取Android方法的返回值复杂  
  * 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）拦截JS对话框alert()、confirm()、prompt（）消息
    ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-1385f748618af886.png)
    > Android通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调分别拦截JS对话框 （即上述三个方法），得到他们的消息内容，然后解析即可。对比三个方法我们可以发现只有prompt（）可以返回任意类型的值，操作最全面方便、更加灵活；而alert（）对话框没有返回值；confirm（）对话框只能返回两种状态（确定 / 取消）两个值，因此promt()方法较为合适
  * 总结  
    对比三种方式如下图
    
    ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-8c91481325a5253e.png)
    
    可以发现，利用WebChromeClient的onJsPrompt（）方法拦截js侧的promt()，这种方式最合理
  ### Native主动调用Js
  * 通过WebView的loadUrl（）,及我们熟知的js注入
    > 通过webview的loadUrl()方法， mWebView.loadUrl("javascript:callJS()")，注意javascript为必加的前缀，callJS()为js对应方法名
    > 特别注意：
      1. JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。
      2. loadurl方法在url过长（2000个字符）时会失败，所以不要尝试将一些js文件通过注入的方式直接使用，[What is the maximum length of a URL in different browsers?](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers)
    > 优点：对Api无要求，不存在安全漏洞，较为通用
      缺点：对注入代码长度有限制，且该方法执行会使页面刷新，并且无返回值
  * 通过WebView的evaluateJavascript（）
    ```
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
          }
        });
      }
    ```
    > 优点：1. 该方法的执行不会使页面刷新。
           2. 有返回值，效率更高、使用更简洁。
      缺点：1. 要求Android4.4以上
           2. onReceiveValue(String value)，value会多一对引号，需要特殊处理
  * 总结
    ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-30f095d4c9e638fd.png)
    
  ### 总体对比
  * 对比图
    ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-613b57c93dff2eb8.png)
  
  ## JsBridge原理介绍
        Android侧JsBridge一般指 [JsBridge](https://github.com/lzyzsd/JsBridge)，该框架对应ios侧的
    [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge),两者的实现细节各有不同，但是总体原理一致。我们主要看一下其Js与Native通信原理的实现，对于具体的代码细节不做深究。
   ### JsBridge集成
    * Js端
    > 集成源码中的js文件，WebViewJavascriptBridge.js，注意此处不可以通过注入的方式实现，不要被各种讲解博客误导。
    * Android侧
    ```
    dependencies {
      compile 'com.github.lzyzsd:jsbridge:1.0.4'
      }
    ```
   ### Js调用Native
    步骤
      1. js侧
         ```
       //message：传递信息, responseCallback：回调
       function _doSend(message, responseCallback) {
            if (responseCallback) {
                //生成唯一callbackid用于标识该次jsbridge通信过程
                var callbackId = 'cb_' + (uniqueId++) + '_' + new Date().getTime();
                responseCallbacks[callbackId] = responseCallback;
                message.callbackId = callbackId;
             }
             sendMessageQueue.push(message);
             //src："yy://__QUEUE_MESSAGE__/"
             messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://' + QUEUE_HAS_MESSAGE;
           }
         ```
      2.native侧
         ```
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        try {
            url = URLDecoder.decode(url, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        if (url.startsWith(BridgeUtil.YY_RETURN_DATA)) { // 如果是返回数据
            webView.handlerReturnData(url);
            return true;
        } else if (url.startsWith(BridgeUtil.YY_OVERRIDE_SCHEMA)) { //
            webView.flushMessageQueue();
            return true;
        } else {
            return super.shouldOverrideUrlLoading(view, url);
        }
    }
         ```
         这里会走第二个if, 调用BridgeWebView的flushMessageQueue()方法
         ```
         /**
     * 刷新消息队列
     */
    void flushMessageQueue() {
        if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
            loadUrl(BridgeUtil.JS_FETCH_QUEUE_FROM_JAVA, new CallBackFunction() {

                @Override
                public void onCallBack(String data) {
                    ...
                }
            });
        }
    }
         ```
         在这个flushMessageQueue方法里, 如果当前是主线程就调用一个loadUrl方法
         ```
         public void loadUrl(String jsUrl, CallBackFunction returnCallback) {
        // jsUrl = "javascript:WebViewJavascriptBridge._fetchQueue();"
        this.loadUrl(jsUrl);
        // 添加至 Map<String, CallBackFunction>
        String functionName = BridgeUtil.parseFunctionName(jsUrl);
        // functionName = "_fetchQueue"
        responseCallbacks.put(functionName, returnCallback);
    }
         ```
         在这个方法里, 首先会调用WebViewJavascriptBridge的_fetchQueue()方法, 然后解析方法名字, 因为这里的方法名字是写死的, 其实就是_fetchQueue, 请记住这个名字, 因为后面会用到.然后将以这个_fetchQueue为key, 回调方法为value, 放到一个map里面.然后我们再去看js那端的方法.

      3.js侧
         ```
         // 提供给native调用,该函数作用:获取sendMessageQueue返回给native,由于android不能直接获取返回的内容,所以使用url shouldOverrideUrlLoading 的方式返回内容
    function _fetchQueue() {
        var messageQueueString = JSON.stringify(sendMessageQueue);
        console.log('messageQueueString = ' + messageQueueString);
        sendMessageQueue = [];
        // android can't read directly the return data, so we can reload iframe src to communicate with java
        var src = CUSTOM_PROTOCOL_SCHEME + '://return/_fetchQueue/' + encodeURIComponent(messageQueueString);
        messagingIframe.src = src;
    }
         ```
        sendMessageQueue这个数组我们在_doSend()方法中用到过, 里面push了一个message对象, json格式化之后字符串就是[{"handlerName":"xx","data":"xxx","callbackId":"cb_2_1532852750705"}]这样的, 然后将sendMessageQueue这个数组置空, 接着再次变更iframe的src属性, 触发java的shouldOverrideUrlLoading方法, 传递的url是类似yy://return/_fetchQueue/%5B%7B%22handlerName%22%3A%22takePhoto%22%2C%22data%22%3A1234%2C%22callbackId%22%3A%22cb_1_1532853343154%22%7D%5D这样的,现在又回到java端的方法了。
    4.native侧
        触发shouldOverrideUrlLoading方法，并走第一个if，触发handlerReturnData方法
        ```
        void handlerReturnData(String url) {
        // _fetchQueue
        String functionName = BridgeUtil.getFunctionFromReturnUrl(url);
        //取出flushMessageQueue方法中放入responseCallbacks队列中的callback 
        CallBackFunction f = responseCallbacks.get(functionName);
        //取出js侧传来的数据
        String data = BridgeUtil.getDataFromReturnUrl(url);
        if (f != null) {
            //执行callback
            f.onCallBack(data);
            responseCallbacks.remove(functionName);
            return;
        }
    }
        ```
        在看一下这个callback
        
        ```
         void flushMessageQueue() {
        if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
            loadUrl(BridgeUtil.JS_FETCH_QUEUE_FROM_JAVA, new CallBackFunction() {

                @Override
                public void onCallBack(String data) {
                    // deserializeMessage 反序列化消息
                    List<Message> list = null;
                    try {
                        list = Message.toArrayList(data);
                    } catch (Exception e) {
                        e.printStackTrace();
                        return;
                    }
                    if (list == null || list.size() == 0) {
                        return;
                    }
                    for (int i = 0; i < list.size(); i++) {
                        Message m = list.get(i);
                        String responseId = m.getResponseId();
                        // 是否是response  CallBackFunction
                        if (!TextUtils.isEmpty(responseId)) {
                            CallBackFunction function = responseCallbacks.get(responseId);
                            String responseData = m.getResponseData();
                            function.onCallBack(responseData);
                            responseCallbacks.remove(responseId);
                        } else {
                            CallBackFunction responseFunction = null;
                            // if had callbackId 如果有回调Id
                            final String callbackId = m.getCallbackId();
                            if (!TextUtils.isEmpty(callbackId)) {
                                responseFunction = new CallBackFunction() {
                                    @Override
                                    public void onCallBack(String data) {
                                        Message responseMsg = new Message();
                                        responseMsg.setResponseId(callbackId);
                                        responseMsg.setResponseData(data);
                                        queueMessage(responseMsg);
                                    }
                                };
                            } else {
                                responseFunction = new CallBackFunction() {
                                    @Override
                                    public void onCallBack(String data) {
                                        // do nothing
                                    }
                                };
                            }
                            // BridgeHandler执行
                            BridgeHandler handler;
                            if (!TextUtils.isEmpty(m.getHandlerName())) {
                                handler = messageHandlers.get(m.getHandlerName());
                            } else {
                                handler = defaultHandler;
                            }
                            if (handler != null){
                                handler.handler(m.getData(), responseFunction);
                            }
                        }
                    }
                }
            });
        }
    }
        ```


    

  
