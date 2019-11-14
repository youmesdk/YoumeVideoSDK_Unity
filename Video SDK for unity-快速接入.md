# Video SDK for Unity3D 快速接入

##适用范围

本文档适用于游密实时音视频引擎（Video SDK）Unity3D平台下接入，API接口说明部分同样适用于其他平台，可互相参考。

##SDK目录概述

YMRTC SDK提供了以下文件：

- Plugins库文件，分为Android平台和iOS平台。
>`AndroidManifest.xml`：可用来配置安卓下SDK所需要的权限、服务等。
`Plugins/Android`： Android平台使用的动态库，包括ARMv5、ARMv7和 X86 三种 CPU 架 构下的libyoume_voice_engine.so文件,还包括youme_voice_engine.jar。
`Plugins/iOS`：iOS平台使用的静态库，包含libyoume_voice_engine.a文件。

- YouMeVoiceEngine文件夹，内含封装SDK的C#接口文件。
>`YouMeVoiceAPI.cs`：封装了Video SDK 的全部功能接口。
>`YouMeConstDefine.cs`：包含Video SDK 错误码等枚举类型定义。

##开发环境集成

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
- ,`AVKit.framework`,`CoreVideo.framework`,`CoreFoundation.framework`
- ,`AVFoundation.framework`,`SystemConfiguration.framework`,`CFNetwork.framework`
- ,`GLKit.framework`
- **为iOS10以上版本添加录音权限配置**
iOS10系统使用录音权限，需要在target的`info.plist`中新加`NSMicrophoneUsageDescription`键，值为字符串(授权弹窗出现时提示给用户)。首次录音时会向用户申请权限。配置方式如下：
![iOS10录音权限配置](https://youme.im/doc/images/im_iOS_record_config.jpg)




### API调用说明
接口对象实例需要通过调用`YouMe.YouMeVoiceAPI.GetInstance ()`来获取，接口使用的基本流程为`初始化`->`收到初始化成功回调通知`->`加入频道`->`收到加入频道成功回调通知`->`使用其它接口`->`离开频道`->`反初始化`，要确保严格按照上述的顺序使用接口。

### 设置回调
* **语法**

```
void SetCallback(string strObjName);
```

* **功能**
设置接收回调消息的对象名。这个函数必须最先调用，这样才能收到后面所有调用的回调消息。

* **参数说明**
`strObjName`：string类型，注册的GameObject的名字。

* **返回值**
无。

### 实现回调函数
上面注册好回调的GameObject类还需实现以下方法：

#### 通用回调：
```
void OnEvent (string strParam);
```

* **代码示例**

```

	//以下代码仅供参考
	void OnEvent (string strParam)
	{
		string[] strSections = strParam.Split (new char[] { ',' });
		if (strSections == null){

			return;
		}

		YouMe.YouMeEvent eventType = (YouMeEvent)int.Parse (strSections [0]);
		YouMe.YouMeErrorCode errorCode = (YouMeErrorCode)int.Parse (strSections [1]);
		string channelID =  strSections [2];
		string userID =  strSections [3];

		switch (eventType) {
		//对eventType的case列举请查询枚举类型YouMeEvent的定义，以下只是部分列举
		//使用者请按需自行添加或删除
		case YouMe.YouMeEvent.YOUME_EVENT_INIT_OK:
			//"初始化成功";
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_INIT_FAILED:
	      //"初始化失败，错误码：" + errorCode;
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_JOIN_OK:
			//加入频道成功
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_LEAVED_ALL:
		   //"离开所有频道成功";
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_JOIN_FAILED:
			//进入语音频道失败
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_REC_PERMISSION_STATUS:
			//"通知录音权限状态，成功获取权限时错误码为YOUME_SUCCESS，获取失败为YOUME_ERROR_REC_NO_PERMISSION（此时不管麦克风mute状态如何，都没有声音输出）";
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_RECONNECTING:
			//"断网了，正在重连";
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_RECONNECTED:
			//"断网重连成功";
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_MEMBER_CHANGE:
			//房间内成员列表变化，param参数是包含自己的所有用户列表
			List<string> members = new List<string>(12);
               for(int index = 2 ; index < strSections.Length ;index++){
                   //会议内的用户，包括自己
                   members.Add(strSections[index]);
               }
			break;
		case  YouMe.YouMeEvent.YOUME_EVENT_OTHERS_MIC_OFF:
			//其他用户的麦克风关闭：param是关闭用户的userid
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_MIC_ON:
			//其他用户的麦克风打开
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_SPEAKER_ON:
			//其他用户的扬声器打开
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_SPEAKER_OFF:
			//其他用户的扬声器关闭
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_VOICE_ON:
			//其他用户开始讲话
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_VOICE_OFF:
			//其他用户结束讲话
			break;
		case YOUME_EVENT_MY_MIC_LEVEL:
			//麦克风的语音级别
			break;
		case YOUME_EVENT_MIC_CTR_ON:
			//麦克风被其他用户打开
			break;
		case YOUME_EVENT_MIC_CTR_OFF:
			//麦克风被其他用户关闭
			break;
		case YOUME_EVENT_SPEAKER_CTR_ON:
			//扬声器被其他用户打开
			break;
		case YOUME_EVENT_SPEAKER_CTR_OFF:
			//扬声器被其他用户关闭
			break;
		case YOUME_EVENT_LISTEN_OTHER_ON:
			//取消屏蔽某人语音
			break;
		case YOUME_EVENT_LISTEN_OTHER_OFF:
			//屏蔽某人语音
			break;
		//=====================视频相关=========================
		case YOUME_EVENT_OTHERS_VIDEO_ON:
		      //收到其他用户的视频流通知，可以开始渲染这个用户的视频流
		      //VideoJoin(userid);
		      break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_CAMERA_PAUSE:
			//userid 关闭了摄像头
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_CAMERA_RESUME:
			//userid 打开了摄像头
			break;
		case YouMe.YouMeEvent.YOUME_EVENT_OTHERS_VIDEO_SHUT_DOWN://视频断开事件
			{
				// 这个事件在暂停后、临时断网也会通知，不适合做用户下线通知
                // RemoveUserVideoRender(param);
			}
			break;
		default:
			// "事件类型" + eventType + ",错误码" + errorCode;
			break;
		}

	}

```

#### RestApi回调：
```
void OnRequestRestApi (string strParam);
```

`strParam`：json串,包含:
    `requestid`：整数类型，回传ID
    `error`：错误码
    `query`：回传查询命令,json串（包含command和query字段）
    `result`：查询结果，详情见RequestRestApi接口说明文档

#### 获取成员列表及成员变更回调：
```
void OnMemberChange (string strParam);
```

`strParam`：json串,包含:
    `channelid`：字符串，频道ID
    `memchange`：数字类型，成员列表，或者变更列表。（userid 字符串，用户ID ；
isJoin， bool类型，false为离开）

### 初始化
* **语法**

```
YouMeErrorCode Init(
string strAppKey,
string strAPPSecret,
YOUME_RTC_SERVER_REGION serverRegionId,
string strExtServerRegionName);
```

* **功能**
初始化语音引擎，做APP验证和资源初始化。

* **参数说明**
`strAPPKey`：从游密申请到的 app key, 这个你们应用程序的唯一标识。
`strAPPSecret`：对应 strAPPKey 的私钥, 这个需要妥善保存，不要暴露给其他人。
`serverRegionId`：设置首选连接服务器的区域码，如果在初始化时不能确定区域，可以填RTC_DEFAULT_SERVER，后面确定时通过 SetServerRegion 设置。如果YOUME_RTC_SERVER_REGION定义的区域码不能满足要求，可以把这个参数设为 RTC_EXT_SERVER，然后通过后面的参数strExtServerRegionName 设置一个自定的区域值（如中国用 "cn" 或者 “ch"表示），然后把这个自定义的区域值同步给游密，我们将通过后台配置映射到最佳区域的服务器。
`strExtServerRegionName`：自定义的扩展的服务器区域名。不能为null，可为空字符串“”。只有前一个参数serverRegionId设为RTC_EXT_SERVER时，此参数才有效（否则都将当空字符串“”处理）。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouNeErrorCode类型定义](#YouMeErrorCode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	// YOUME_EVENT_INIT_OK  - 表明初始化成功
	// YOUME_EVENT_INIT_FAILED - 表明初始化失败，最常见的失败原因是网络错误或者 AppKey-AppSecret 错误
	void OnEvent (string strParam);
```

### 加入语音频道（单频道）
* **语法**

```

	YouMeErrorCode JoinChannelSingleMode (string strUserID, string strChannelID, YouMeUserRole userRole);
```
* **功能**
加入语音频道（单频道模式，每个时刻只能在一个语音频道里面）。

* **参数说明**
`strUserID`：全局唯一的用户标识，全局指在当前应用程序的范围内。
`strChannelID`：全局唯一的频道标识，全局指在当前应用程序的范围内。
`userRole`：用户在语音频道里面的角色，见YouMeUserRole定义。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouNeErrorCode类型定义](#YouMeErrorCode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	//YOUME_EVENT_JOIN_OK - 成功进入语音频道
	//YOUME_EVENT_JOIN_FAILED - 进入语音频道失败，可能原因是网络或服务器有问题
	void OnEvent (string strParam);
```


### 渲染视频数据

  示例：

    ``` c#
    // 开启摄像头
    YouMe.YouMeVoiceAPI.GetInstance().StartCapture();
    // 根据用户id获取渲染id
    int videoRenderID = YouMeTexture.GetInstance ().CreateTexture (selfUserID, gameObject.name);
    // 更新视频，目前约15帧每秒
    YouMeTexture.GetInstance().SetVideoRenderUpdateCallback(videoRenderID,(Texture2D videoTexture)=>{
        // 把 videoTexture 这个Texture2D对象显示到游戏里即可
    });
    ```
    
  具体方法参见demo

#### 创建渲染
* **语法**

```
 	YouMeErrorCode CreateRender(string userId);
```

* **功能**
创建渲染

* **参数说明**
`userId`:用户id

* **返回值**
大于等于0 - renderId 小于0 - 具体错误码

### 开始摄像头采集

* **语法**

```
 	YouMeErrorCode StartCapture();
```

* **功能**
开始camera capture

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouNeErrorCode类型定义](#YouMeErrorCode类型定义)。



# Video SDK 状态码

## YouMeEvent类型定义

枚举常量 |值| 含义
-------|------|------
YOUME_EVENT_INIT_OK                   |0|SDK初始化成功
YOUME_EVENT_INIT_FAILED               | 1|SDK初始化失败
YOUME_EVENT_JOIN_OK                   | 2| 进入语音频道成功
YOUME_EVENT_JOIN_FAILED               | 3| 进入语音频道失败
YOUME_EVENT_LEAVED_ONE                | 4|退出单个语音频道完成
YOUME_EVENT_LEAVED_ALL                | 5|退出所有语音频道完成
YOUME_EVENT_PAUSED                    | 6|暂停语音频道完成
YOUME_EVENT_RESUMED                   | 7| 恢复语音频道完成
YOUME_EVENT_SPEAK_SUCCESS             | 8|切换对指定频道讲话成功（适用于多频道模式）
YOUME_EVENT_SPEAK_FAILED              | 9| 切换对指定频道讲话失败（适用于多频道模式）
YOUME_EVENT_RECONNECTING              | 10| 断网了，正在重连
YOUME_EVENT_RECONNECTED               | 11|断网重连成功
YOUME_EVENT_REC_PERMISSION_STATUS     | 12|通知录音权限状态，成功获取权限时错误码为YOUME_SUCCESS，获取失败为YOUME_ERROR_REC_NO_PERMISSION（此时不管麦克风mute状态如何，都没有声音输出）
YOUME_EVENT_BGM_STOPPED               | 13| 通知背景音乐播放结束
YOUME_EVENT_BGM_FAILED                | 14| 通知背景音乐播放失败
YOUME_EVENT_OTHERS_MIC_ON             | 16| 其他用户麦克风打开
YOUME_EVENT_OTHERS_MIC_OFF            | 17| 其他用户麦克风关闭
YOUME_EVENT_OTHERS_SPEAKER_ON         | 18| 其他用户扬声器打开
YOUME_EVENT_OTHERS_SPEAKER_OFF        | 19| 其他用户扬声器关闭
YOUME_EVENT_OTHERS_VOICE_ON           | 20|其他用户进入讲话状态
YOUME_EVENT_OTHERS_VOICE_OFF          | 21| 其他用户进入静默状态
YOUME_EVENT_MY_MIC_LEVEL              | 22| 麦克风的语音级别
YOUME_EVENT_MIC_CTR_ON                | 23| 麦克风被其他用户打开
YOUME_EVENT_MIC_CTR_OFF               | 24| 麦克风被其他用户关闭
YOUME_EVENT_SPEAKER_CTR_ON            | 25| 扬声器被其他用户打开
YOUME_EVENT_SPEAKER_CTR_OFF           | 26| 扬声器被其他用户关闭
YOUME_EVENT_LISTEN_OTHER_ON           | 27| 取消屏蔽某人语音
YOUME_EVENT_LISTEN_OTHER_OFF          | 28| 屏蔽某人语音
YOUME_EVENT_LOCAL_MIC_ON              | 29| 自己的麦克风打开
YOUME_EVENT_LOCAL_MIC_OFF             | 30| 自己的麦克风关闭
YOUME_EVENT_LOCAL_SPEAKER_ON	      | 31| 自己的扬声器打开
YOUME_EVENT_LOCAL_SPEAKER_OFF	      | 32| 自己的扬声器关闭
YOUME_EVENT_GRABMIC_START_OK          | 33| 发起抢麦活动成功
YOUME_EVENT_GRABMIC_START_FAILED      | 34| 发起抢麦活动失败
YOUME_EVENT_GRABMIC_STOP_OK           | 35| 停止抢麦活动成功
YOUME_EVENT_GRABMIC_STOP_FAILED       | 36| 停止抢麦活动失败
YOUME_EVENT_GRABMIC_REQUEST_OK	      | 37| 抢麦成功（可以说话）
YOUME_EVENT_GRABMIC_REQUEST_FAILED    | 38| 抢麦失败
YOUME_EVENT_GRABMIC_REQUEST_WAIT      | 39| 进入抢麦等待队列（仅权重模式下会回调此事件）
YOUME_EVENT_GRABMIC_RELEASE_OK        | 40| 释放麦成功
YOUME_EVENT_GRABMIC_RELEASE_FAILED    | 41| 释放麦失败
YOUME_EVENT_GRABMIC_ENDMIC            | 42| 不再占用麦（到麦使用时间或者其他原因）
YOUME_EVENT_GRABMIC_NOTIFY_START      | 43| [通知]抢麦活动开始
YOUME_EVENT_GRABMIC_NOTIFY_STOP       | 44| [通知]抢麦活动结束
YOUME_EVENT_GRABMIC_NOTIFY_HASMIC     | 45| [通知]有麦可以抢
YOUME_EVENT_GRABMIC_NOTIFY_NOMIC      | 46| [通知]没有麦可以抢
YOUME_EVENT_INVITEMIC_SETOPT_OK       | 47| 连麦设置成功
YOUME_EVENT_INVITEMIC_SETOPT_FAILED   | 48| 连麦设置失败
YOUME_EVENT_INVITEMIC_REQUEST_OK      | 49| 请求连麦成功（连上了，需等待对方回应）
YOUME_EVENT_INVITEMIC_REQUEST_FAILED  | 50| 请求连麦失败
YOUME_EVENT_INVITEMIC_RESPONSE_OK     | 51| 响应连麦成功（被叫方无论同意/拒绝都会收到此事件，错误码是YOUME_ERROR_INVITEMIC_REJECT表示拒绝）
YOUME_EVENT_INVITEMIC_RESPONSE_FAILED | 52| 响应连麦失败
YOUME_EVENT_INVITEMIC_STOP_OK         | 53| 停止连麦成功
YOUME_EVENT_INVITEMIC_STOP_FAILED     | 54| 停止连麦失败
YOUME_EVENT_INVITEMIC_CAN_TALK        | 55| 双方可以通话了（响应方已经同意）
YOUME_EVENT_INVITEMIC_CANNOT_TALK     | 56| 双方不可以再通话了（有一方结束了连麦或者连麦时间到）
YOUME_EVENT_INVITEMIC_NOTIFY_CALL     | 57| [通知]有人请求与你连麦
YOUME_EVENT_INVITEMIC_NOTIFY_ANSWER   | 58| [通知]对方对你的连麦请求作出了响应（同意/拒绝/超时，同意的话双方就可以通话了）
YOUME_EVENT_INVITEMIC_NOTIFY_CANCEL   | 59| [通知]连麦过程中，对方结束了连麦或者连麦时间到
YOUME_EVENT_SEND_MESSAGE_RESULT       | 60| sendMessage成功与否的通知，param为回传的requestID
YOUME_EVENT_MESSAGE_NOTIFY            | 61| 收到Message, param为message内容
YOUME_EVENT_KICK_RESULT               | 64| 踢人的应答, param: 被踢者ID
YOUME_EVENT_KICK_NOTIFY               | 65| 被踢通知   ,param: （踢人者ID，被踢原因，被禁时间）
YOUME_EVENT_FAREND_VOICE_LEVEL        | 66| 远端说话人音量大小
YOUME_EVENT_OTHERS_BE_KICKED          | 67| 房间里其他人被踢出房间
YOUME_EVENT_OTHERS_VIDEO_ON           | 200| 收到其它用户的视频流
YOUME_EVENT_MASK_VIDEO_BY_OTHER_USER  | 204| 视频被其他用户屏蔽
YOUME_EVENT_RESUME_VIDEO_BY_OTHER_USER| 205| 视频被其他用户恢复
YOUME_EVENT_MASK_VIDEO_FOR_USER       | 206| 屏蔽了谁的视频
YOUME_EVENT_RESUME_VIDEO_FOR_USER     | 207| 恢复了谁的视频
YOUME_EVENT_OTHERS_VIDEO_SHUT_DOWN    | 208| 其它用户的视频流断开（包含网络中断的情况）
YOUME_EVENT_OTHERS_VIDEO_INPUT_START  | 209| 其他用户视频输入开始（内部采集下开启摄像头/外部输入下开始input）
YOUME_EVENT_OTHERS_VIDEO_INPUT_STOP   | 210| 其他用户视频输入停止（内部采集下停止摄像头/外部输入下停止input）
YOUME_EVENT_MEDIA_DATA_ROAD_PASS      | 211| 音视频数据通路连通，定时检测，一开始收到数据会收到PASS事件，之后变化的时候会发送
YOUME_EVENT_MEDIA_DATA_ROAD_BLOCK     | 212| 音视频数据通路不通
YOUME_EVENT_QUERY_USERS_VIDEO_INFO    | 213| 查询用户视频信息返回
YOUME_EVENT_SET_USERS_VIDEO_INFO      | 214| 设置用户接收视频信息返回
YOUME_EVENT_LOCAL_VIDEO_INPUT_START   | 215| 本地视频输入开始（内部采集下开始摄像头/外部输入下开始input）
YOUME_EVENT_LOCAL_VIDEO_INPUT_STOP    | 216| 本地视频输入停止（内部采集下停止摄像头/外部输入下停止input）
YOUME_EVENT_OTHERS_DATA_ERROR         | 300| 数据错误
YOUME_EVENT_OTHERS_NETWORK_BAD        | 301| 网络不好
YOUME_EVENT_OTHERS_BLACK_FULL         | 302| 黑屏
YOUME_EVENT_OTHERS_GREEN_FULL         | 303| 绿屏
YOUME_EVENT_OTHERS_BLACK_BORDER       | 304| 黑边
YOUME_EVENT_OTHERS_GREEN_BORDER       | 305| 绿边
YOUME_EVENT_OTHERS_BLURRED_SCREEN     | 306| 花屏
YOUME_EVENT_OTHERS_ENCODER_ERROR      | 307| 编码错误
YOUME_EVENT_OTHERS_DECODER_ERROR      | 308| 解码错误
YOUME_EVENT_CAMERA_DEVICE_CONNECT     | 400| 摄像头设备插入，移动端无效
YOUME_EVENT_CAMERA_DEVICE_DISCONNECT  | 401| 摄像头设备拔出，移动端无效
YOUME_EVENT_EOF                       |1000|

## YouMeErrorCode类型定义

枚举常量|值|含义
----|------|-----
YOUME_SUCCESS                            |0| 成功
YOUME_ERROR_API_NOT_SUPPORTED            |-1| 正在使用的SDK不支持特定的API
YOUME_ERROR_INVALID_PARAM                |-2| 传入参数错误
YOUME_ERROR_ALREADY_INIT                 |-3| 已经初始化
YOUME_ERROR_NOT_INIT                     |-4| 还没有初始化，在调用某函数之前要先调用初始化并且要据返回值确保初始化成功
YOUME_ERROR_CHANNEL_EXIST                |-5| 要加入的频道已经存在
YOUME_ERROR_CHANNEL_NOT_EXIST            |-6| 要退出的频道不存在，或者其它操作指定的频道不存在
YOUME_ERROR_WRONG_STATE                  |-7| 状态错误
YOUME_ERROR_NOT_ALLOWED_MOBILE_NETWROK   |-8| 不允许使用移动网络
YOUME_ERROR_WRONG_CHANNEL_MODE           |-9| 在单频道模式下调用了多频道接口，或者反之
YOUME_ERROR_TOO_MANY_CHANNELS            |-10|同时加入了太多频道
YOUME_ERROR_TOKEN_ERROR                  |-11|传入的token认证错误
YOUME_ERROR_NOT_IN_CHANNEL               |-12|用户不在该频道
YOUME_ERROR_BE_KICK                      |-13|被踢了，还在禁止时间内，不允许进入房间
YOUME_ERROR_DEVICE_NOT_VALID             |-14|设置的设备不可用
YOUME_ERROR_API_NOT_ALLOWED              |-15|没有特定功能的权限，需要的话请联系我们
YOUME_ERROR_MEMORY_OUT                   |-100|内存错误
YOUME_ERROR_START_FAILED                 |-101|启动引擎失败
YOUME_ERROR_STOP_FAILED                  |-102| 停止引擎失败
YOUME_ERROR_ILLEGAL_SDK                  |-103|非法使用SDK
YOUME_ERROR_SERVER_INVALID               |-104|语音服务不可用
YOUME_ERROR_NETWORK_ERROR                |-105|网络错误
YOUME_ERROR_SERVER_INTER_ERROR           |-106|服务器内部错误
YOUME_ERROR_QUERY_RESTAPI_FAIL           |-107|请求RestApi信息失败了
YOUME_ERROR_USER_ABORT                   |-108|用户请求中断当前操作
YOUME_ERROR_SEND_MESSAGE_FAIL            |-109|发送消息失败
YOUME_ERROR_REC_INIT_FAILED              |-201|录音模块初始化失败
YOUME_ERROR_REC_NO_PERMISSION            |-202|没有录音权限
YOUME_ERROR_REC_NO_DATA                  |-203|虽然初始化成功，但没有音频数据输出，比如oppo系列机在录音权限被禁止的时候
YOUME_ERROR_REC_OTHERS                   |-204|其他录音模块的错误
YOUME_ERROR_REC_PERMISSION_UNDEFINED     |-205|录音权限未确定，iOS显示是否允许录音权限对话框时，回的是这个错误码
YOUME_ERROR_GRABMIC_FULL				 |-301|抢麦失败，人数满
YOUME_ERROR_GRABMIC_HASEND				 |-302|抢麦失败，活动已经结束
YOUME_ERROR_INVITEMIC_NOUSER			 |-401|连麦失败，用户不存在
YOUME_ERROR_INVITEMIC_OFFLINE			 |-402|连麦失败，用户已离线
YOUME_ERROR_INVITEMIC_REJECT			 |-403|连麦失败，用户拒绝
YOUME_ERROR_INVITEMIC_TIMEOUT			 |-404|连麦失败，两种情况：1.连麦时，对方超时无应答 2.话中，双方通话时间到
YOUME_ERROR_UNKNOWN                      |-1000|未知错误



## YOUME_RTC_SERVER_REGION类型定义

枚举常量|值|含义
----|------|-----
    RTC_CN_SERVER       | 0| 中国
    RTC_HK_SERVER       | 1| 香港
    RTC_US_SERVER       | 2| 美国东部
    RTC_SG_SERVER       | 3| 新加坡
    RTC_KR_SERVER       | 4| 韩国
    RTC_AU_SERVER       | 5| 澳洲
    RTC_DE_SERVER       | 6| 德国
    RTC_BR_SERVER       | 7| 巴西
    RTC_IN_SERVER       | 8| 印度
    RTC_JP_SERVER       | 9| 日本
    RTC_IE_SERVER       | 10|爱尔兰
    RTC_USW_SERVER      | 11|美国西部
    RTC_USM_SERVER      | 12|美国中部
    RTC_CA_SERVER       | 13|加拿大
    RTC_LON_SERVER      | 14| 伦敦
    RTC_FRA_SERVER      | 15|法兰克福
    RTC_DXB_SERVER      | 16| 迪拜
    RTC_EXT_SERVER      |10000|使用扩展服务器
    RTC_DEFAULT_SERVER  |10001|缺省服务器


## YouMeKickReason类型定义
枚举常量|值|含义
----|------|-----
YOUME_KICK_ADMIN    |1     |管理员踢人
YOUME_KICK_RELOGIN  |2     | 多端登录被踢

## YouMeBroadcast房间内的广播消息
枚举常量|值|含义
----|------|-----
YOUME_BROADCAST_NONE    |0|
YOUME_BROADCAST_GRABMIC_BROADCAST_GETMIC  |1| 有人抢到了麦
YOUME_BROADCAST_GRABMIC_BROADCAST_FREEMIC    |2|有人释放了麦
YOUME_BROADCAST_INVITEMIC_BROADCAST_CONNECT  |3| A和B正在连麦
YOUME_BROADCAST_INVITEMIC_BROADCAST_DISCONNECT    |4|A和B取消了连麦


## YOUME_LOG_LEVEL类型定义
枚举常量|值|含义
----|------|-----
LOG_DISABLED    | 0|禁用日志
LOG_FATAL       | 1|严重错误
LOG_ERROR       | 10|错误
LOG_WARNING     | 20|警告
LOG_INFO        | 40|信息
LOG_DEBUG       | 50|调试
LOG_VERBOSE     | 60|所有日志

## YOUME_SAMPLE_RATE类型定义
枚举常量|值
----|------
SAMPLE_RATE_8  | 8000
SAMPLE_RATE_16 | 16000
SAMPLE_RATE_24 | 24000
SAMPLE_RATE_32 | 32000
SAMPLE_RATE_44 | 44100
SAMPLE_RATE_48 | 48000

## YOUME_AUDIO_QUALITY类型定义
枚举常量|值
----|------
LOW_QUALITY  | 0
HIGH_QUALITY | 1

## YouMeVideoRenderMode类型定义
枚举常量|值|含义
----|------|-----
SCALE_ASPECT_FILL  | 0| 填充
SCALE_ASPECT_FIT | 1| 裁剪
SCALE_ASPECT_BALANCED | 2| 自适应

## YOUME_VIDEO_FMT类型定义
枚举常量|值
----|------
VIDEO_FMT_RGBA32 |0
VIDEO_FMT_BGRA32 |1
VIDEO_FMT_YUV420P |2
VIDEO_FMT_NV21 |3
VIDEO_FMT_YV12 |4
VIDEO_FMT_CVPIXELBUFFER |5
VIDEO_FMT_TEXTURE |6
VIDEO_FMT_TEXTURE_OES |7

## YouMeUserRole类型定义

枚举常量|值|含义
----|------|-----
YOUME_USER_NONE | 0|非法用户，调用API时不能传此参数
YOUME_USER_TALKER_FREE|1|      自由讲话者，适用于小组通话（建议小组成员数最多10个），每个人都可以随时讲话, 同一个时刻只能在一个语音频道里面
YOUME_USER_TALKER_ON_DEMAND|2 |需要通过抢麦等请求麦克风权限之后才可以讲话，适用于较大的组或工会等（比如几十个人），同一个时刻只能有一个或几个人能讲话, 同一个时刻只能在一个语音频道里面
YOUME_USER_LISTENER|3         | 听众，主播/指挥/嘉宾的听众，同一个时刻只能在一个语音频道里面，只听不讲
YOUME_USER_COMMANDER|4        |指挥，国家/帮派等的指挥官，同一个时刻只能在一个语音频道里面，可以随时讲话，可以播放背景音乐，戴耳机情况下可以监听自己语音
YOUME_USER_HOST|5            |主播，广播型语音频道的主持人，同一个时刻只能在一个语音频道里面，可以随时讲话，可以播放背景音乐，戴耳机情况下可以监听自己语音
YOUME_USER_GUSET|6            |嘉宾，主播或指挥邀请的连麦嘉宾，同一个时刻只能在一个语音频道里面， 可以随时讲话


