### webview是Android中常用组件，主要用来加载网页或HTML代码片段，而且现在市面上出现的非常流行Hbird开发，也是借助于webview来实现。现在就来说说native和js之间是怎么通过webview进行通讯的：  
#### native调用js中的方法,主要有两种方式：
1.直接使用webview.loadUrl("js地址")
```
//java
mWebView.loadUrl("javascript:show(" + result + ")");

//javascript
<script type="text/javascript">

function show(result){
    alert("result"=result);
    return "success";
}

</script>
```
需要注意的是名字一定要对应上，要不然是调用不成功的，而且还有一点是 JS 的调用一定要在 onPageFinished 函数回调之后才能调用，要不然也是会失败的。 

2.如果现在有需求，我们要得到一个 Native 调用 Web 的回调怎么办，Google 在 Android4.4 为我们新增加了一个新方法，这个方法比 loadUrl 方法更加方便简洁，而且比 loadUrl 效率更高，因为 loadUrl 的执行会造成页面刷新一次，这个方法不会，因为这个方法是在 4.4 版本才引入的，所以我们使用的时候需要添加版本的判断：
```
final int version = Build.VERSION.SDK_INT;
if (version < 18) {
    mWebView.loadUrl(jsStr);
} else {
    mWebView.evaluateJavascript(jsStr, new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}
```
#### js调用native，主要有三种方式：  
1.通过 addJavascriptInterface 方法进行添加对象映射  
```
这种是使用最多的方式了，首先第一步我们需要设置一个属性：

mWebView.getSettings().setJavaScriptEnabled(true);

这个函数会有一个警告，因为在特定的版本之下会有非常危险的漏洞，我们下面将会着重介绍到，设置完这个属性之后，Native 需要定义一个类：

public class JSObject {
    private Context mContext;
    public JSObject(Context context) {
        mContext = context;
    }

    @JavascriptInterface
    public String showToast(String text) {
        Toast.show(mContext, text, Toast.LENGTH_SHORT).show();
        return "success";
    }
}
...
//特定版本下会存在漏洞
mWebView.addJavascriptInterface(new JSObject(this), "myObj");

需要注意的是在 API17 版本之后，需要在被调用的地方加上 @addJavascriptInterface 约束注解，因为不加上注解的方法是没有办法被调用的，JS 代码也很简单：

function showToast(){
    var result = myObj.showToast("我是来自web的Toast");
}
```
2.利用 WebViewClient 接口回调方法拦截 url  

这种方式其实实现也很简单，使用的频次也很高，上面我们介绍到了 WebViewClient ，其中有个回调接口 shouldOverrideUrlLoading (WebView view, String url) ，我们就是利用这个拦截 url，然后解析这个 url 的协议，如果发现是我们预先约定好的协议就开始解析参数，执行相应的逻辑，我们先来看看这个函数的介绍：

注意这个方法在 API24 版本已经废弃了，需要使用 shouldOverrideUrlLoading (WebView view, WebResourceRequest request) 替代，使用方法很类似，我们这里就使用 shouldOverrideUrlLoading (WebView view, String url) 方法来介绍一下：
```
public boolean shouldOverrideUrlLoading(WebView view, String url) {
    //假定传入进来的 url = "js://openActivity?arg1=111&arg2=222"，代表需要打开本地页面，并且带入相应的参数
    Uri uri = Uri.parse(url);
    String scheme = uri.getScheme();
    //如果 scheme 为 js，代表为预先约定的 js 协议
    if (scheme.equals("js")) {
          //如果 authority 为 openActivity，代表 web 需要打开一个本地的页面
        if (uri.getAuthority().equals("openActivity")) {
              //解析 web 页面带过来的相关参数
            HashMap<String, String> params = new HashMap<>();
            Set<String> collection = uri.getQueryParameterNames();
            for (String name : collection) {
                params.put(name, uri.getQueryParameter(name));
            }
            Intent intent = new Intent(getContext(), MainActivity.class);
            intent.putExtra("params", params);
            getContext().startActivity(intent);
        }
        //代表应用内部处理完成
        return true;
    }
    return super.shouldOverrideUrlLoading(view, url);
}

代码很简单，这个方法可以拦截 WebView 中加载 url 的过程，得到对应的 url，我们就可以通过这个方法，与网页约定好一个协议，如果匹配，执行相应操作，我们看一下 JS 的代码：

function openActivity(){
    document.location = "js://openActivity?arg1=111&arg2=222";
}

这个代码执行之后，就会触发本地的 shouldOverrideUrlLoading 方法，然后进行参数解析，调用指定方法。这个方式不会存在第一种提到的漏洞问题，但是它也有一个很繁琐的地方是，如果 web 端想要得到方法的返回值，只能通过 WebView 的 loadUrl 方法去执行 JS 方法把返回值传递回去，相关的代码如下：

//java
mWebView.loadUrl("javascript:returnResult(" + result + ")");

//javascript
function returnResult(result){
    alert("result is" + result);
}

所以说第二种方式在返回值方面还是很繁琐的，但是在不需要返回值的情况下，比如打开 Native 页面，还是很合适的，制定好相应的协议，就能够让 web 端具有打开所有本地页面的能力了。
```
3.利用 WebChromeClient 回调接口的三个方法拦截消息  
这个方法的原理和第二种方式原理一样，都是拦截相关接口，只是拦截的接口不一样：
```
@Override
public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    return super.onJsAlert(view, url, message, result);
}

@Override
public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    return super.onJsConfirm(view, url, message, result);
}

@Override
public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
    //假定传入进来的 message = "js://openActivity?arg1=111&arg2=222"，代表需要打开本地页面，并且带入相应的参数
    Uri uri = Uri.parse(message);
    String scheme = uri.getScheme();
    if (scheme.equals("js")) {
        if (uri.getAuthority().equals("openActivity")) {
            HashMap<String, String> params = new HashMap<>();
            Set<String> collection = uri.getQueryParameterNames();
            for (String name : collection) {
                params.put(name, uri.getQueryParameter(name));
            }
            Intent intent = new Intent(getContext(), MainActivity.class);
            intent.putExtra("params", params);
            getContext().startActivity(intent);
            //代表应用内部处理完成
            result.confirm("success");
        }
        return true;
    }
    return super.onJsPrompt(view, url, message, defaultValue, result);
}

和 WebViewClient 一样，这次添加的是 WebChromeClient 接口，可以拦截 JS 中的几个提示方法，也就是几种样式的对话框，在 JS 中有三个常用的对话框方法：
onJsAlert 方法是弹出警告框，一般情况下在 Android 中为 Toast，在文本里面加入\n就可以换行；
onJsConfirm 弹出确认框，会返回布尔值，通过这个值可以判断点击时确认还是取消，true表示点击了确认，false表示点击了取消；
onJsPrompt 弹出输入框，点击确认返回输入框中的值，点击取消返回 null。
但是这三种对话框都是可以本地拦截到的，所以可以从这里去做一些更改，拦截这些方法，得到他们的内容，进行解析，比如如果是 JS 的协议，则说明为内部协议，进行下一步解析然后进行相关的操作即可，prompt 方法调用如下所示：

function clickprompt(){
    var result=prompt("js://openActivity?arg1=111&arg2=222");
    alert("open activity " + result);
}

这里需要注意的是 prompt 里面的内容是通过 message 传递过来的，并不是第二个参数的 url，返回值是通过 JsPromptResult 对象传递。为什么要拦截 onJsPrompt 方法，而不是拦截其他的两个方法，这个从某种意义上来说都是可行的，但是如果需要返回值给 web 端的话就不行了，因为 onJsAlert 是不能返回值的，而 onJsConfirm 只能够返回确定或者取消两个值，只有 onJsPrompt 方法是可以返回字符串类型的值，操作最全面方便。
```
