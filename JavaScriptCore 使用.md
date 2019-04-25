**前言**

动态化是移动开发技术中的重要的一部分 ，当前普遍的动态化方案 ， 如 React Native 、Weex 、Hybrid部分解决方案及之前流行的热修复框架 JSPatch ，背后都用到了 JavaScriptCore 框架 ，由它建立起 OC 跟 JS 语言沟通的桥梁 。

**JavaScriptCore 介绍**

JavaScriptCore 是 Safari 浏览器 JavaScript 引擎 ，它用来解释和执行 JavaScript 代码 。

JavaScriptCore 框架是一个苹果在 iOS7 引入的框架 ，该框架让 Objective-C 和 JavaScript 代码直接的交互变得更加的简单方便 ，其实就是基于 webkit 中以C/C++实现的 JavaScriptCore 的一个 OC 版本的封装 。

 JavaScriptCore 和 JavaScriptCore 框架是不同的两个概念 ，可以自己理解下 。



**OC 调用 JS 代码**

```
// 直接执行js代码
JSContext *cxt = [JSContext new];
JSValue *val =  [cxt evaluateScript:@"(function ocCallJS() { return 'ocCallJS'})()"];
NSLog(@"%@",[val toString]); // ocCallJS

// 注册js方法，利用JSValue调用
JSContext *cxt = [JSContext new];
JSValue *jsFunction = [cxt evaluateScript:@" (function(arg) { return arg })"];
JSValue *val = [jsFunction callWithArguments:@[@"hello objc"]];
NSLog(@"%@",[val toString]); // hello objc
```

这里有几个对象理解下

 `JSContext `

JSContext 对象表示 JavaScript 执行环境 ，所有  JavaScript 执行发生在上下文 ， 所有 JavaScript 值中与上下文联系在一起 。

`JSValue`

JSValue 实例是对 JavaScript 值的包装 ( 引用 ) ，您可以使用 JSValue 类在 JavaScript 和 Objective-C 或 Swift 表示之间转换基本值（例如数字和字符串），以便在本机代码和 JavaScript 代码之间传递数据。您还可以使用此类创建 JavaScript 对象，这些对象包含自定义类或 JavaScript 函数的本机对象，这些函数的实现由本机方法或块提供。

**JS 调用 OC 代码**

`Block 方式`

```
// 注册一个 oc 方法给 js 调用
JSContext *cxt = [JSContext new];
cxt[@"nativeMethod"] = ^(NSString *msg) {
  NSLog(@"%@",msg);	//  jsCallOC
};
// js 调用 oc 的方法
[cxt evaluateScript:@"nativeMethod('jsCallOC')"];  
```

`JSExport方式`

```
// 定义类 暴露给 js
@protocol JSBridgeObjProtocol <JSExport>
- (NSString *)fetchArticleContent;
@end

@interface JSBridgeObj : NSObject<JSBridgeObjProtocol>
@property (nonatomic, copy) NSString *articleTitle;
- (NSString *)fetchArticleContent;
@end

@implementation JSBridgeObj
- (NSString *)fetchArticleContent {
    return @"js call oc";
}
@end

// js 调用 oc 方法
JSContext *cxt = [JSContext new];
cxt[@"jsBridge"] = [JSBridgeObj new];
JSValue *val = [cxt 		  evaluateScript:@"jsBridge.fetchArticleContent()"];
NSLog(@"%@",[val toString]); // js call oc
```

JXExport  实现的协议将 OC 类及其实例方法，类方法和属性导出到 JavaScript 代码

这样基于 JSContext 我们可以完成两种语言间通信

**Hybrid 中的应用**

APP 混合开发中 ，在 UIWebView 中获取 JSContext 对象 ，该操作借用了苹果的私有方法 。

```
// 获取当前 WebView 的 JS 上下文
JSContext *context=[webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
```

不过该方法获取 JS 上下文有几个问题：

1 )  获取 JS 上下文的时机不确定 ，比如创建 UIWebView 对象 ，UIWebView 不同代理回调方法中获取到 JS 上下文都是不一样的 ，而且每次加载一个新的 URL 时 ， 都会废弃旧的 JS 上下文 ，创建新的 JS 上下文 。

因此获取 JS 上下文的时间点很重要 ，也就是在刚刚创建好新的 JS 上下文那一刻 。

只不过苹果并没有在 iOS 的 SDK 中暴露出来 ，而 macOS 的 SDK 中有获取创建好的 JS 上下文的代理方法。

```webView:didCreateJavaScriptContext:forFrame:
webView:didCreateJavaScriptContext:forFrame:
```

在 GitHub 上有这样一个项目 [TS_JavaScriptContext](https://github.com/TomSwift/UIWebView-TS_JavaScriptContext) 可以拿到了 JS 上下文创建的事件 ，只不过也是改获取方法也是苹果的私有 API ， 原来项目中使用了这个库上架苹果应用商店没有问题 ，现在审核情况不太了解 。

2 ） WKWebView 目前我还没有找到获取 JS 上下文的方法

在 UIWebView 中获取 JS 上下文的方法在 WKWebView 中是不起作用的 。	

WKWebView 不支持 JavaScriptCore 的方式, 但提供 messagehandler 的方式为 JS 与 OC 通信 。关于 WKWebView 相关知识 ，后续再聊 。