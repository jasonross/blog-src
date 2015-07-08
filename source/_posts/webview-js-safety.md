title: Android Webview Java和Javascript安全交互
date: 2014-09-16 23:25:18
tags: [webview,Android,Javascript,安全]
---

最近要对一个网页的源代码进行检测，Android Webview中没有直接获取网页源代码的接口，传统的`addJavascriptInterface`方法存在安全隐患，所以研究了一下Java和Javascript的安全交互。

##Android Webview漏洞
Android Webview有两个非常知名的漏洞:

 - 最近爆出来的UXSS漏洞，可以越过同源策略，获得任意网页的Cookie等信息，Android 4.4以下都有此问题，基本无解，只能重新编译浏览器内核解决，详情可以参考[最近移动安全三两事][1]，感兴趣的可以去看一下[@RAyH4c][2]劫持微博、QQ空间的视频。
 - 成名已久的任意命令执行漏洞，通过`addJavascriptInterface`方法，Js可以调用Java对象方法，通过反射机制，Js可以直接获取Runtime，从而执行任意命令。Android 4.2以上，可以通过声明`@JavascriptInterface`保证安全性，4.2以下不能再调用`addJavascriptInterface`，需要另谋他法。

##Java和Javascript安全交互
首先要说明几点：

1.Android Webview中Java调用Js方法很容易，`loadUrl("javascript:isOk()")`就可以调用isOk这个Js方法，但不能直接获取Js方法的返回结果。
2.传统的方法中，Js获取Java信息可以采用如下方式：

     class JsObject {
           @JavascriptInterface
           public String toString() { return "injectedObject"; }
        }
        webView.addJavascriptInterface(new JsObject(), "injectedObject");
        webView.loadData("", "text/html", null);
        webView.loadUrl("javascript:alert(injectedObject.toString())");

 Java获取Js信息（如通过Js获取网页源代码）可以这样：

    import android.app.Activity;
    import android.graphics.Bitmap;
    import android.os.Bundle;
    import android.util.Log;
    import android.webkit.WebView;
    import android.webkit.WebViewClient;
         
    public class HtmlSource extends Activity {
        private WebView webView;
        
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            webView = (WebView)findViewById(R.id.webview);
            webView.getSettings().setJavaScriptEnabled(true);
            webView.addJavascriptInterface(new InJavaScriptLocalObj(), "local_obj");
            webView.setWebViewClient(new MyWebViewClient());
            webView.loadUrl("http://www.cnblogs.com/hibraincol/");
        }
         
         
       final class MyWebViewClient extends WebViewClient{  
            public boolean shouldOverrideUrlLoading(WebView view, String url) {   
                view.loadUrl(url);   
                return true;   
            }  
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                Log.d("WebView","onPageStarted");
                super.onPageStarted(view, url, favicon);
            }    
            public void onPageFinished(WebView view, String url) {
                Log.d("WebView","onPageFinished ");
                view.loadUrl("javascript:window.local_obj.showSource('<head>'+" +
                    "document.getElementsByTagName('html')[0].innerHTML+'</head>');");
                super.onPageFinished(view, url);
            }
        }
         
        final class InJavaScriptLocalObj {
            
            public void showSource(String html) {
                Log.d("HTML", html);
            }
        }
    }

3.当网页中有超链接跳转时，将会调用WebClient的`shouldOverrideUrlLoading`方法，若设置 WebViewClient 且该方法返回 true，则说明由应用的代码处理该 url，WebView 不处理，就可以达到拦截跳转的效果。

明白了上面几点，我们可以总结出一个比较安全的Java和Js交互方式：
> 可以借鉴Android Intent的思路，Java和Js定义一个url格式如`js://_`,Java调用Js方法，在Js方法中通过`window.location.href='js://_?key=value#key1=value1'`模拟跳转，被Java的`shouldOverrideUrlLoading`捕获，函数的返回值可以放在url的参数中。（Js调用Java方法原理相同）
这样的交互方式是异步的，如果你想知道调用一个Js方法是否返回了值怎么办？一般Java调用Js方法是在`onPageFinished`方法中，获得Js返回值是在`shouldOverrideUrlLoading`方法中，两个方法有个共同的参数webview,所以可以首先`webview.setTag(false)`，如果捕获到返回结果，则`webview.setTag(true)`,postDelayed在很短时间比如300毫秒后，`webview.getTag()`检查是否有变化即可。

  [1]: http://zhuanlan.zhihu.com/fooying/19840752
  [2]: http://weibo.com/rayh4c
