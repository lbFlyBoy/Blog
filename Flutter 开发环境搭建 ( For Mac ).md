谷歌发布 [Flutter for web](https://github.com/flutter/flutter_web)，正式宣布 Flutter 成为全平台框架，支持手机、`Web`、桌面电脑和嵌入式设备。现在学跨平台应用开发，第一个要看的可能不是 `React Native`，而是 `Flutter`，来自[阮一峰的评价](http://www.ruanyifeng.com/blog/2019/05/weekly-issue-55.html)。

学习一门开发框架，首先从配置框架的开发环境开始，这篇文章记录了自己在 `Mac` 电脑上如何搭建 `Flutter` 开发环境，参考[中文文档](https://flutterchina.club/get-started/install/)，也是有坑要踩的。

**系统要求**

`Flutter` 支持在 `Window`、`MacOS`、`Linux`等操作系统环境下开发 。

**安装 `Flutter`**

1. 系统环境变量配置

Google 提供的服务，国内无法访问的或速度慢的要死，所以下载资源需要翻墙或者官方提供的镜像，Flutter 官方为中国开发者搭建了[临时镜像 for china](https://flutter.dev/community/china)。

在 .bash_profile 文件中添加镜像地址 ：

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

终端命令：
```
1、vim ~/.bash_profile
2、将上面的镜像地址添加到该文件中，保存退出。
3、source ~/.bash_profile 添加的地址生效
```

2. 下载 `Flutter SDK` 

  ` Flutter` [官网下载地址](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos)分为三个不同的版本：

- `Stable channel` (稳定版)

- `Dev channel`     (开发版)

  最新的完全测试过的版本。也包含了新功能，但是也会有一些 `" bad " dev builds`，可以查看 [Bad Builds](https://github.com/flutter/flutter/wiki/Bad-Builds) 列表。

- `Beta channel`    (测试版)

  每隔几周都会选取近几个月中最好的一个 `dev` 版本，当作 `beta` 版，这个版本是通过了`Google`的 [codelabs](https://github.com/flutter/flutter/wiki/Codelabs) 测试的。

建议大家选择稳定版进行开发，不过官网下载确实很慢，不过可以参考 [Using Flutter in China](https://flutter.io/community/china) 方法替换 `host` ，亲测有效。

```
- Original URL:
https://storage.googleapis.com/flutter_infra/releases/stable/windows/flutter_windows_v1.0.0-stable.zip

- Mirrored URL:
https://storage.flutter-io.cn/flutter_infra/releases/stable/windows/flutter_windows_v1.0.0-stable.zip
```

也可以到 `GitHub` 下载最新版 [Flutter SDK](https://github.com/flutter/flutter)

3. 安装`Flutter`

解压 `Flutter SDK` 到自定义目录 

获取 `Flutter SDK` 解压缩后 `flutter/bin` 的完整路径 ，例如我的路径是

```
/Users/admin/Documents/workspace/flutter/flutter/bin
```

解释下 `Mac` 中的 `$HOME` 是指定当前用户的主目录的环境变量

```
终端中输入 
echo $HOME
输出
/User/admin
```

我们需要把 Flutter 命令所在目录添加到系统的 PATH 变量中，方便后续在任何终端直接使用，而不用切换到特定目录。

```
export PATH=$PATH:$HOME/Documents/workspace/flutter/flutter/bin
```

同添加镜像地址一样将这个添加到 `.bash_profile` 文件中 。

**设置 Flutter 编译**

在终端运行 `flutter doctor` 命令查看是否需要安装其它依赖项来完成安装，备注 (`Dart SDK` 已经包含在 `Flutter SDK`中无须再单独下载安装 )		

执行该命令会得到相关工具配置的详细信息：

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel beta, v1.0.0, on Mac OS X 10.13.6 17G65, locale zh-Hans-US)
[!] Android toolchain - develop for Android devices (Android SDK 26.0.2)
    ! Some Android licenses not accepted.  To resolve this, run: flutter doctor
      --android-licenses
[!] iOS toolchain - develop for iOS devices (Xcode 10.1)
    ✗ libimobiledevice and ideviceinstaller are not installed. To install with
      Brew, run:
        brew update
        brew install --HEAD usbmuxd
        brew link usbmuxd
        brew install --HEAD libimobiledevice
        brew install ideviceinstaller
    ✗ ios-deploy not installed. To install with Brew:
        brew install ios-deploy
[✓] Android Studio (version 3.0)
    ✗ Flutter plugin not installed; this adds Flutter specific functionality.
    ✗ Dart plugin not installed; this adds Dart specific functionality.
[✓] Connected device (1 available)

! Doctor found issues in 2 categories.
```

我们只需要关心画 ` ✗` 的内容，然后按照提示安装所需的工具配置。

```
✗ Flutter plugin not installed; this adds Flutter specific functionality.
```

对于上面的的提示错误 ，如果没用过 `Android Studio` 来说，可能不知道怎么解决，这里是说 `Android Studio` 需要安装 `Flutter` 插件，在 `Andriod Studio` 的偏好设置里。

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g30vcsmq9wj30st0itjun.jpg)

直到 `flutter doctor` 的运行结果都是 `[✓]` ，编译环境工具配置就 `OK` 了。

**`IDE` 配置**

`Flutter` 的集成开发环境 `IDE` 有 `Andriod Studio` 、`VS Code`、`IntelliJ`等，不过这里我选择了 `Andriod Studio` ，毕竟 `Flutter` 是 `Google` 自家的产品，

`Andriod Studio` 创建 `Flutter` 工程，不过注意需要在偏好设置指定 `Flutter SDK path`

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g30vyt1wqfj30st0itjuu.jpg)

接下来创建工程一直选择 `next`

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g30vvnzmhyj30q20rgtda.jpg)

![](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g30vvzuodoj30m80ie3zt.jpg)

对于 `iOS`、`Andriod` 配置 ，其实大部分工作 `dector` 都跟踪解决了，对与模拟器真机调试 、证书安装等，因为我的电脑 `Xcode` 之前都配置好，所以一路下来都没有遇到问题，如果你遇到什么问题，欢迎交流学习 ，一起进步。