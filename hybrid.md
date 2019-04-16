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
    
  
    

  
