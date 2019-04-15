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
    > 
  ### Native主动调用Js
  
  
