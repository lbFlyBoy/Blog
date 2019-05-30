上篇我们介绍了 Flutter 模块集成到已有的项目工程，接下来我们看看 Native 跟 Flutter 间的交互问题。

***交互通信***

Flutter 与原生之间的通信依赖灵活的消息传递方式：

1，Flutter 部分通过平台通道将消息发送到其应用程序的所在的宿主环境（原生应用）。

2，宿主环境通过监听平台通道，接收消息。然后它会调用平台的 API，响应 Flutter 发送的消息。

***Flutter主动 调用 宿主环境***

在 Flutter 中通过 MethodChannel 的 API 可以发送与方法相对于的消息，宿主环境  iOS 中通过 FlutterMethodChannel 接受方法的调用并返回结果。

Flutter 需要引入 `services.dart` 模块才可以使用 MethodChannel 

```Dart
import 'package:flutter/services.dart';
```

Flutter 中的调用代码

```Dart
const methodChannel = const MethodChannel("com.pages.flutter/call_native");
```

```Dart
RaisedButton(
        child: Text("call_native_method_no_params"),
        onPressed: (){
      		   methodChannel.invokeMethod("call_native_method_no_params",null);
        },
      ),
      RaisedButton(
        child: Text("call_native_method_params"),
        onPressed: (){
          Map<String,String> params = {"params":"flutter params"};
          methodChannel.invokeMethod("call_native_method_params",params);
        },
      ),
      RaisedButton(
        child: Text("call_native_method_native_result_callback"),
        onPressed: (){
          _nativeCallbackWithParams();
        },
      ),
      Text(_content,style: TextStyle(color: Colors.red),)
```

```Dart
Future<Null> _nativeCallbackWithParams() async{
  dynamic result;
  try {
      result = await methodChannel.invokeMethod(
            "call_native_method_native_result_callback", null);
  } on PlatformException catch (e) {
    result = "Failed to get params: '${e.message}'.";
  }
  setState(() {
    _content = result;
  });
}
```

iOS 中的调用代码

```objective-c
FlutterViewController* flutterViewController = [[FlutterViewController alloc] init];
flutterViewController.fd_prefersNavigationBarHidden = YES;
FlutterMethodChannel * messageChannel = [FlutterMethodChannel methodChannelWithName:@"com.pages.flutter/call_native" binaryMessenger:flutterViewController];
[messageChannel setMethodCallHandler:^(FlutterMethodCall * _Nonnull call, FlutterResult  _Nonnull result) {
        NSLog(@"flutter call native：\n method=%@ \n arguments = %@",call.method,call.arguments);
        if ([call.method isEqualToString:@"call_native_method_native_result_callback"]) {
            if (result) {
                result(@"flutter hello");
            }
        }else if([call.method isEqualToString:@"call_native_method_pop_flutter_nav"]){
            [weakSelf.navigationController popViewControllerAnimated:YES];
        }
    }];
[self.navigationController pushViewController:flutterViewController animated:YES];
```

分别看下控制台输出：

```
flutter call native：
method=call_native_method_no_params 
arguments = (null)
```

```
flutter call native：
method=call_native_method_params 
arguments = {
  params = "flutter params";
}
```

第三个事件会在 Flutter 页面显示flutter hello 该值由宿主环境返回。

<font color=red>注意:</font> 这里有个设计上的细节，上节提到过就是导航栏的问题，因为宿主环境有个导航栏，而 Flutter 自身也有导航栏，出现了矛盾，到底我们应该保留宿主环境的，还是 Flutter 页面使用自身，隐藏宿主环境的导航栏。我个人觉得后则更合理，Flutter 页面更了解自己导航栏具体功能、主题、交互及显示，我们只需要处理点击返回按钮 pop 回到宿主环境中，如下：

```Dart
appBar: AppBar(
      title: Text('Flutter Page') ,
      leading: IconButton(icon: Icon(Icons.arrow_back_ios), onPressed:()=>methodChannel.invokeMethod("call_native_method_pop_flutter_nav",null)),
    ),
```

我们只需要在宿主环境中监听到该事件后调用导航的 pop 功能。

***宿主环境主动调用 Flutter***

一般可以用作宿主环境为 Flutter 提供参数

EventChannel 是 Flutter 监听宿主环境的 API ，FlutterEventChannel 是 iOS 宿主环境与 Flutter 交互平台通道的 API 。

Flutter 代码片段

```Dart
const EventChannel eventChannel = const EventChannel('com.pages.flutter/call_flutter');
```

```Dart
@override
void initState(){
  super.initState();
 eventChannel.receiveBroadcastStream(12345).listen(_onEvent,onError: _onError);
}

void _onEvent(Object event){
  setState(() {
    _content = event.toString();
  });
}

void _onError(Object error){
  setState(() {
    _content = error.toString();
  });
}
```

iOS 宿主环境代码片段

```objective-c
NSString *eventChannelName = @"com.pages.flutter/call_flutter";
FlutterEventChannel *eventChannel = [FlutterEventChannel eventChannelWithName:eventChannelName binaryMessenger:flutterViewController];
[eventChannel setStreamHandler:self];
```

```objective-c
- (FlutterError *)onListenWithArguments:(id)arguments eventSink:(FlutterEventSink)events {
    if (events) {
        events(@"hi flutter");
    }
    return nil;
}

- (FlutterError* _Nullable)onCancelWithArguments:(id _Nullable)arguments {
    return nil;
}
```

两端交互通信方式就是这样的，这里也只是介绍了他们通信的方式，具体如何优雅的封装、规范交互流程还需要我们自己去考虑下。