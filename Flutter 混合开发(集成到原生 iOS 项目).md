对于 Flutter 的业务应用，我们更希望它可以集成到已有的项目中，这篇详细的介绍下如何将 Flutter 集成到 iOS 项目工程中，对于后续的通信、交互、管理等内容会在后面的篇章中介绍。

**创建 Flutter 模块**

创建 iOS 工程项目，命名为 FlutterMixDemo ，当然你也可以用已有的工程来集成。

<font color="#dd0000">注意1：</font> 将我们项目 BitCode 选项设置为 NO ， Flutter 目前还不支持。

接下来我们需要创建 Flutter 模块，进入已有工程目录，这里拿我的工程目录举例：

```Dart
flutterMix/FlutterMixDemo/  (FlutterMixDemo 是我的 iOS 工程项目)
进入在 flutterMix 目录下，终端执行命令：
flutter create -t module flutter_module 
```

<font color="#dd0000">注意2：</font>flutter_module 是自己命名的，但要记得字母都要小写，不然会报错。

该命令会创建一个 Flutter 项目模块，我们可以看下它的项目结构及内容。

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3h4djlmvwj30ms0kw0vs.jpg)

**将 Flutter 模块以 pods 的方式加入到已有项目中**

在我们的已有项目 FlutterMixDemo 中初始化 pods ，当然如果你的项目中已初始化过 pods ，请忽略。

```
cd FlutterMixDemo
pod init
```

这时我们项目中会多一个 Podfile 文件，我们在该文件最后面添加命令如下：

```
target 'FlutterMixDemo' do
end
# 新加命令
flutter_application_path = '../flutter_module'
eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
```

<font color="#dd0000">注意3：</font>flutter_application_path 是 Flutter 模块的路径，记得修改为你的模块名称。

添加好后，运行命令

```
pod install
```

配置 Dart 编译脚本

在项目Build Phases 选项中，点击左上角➕号按钮，选择 New Run Script Phase ，添加如下脚本

```shell
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
```

**使用 FlutterViewController**

所有的 Flutter 页面公用一个Flutter实例 (FlutterViewController) 。

我们已点击按钮后跳入 Flutter 页面举例：

```iOS
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = UIColor.whiteColor;
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button addTarget:self
               action:@selector(jumpToFlutter)
     forControlEvents:UIControlEventTouchUpInside];
    [button setTitle:@"jump to flutter" forState:UIControlStateNormal];
    [button setBackgroundColor:[UIColor orangeColor]];
    button.titleLabel.font = [UIFont systemFontOfSize:20 weight:UIFontWeightBold];
    button.frame = CGRectMake(95.0, 210.0, 160.0, 44.0);
    [self.view addSubview:button];
}

- (void)jumpToFlutter {
    FlutterViewController* flutterViewController = [[FlutterViewController alloc] init];
    [self.navigationController pushViewController:flutterViewController animated:YES];
}
```

不过这样跳转后本身 Flutter 页面也会有一个导航栏，原生也有一个，造成重复。当然我们可以把 Flutter 页面的 appBar 去掉就可以，如下：

```
 appBar: AppBar(
        title: Text('Flutter Page') ,
        leading: IconButton(icon: Icon(Icons.arrow_back_ios), onPressed:()=>Navigator.pop()),
      ),
```

不过针对整体的设计、管理、交互如何去处理优化，也是一个值得讨论的话题，我会接下来再介绍。