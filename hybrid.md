# Android侧webview与Js通信的方式
  Android 端webview与Js通信的方式很多，[Android：你要的WebView与 JS 交互方式 都在这里了](https://blog.csdn.net/carson_ho/article/details/64904691)
  该篇博客解释对比的十分清楚，简单再介绍下
  * 总体目录
  ![avatar](https://github.com/15895880466/hybrid/blob/master/image/944365-29c6a46c81304f4f.png)
  ## 一、交互方式总结
  ### Js主动调用Native
  Js主动调Native主要有三种方式
  * 通过 WebView的addJavascriptInterface（）
  * 通过 WebViewClient 的shouldOverrideUrlLoading ()拦截url
  * 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）拦截JS对话框alert()、confirm()、prompt（）消息
  ### Native主动调用Js
  
  
