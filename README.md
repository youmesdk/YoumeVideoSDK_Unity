# Video SDK for Unity3D 使用指南

## 适用范围

本文档适用于游密实时音视频引擎（Video SDK）Unity3D平台下接入，API接口说明部分同样适用于其他平台，可互相参考。

## SDK目录概述

YMRTC SDK提供了以下文件：

- Plugins库文件，分为Android平台和iOS平台。
>`AndroidManifest.xml`：可用来配置安卓下SDK所需要的权限、服务等。
`Plugins/Android`： Android平台使用的动态库，包括ARMv5、ARMv7和 X86 三种 CPU 架 构下的libyoume_voice_engine.so文件,还包括youme_voice_engine.jar。
`Plugins/iOS`：iOS平台使用的静态库，包含libyoume_voice_engine.a文件。

- YouMeVoiceEngine文件夹，内含封装SDK的C#接口文件。
>`YouMeVoiceAPI.cs`：封装了Video SDK 的全部功能接口。
>`YouMeConstDefine.cs`：包含Video SDK 错误码等枚举类型定义。

## 开发环境集成

### 从Unity3D开发环境集成

- 双击 `unitypackage`包;
- 在弹出的`Import Unity Package`对话框中，所有的复选框打勾(如下图所示)，但要格外注意AndroidManifest.xml可能与工程已有或将有的同名文件直接覆盖，应先比对合并；

![1.png](https://youme.im/doc/images/1.png)

- 为应用选择要支持的CPU架构：
在Unity3D的`Project View`中选中`Assets/Plugins/Android/libs`下特定的CPU架构，比如`armeabi-v7a`，勾选右侧的`Inspector View`中的`Select platforms for plugin`下的`Android`右侧的复选框，此处`注意armeabi和armeabi-v7a两种ARM架构的库文件，只能勾选其中一种架构，否则会出现库文件冲突的错误`。

![2.png](https://youme.im/doc/images/2.jpg)

### 导出Android工程

- 在`File`->`Build Settings…`的`Platform`列表中选择`Android`，酌情点击`Switch Platform`按钮，然后勾选右侧`Google Android Project`；
- 点击`Build Settings`->`Export`，选择输出的Android工程路径，导出Android工程；
![3.gif](https://youme.im/doc/images/3.gif)

### 导出iOS工程

- 在`File`->`Build Settings…`的Platform列表中选择iOS，酌情点击`Switch Platform`按钮；
- 点击`Build Settings`->`Build`，选择输出iOS工程的路径，输入工程名字，导出iOS工程；
- 在Xcode中打开上一步输出的iOS工程，在工程配置中`Build Phases`->`Link Binary With Libraries`下拉菜单中添加添加这几个框架文件:`libsqlite3.0.tbd`、`libz.1.2.5.tbd`、`libresolv.9.tbd` 和 `CoreTelephony.framework`,`VideoToolbox.framework`,`AudioToolbox.framework`
- `AVKit.framework`,`CoreVideo.framework`,`CoreFoundation.framework`
- `AVFoundation.framework`,`SystemConfiguration.framework`,`CFNetwork.framework`
- `GLKit.framework`
- **为iOS10以上版本添加录音权限配置**
iOS10系统使用录音权限，需要在target的`info.plist`中新加`NSMicrophoneUsageDescription`键，值为字符串(授权弹窗出现时提示给用户)。首次录音时会向用户申请权限。配置方式如下：
![iOS10录音权限配置](https://youme.im/doc/images/im_iOS_record_config.jpg)

### 备注：
详细接口介绍可查看文档[Video SDK for unity-API手册.md](https://github.com/youmesdk/YoumeVideoSDK_Unity/blob/master/Video%20SDK%20for%20unity-API%E6%89%8B%E5%86%8C.md)

实际案例可点击此处下载->
[Video Demo for Unity](https://github.com/youmesdk/YoumeVideoDemo_Unity)

