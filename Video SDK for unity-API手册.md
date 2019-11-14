# Video SDK for Unity3D API接口手册

### 相关异步/同步处理方法介绍
Unity3D版本的游密实时音视频引擎SDK提供的全部为C#接口,接口调用都会立即返回,凡是本身需要较长耗时的接口调用都会采用异步回调的方式,所有接口都可以在主线程中直接使用。回调都在子线程中进行，请注意不要在回调中直接操作UI主线程。

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
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	// YOUME_EVENT_INIT_OK  - 表明初始化成功
	// YOUME_EVENT_INIT_FAILED - 表明初始化失败，最常见的失败原因是网络错误或者 AppKey-AppSecret 错误
	void OnEvent (string strParam);
```

## 判断是否初始化完成

* **语法**
```
bool IsInited()
```

* **功能**
判断是否初始化完成

* **返回值**
true——初始化完成，false——初始化未完成。


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
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	//YOUME_EVENT_JOIN_OK - 成功进入语音频道
	//YOUME_EVENT_JOIN_FAILED - 进入语音频道失败，可能原因是网络或服务器有问题
	void OnEvent (string strParam);
```

### 加入语音频道（多频道）
* **语法**

```

	YouMeErrorCode JoinChannelMultiMode (string strUserID, string strChannelID, YouMeUserRole userRole)
```
* **功能**
加入语音频道（多频道模式，可以同时听多个语音频道的内容，但每个时刻只能对着一个频道讲话）。

* **参数说明**
`strUserID`：全局唯一的用户标识，全局指在当前应用程序的范围内。
`strChannelID`：全局唯一的频道标识，全局指在当前应用程序的范围内。
`userRole`：用户在语音频道里面的角色，见YouMeUserRole定义。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	//YOUME_EVENT_JOIN_OK - 成功进入语音频道
	//YOUME_EVENT_JOIN_FAILED - 进入语音频道失败，可能原因是网络或服务器有问题
	void OnEvent (string strParam);
```

### 指定讲话频道
* **语法**

```

	YouMeErrorCode SpeakToChannel (string strChannelID);
```
* **功能**
多频道模式下，指定当前要讲话的频道。

* **参数说明**
`strChannelID`：全局唯一的频道标识，全局指在当前应用程序的范围内。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	//YOUME_EVENT_SPEAK_SUCCESS - 成功切入到指定语音频道
	//YOUME_EVENT_SPEAK_FAILED - 切入指定语音频道失败，可能原因是网络或服务器有问题
	void OnEvent (string strParam);
```


### 退出指定的语音频道
* **语法**

```
YouMeErrorCode LeaveChannelMultiMode (string strChannelID);
```
* **功能**
多频道模式下，退出指定的语音频道。

* **参数说明**
`strChannelID`：全局唯一的频道标识，全局指在当前应用程序的范围内。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```

	//涉及到的主要回调事件有：
	//YOUME_EVENT_LEAVED_ONE - 成功退出指定语音频道
	void OnEvent (string strParam);
```

### 退出所有语音频道
* **语法**

```
YouMeErrorCode LeaveChannelAll ();
```
* **功能**
退出所有的语音频道（单频道模式下直接调用此函数离开频道即可）。

* **参数说明**
无

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
//YOUME_EVENT_LEAVED_ALL - 成功退出所有语音频道
void OnEvent (string strParam);
```


### 切换语音输出设备
* **语法**

```
	YouMeErrorCode SetOutputToSpeaker (bool bOutputToSpeaker);
```
* **功能**
默认输出到扬声器，在加入房间成功后设置（iOS受系统限制，如果已释放麦克风则无法切换到听筒）

* **参数说明**
`bOutputToSpeaker`:true为扬声器，false为听筒。


### 设置扬声器状态
* **语法**

```
	void SetSpeakerMute (bool mute);
```
* **功能**
打开/关闭扬声器。建议该状态值在加入房间成功后按需再重置一次。

* **参数说明**
`mute`:true——静音，false——没有静音。


### 获取扬声器静音状态
* **语法**

```
	bool GetSpeakerMute();
```

* **功能**
获取扬声器静音状态

* **返回值**
true——静音，false——没有静音。



### 设置麦克风静音
* **语法**

``
	void SetMicrophoneMute (bool mute);
```

* **功能**
打开／关闭麦克风。建议该状态值在加入房间成功后按需再重置一次。

* **参数说明**
`mute`:true——静音，false——取消静音。


### 获取麦克风静音状态
* **语法**

```
	bool GetMicrophoneMute ();
```

* **功能**
获取麦克风静音状态

* **返回值**
true——静音，false——没有静音。


### 设置麦克风音量回调参数
* **语法**

```
	YouMeErrorCode SetMicLevelCallback (int maxMicLevel);
```

* **功能**
可以在初始化成功后随时调用这个接口。在整个APP生命周期只需要调用一次，除非想修改参数。
设置成功后，当用户讲话时，将收到回调事件 MY_MIC_LEVEL， 回调参数 iStatus 表示当前讲话的音量级别。

* **参数说明**
`maxMicLevel`:设为 0 表示关闭麦克风音量回调,设为 大于0的值表示音量最大时对应的值，可以根据UI设计来设定。比如用10级的音量条来表示音量变化，则传10。这样当底层回传音量是10时，则表示当前mic音量达到最大值。

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置是否开启远端语音音量回调
* **语法**

```
	YouMeErrorCode SetFarendVoiceLevelCallback(int maxLevel)
```

* **功能**
设置是否开启远端语音音量回调, 并设置相应的参数

* **参数说明**
`maxLevel`:音量最大时对应的级别，最大可设100。比如你只在UI上呈现10级的音量变化，那就设10就可以了。设 0 表示关闭回调。

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置是否通知别人麦克风和扬声器的开关
* **语法**

```
	void SetAutoSendStatus( bool bAutoSend );
```

* **功能**
设置是否通知别人,自己麦克风和扬声器的开关状态

### 设置音量
* **语法**

```
	void SetVolume (uint uiVolume);
```

* **功能**
设置当前程序输出音量大小。建议该状态值在加入房间成功后按需再重置一次。

* **参数说明**
`uiVolume`:当前音量大小，范围[0-100]。

### 获取音量
* **语法**

```
	int GetVolume ();
```

* **功能**
获取当前程序输出音量大小。

* **返回值**
当前音量大小，范围[0-100]。


### 设置是否允许使用移动网络
* **语法**

```
	void SetUseMobileNetworkEnabled (bool bEnabled);
```

* **功能**
设置是否允许使用移动网络。在WIFI和移动网络都可用的情况下会优先使用WIFI，在没有WIFI的情况下，如果设置允许使用移动网络，那么会使用移动网络进行语音通信，否则通信会失败。


* **参数说明**
`bEnabled`:true——允许使用移动网络，false——禁止使用移动网络。


### 获取是否允许使用移动网络
* **语法**

```
	bool GetUseMobileNetworkEnabled () ;
```

* **功能**
获取是否允许SDK在没有WIFI的情况使用移动网络进行语音通信。

* **返回值**
true——允许使用移动网络，false——禁止使用移动网络，默认情况下允许使用移动网络。


### 设置当麦克风静音时，是否释放麦克风设备
* **语法**

```
	YouMeErrorCode SetReleaseMicWhenMute (bool enabled);
```

* **功能**
设置当麦克风静音时，是否释放麦克风设备

* **参数说明**
`enabled`：true 当麦克风静音时，释放麦克风设备，此时允许第三方模块使用麦克风设备录音。在Android上，语音通过媒体音轨，而不是通话音轨输出。
false 不管麦克风是否静音，麦克风设备都会被占用。

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

## 设置插入耳机时，是否自动退出系统通话模式

* **语法**
```
YouMeErrorCode SetExitCommModeWhenHeadsetPlugin (bool enabled);
```

* **功能**
设置插入耳机时，是否自动退出系统通话模式(禁用手机硬件提供的回声消除等信号前处理)
系统提供的前处理效果包括回声消除、自动增益等，有助于抑制背景音乐等回声噪音，减少系统资源消耗
由于插入耳机可从物理上阻断回声产生，故可设置禁用该效果以保留背景音乐的原生音质效果
注：Windows和macOS不支持该接口

* **参数说明**
`enabled`： true--当插入耳机时，自动禁用系统硬件信号前处理，拔出时还原；false--插拔耳机不做处理。

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 抢麦相关设置
* **语法**

```
YouMeErrorCode SetGrabMicOption (string pChannelID, int mode, int maxAllowCount, int maxTalkTime, uint voteTime);
```

* **功能**
抢麦相关设置（抢麦活动发起前调用此接口进行设置）

* **参数说明**
`pChannelID`：抢麦活动的频道id
`mode`：抢麦模式（1:先到先得模式；2:按权重分配模式）
`maxAllowCount`：允许能抢到麦的最大人数
`maxTalkTime`：允许抢到麦后使用麦的最大时间（秒）
`voteTime`：抢麦仲裁时间（秒），过了X秒后服务器将进行仲裁谁最终获得麦（仅在按权重分配模式下有效）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 发起抢麦活动
* **语法**

```
YouMeErrorCode StartGrabMicAction (string pChannelID, string pContent);
```

* **功能**
发起抢麦活动

* **参数说明**
`pChannelID`：抢麦活动的频道id
`pContent`：游戏传入的上下文内容，通知回调会传回此内容（目前只支持纯文本格式）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 停止抢麦活动
* **语法**

```
YouMeErrorCode StopGrabMicAction (string pChannelID, string pContent);
```

* **功能**
停止抢麦活动

* **参数说明**
`pChannelID`：抢麦活动的频道id
`pContent`：游戏传入的上下文内容，通知回调会传回此内容（目前只支持纯文本格式）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 发起抢麦请求
* **语法**

```
	YouMeErrorCode requestGrabMic (string pChannelID, int score, bool isAutoOpenMic, string pContent);
```

* **功能**
发起抢麦请求

* **参数说明**
`pChannelID`：抢麦的频道id
`score`：积分（权重分配模式下有效，游戏根据自己实际情况设置）
`isAutoOpenMic`：抢麦成功后是否自动开启麦克风权限
`pContent`：游戏传入的上下文内容，通知回调会传回此内容（目前只支持纯文本格式）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 释放抢到的麦
* **语法**

```
YouMeErrorCode releaseGrabMic (string pChannelID);
```

* **功能**
释放抢到的麦

* **参数说明**
`pChannelID`：抢麦的频道id

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 连麦相关设置
* **语法**

```
YouMeErrorCode setInviteMicOption (string pChannelID, int waitTimeout, int maxTalkTime);
```

* **功能**
连麦相关设置（角色是频道的管理者或者主播时调用此接口进行频道内的连麦设置）

* **参数说明**
`pChannelID`：连麦的频道id
`waitTimeout`：等待对方响应超时时间（秒）
`maxTalkTime`：最大通话时间（秒）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 发起与某人的连麦请求
* **语法**

```
YouMeErrorCode requestInviteMic (string pChannelID, string pUserID, string pContent);
```

* **功能**
发起与某人的连麦请求（主动呼叫）

* **参数说明**
`pChannelID`：连麦的频道id
`pUserID`：被叫方的用户id
`pContent`：游戏传入的上下文内容，通知回调会传回此内容（目前只支持纯文本格式）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 对连麦请求做出回应
* **语法**

```
YouMeErrorCode responseInviteMic (string pUserID, bool isAccept, string pContent);
```

* **功能**
对连麦请求做出回应（被动应答）

* **参数说明**
`pUserID`：主叫方的用户id
`isAccept`：是否同意连麦
`pContent`：游戏传入的上下文内容，通知回调会传回此内容（目前只支持纯文本格式）

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 停止连麦
* **语法**

```
YouMeErrorCode stopInviteMic ();
```

* **功能**
停止连麦

* **参数说明**
无

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 控制他人麦克风
* **语法**

```
YouMeErrorCode SetOtherMicMute (string userID,bool mute);
```

* **功能**
控制他人的麦克风状态

* **参数说明**
`userID`：要控制的用户ID
`mute`：是否静音。true:静音别人的麦克风，false：开启别人的麦克风

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 控制他人扬声器
* **语法**

```
YouMeErrorCode SetOtherSpeakerMute (string userID,bool mute);
```

* **功能**
控制其他人的扬声器开关

* **参数说明**
`userID`：要控制的用户ID
`mute`：是否静音。true:静音别人的扬声器，false：取消静音别人的扬声器

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 向房间广播消息
* **语法**

```
YouMeErrorCode SendMessage (string channelID, string content, ref int  requestID);
```

* **功能**
向房间广播消息

* **参数说明**
`channelID`：广播房间
`content`：广播内容-文本串
`requestID`：返回消息标识，回调的时候会回传该值

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 把某人踢出房间
* **语法**

```
YouMeErrorCode KickOtherFromChannel (string userID , string channelID , int lastTime);
```

* **功能**
把某人踢出房间

* **参数说明**
`userID`：被踢的用户ID
`channelID`：从哪个房间踢出-文本串
`lastTime`：踢出后，多长时间内不允许再次进入

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置是否听某人的语音

* **语法**

```
YouMeErrorCode SetListenOtherVoice (string userID,bool isOn);
```

* **功能**
设置是否听某人的语音。

* **参数说明**
`userID`：被控制的用户ID。
`isOn`：true表示开启接收指定用户的语音，false表示屏蔽指定用户的语音。

* **返回值**
如果成功返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 暂停通话
* **语法**

```
YouMeErrorCode PauseChannel();
```

* **功能**
暂停通话，释放对麦克风等设备资源的占用。当需要用第三方模块临时录音时，可调用这个接口。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
//主要回调事件：
//YOUME_EVENT_PAUSED - 暂停语音频道完成
void OnEvent (string strParam);
```

### 恢复通话
* **语法**

```
YouMeErrorCode ResumeChannel();
```

* **功能**
恢复通话，调用PauseChannel暂停通话后，可调用这个接口恢复通话。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
//主要回调事件：
//YOUME_EVENT_RESUMED - 恢复语音频道完成
void OnEvent (string strParam);
```

### 设置语音检测
* **语法**

```
YouMeErrorCode SetVadCallbackEnabled(bool enabled);
```

* **功能**
设置是否开启语音检测回调。开启后频道内有人正在讲话与结束讲话都会发起相应回调通知。

* **参数说明**
`bEnabled`:true——打开，false——关闭。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 播放背景音乐
* **语法**

```
YouMeErrorCode PlayBackgroundMusic (string pFilePath, bool bRepeat);
```

* **功能**
播放指定的音乐文件。播放的音乐将会通过扬声器输出，并和语音混合后发送给接收方。这个功能适合于主播/指挥等使用。

* **参数说明**
`pFilePath`：音乐文件的路径。
`bRepeat`：是否重复播放，true——重复播放，false——只播放一次就停止播放。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
	//主要回调事件：
	//YOUME_EVENT_BGM_STOPPED - 通知背景音乐播放结束
	//YOUME_EVENT_BGM_FAILED - 通知背景音乐播放失败
	void OnEvent(string strParam);
```


### 暂停播放当前正在播放的背景音乐
* **语法**

```
	YouMeErrorCode PauseBackgroundMusic();
```

* **功能**
暂停播放当前正在播放的背景音乐。
这是一个同步调用接口，函数返回时，音乐播放也就停止了。

* **返回值**
如果成功返回YOUME_SUCCESS，表明成功暂停了音乐播放流程；否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 恢复当前暂停播放的背景音乐。
* **语法**

```
	YouMeErrorCode ResumeBackgroundMusic();
```

* **功能**
恢复播放当前暂停播放的背景音乐。
这是一个同步调用接口，函数返回时，音乐播放也就停止了。

* **返回值**
如果成功返回YOUME_SUCCESS，表明成功停止了音乐播放流程；否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 停止播放背景音乐
* **语法**

```
	YouMeErrorCode StopBackgroundMusic();
```

* **功能**
停止播放当前正在播放的背景音乐。
这是一个同步调用接口，函数返回时，音乐播放也就停止了。

* **返回值**
如果成功返回YOUME_SUCCESS，表明成功停止了音乐播放流程；否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置背景音乐播放音量
* **语法**

```
	YouMeErrorCode SetBackgroundMusicVolume(int vol);
```

* **功能**
设定背景音乐的音量。这个接口用于调整背景音乐和语音之间的相对音量，使得背景音乐和语音混合听起来协调。
这是一个同步调用接口。

* **参数说明**
`vol`:背景音乐的音量，范围 [0-100]。

* **返回值**
如果成功（表明成功设置了背景音乐的音量）返回YOUME_SUCCESS，否则返回错误码，具体请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置监听
* **语法**

```
	YouMeErrorCode SetHeadsetMonitorOn(bool micEnabled, bool bgmEnabled = true);
```

* **功能**
设置是否用耳机监听自己的声音，当不插耳机或外部输入模式时，这个设置不起作用
这是一个同步调用接口。

* **参数说明**
`micEnabled`:是否监听麦克风 true 监听，false 不监听。
`bgmEnabled`:是否监听背景音乐 true 监听，false 不监听。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置混响音效
* **语法**

```
	YouMeErrorCode SetReverbEnabled(bool enabled);
```

* **功能**
设置是否开启混响音效，这个主要对主播/指挥有用。

* **参数说明**
`bEnabled`:true——打开，false——关闭。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置录音时间戳
* **语法**

```
	void SetRecordingTimeMs(uint timeMs);
```

* **功能**
设置当前录音的时间戳。当通过录游戏脚本进行直播时，要保证观众端音画同步，在主播端需要进行时间对齐。
这个接口设置的就是当前游戏画面录制已经进行到哪个时间点了。

* **参数说明**
`timeMs`:当前游戏画面对应的时间点，单位为毫秒。

* **返回值**
无。


### 设置播放时间戳
* **语法**

```
 	void SetPlayingTimeMs(uint timeMs);
```

* **功能**
设置当前声音播放的时间戳。当通过录游戏脚本进行直播时，要保证观众端音画同步，游戏画面的播放需要和声音播放进行时间对齐。
这个接口设置的就是当前游戏画面播放已经进行到哪个时间点了。

* **参数说明**
`timeMs`:当前游戏画面播放对应的时间点，单位为毫秒。

* **返回值**
无。

### 设置用户自定义Log路径
* **语法**

```
 	YouMeErrorCode SetUserLogPath(string strFilePath);
```

* **功能**
设置用户自定义Log路径

* **参数说明**
`strFilePath`:文件的路径

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置日志等级
* **语法**

```
 void SetLogLevel(YOUME_LOG_LEVEL consoleLevel, YOUME_LOG_LEVEL  fileLevel);
```

* **功能**
设置日志等级

* **参数说明**
`consoleLevel`:控制台日志等级
`fileLevel`:文件日志等级

* **返回值**
无


### 设置是否开启视频编码器
* **语法**

```
 	YouMeErrorCode OpenVideoEncoder(string pFilePath);
```

* **功能**
设置是否开启视频编码器

* **参数说明**
`pFilePath`:yuv文件的绝对路径

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 创建渲染
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

### 删除渲染
* **语法**

```
 	YouMeErrorCode deleteRender(int renderId);
```

* **功能**
删除渲染

* **参数说明**
`renderId`:renderId

* **返回值**
等于0 - success 小于0 - 具体错误码


### 获取SDK版本号
* **语法**

```
 int GetSDKVersion();
```

* **功能**
获取SDK版本号，版本号分为4段，如 2.5.0.0，这4段在int里面的分布如下
 | 4 bits | 6 bits | 8 bits | 14 bits|

* **参数说明**
无。

* **返回值**
压缩过的版本号。


### 设置服务器区域
* **语法**

```
void SetServerRegion(YOUME_RTC_SERVER_REGION regionId, string strExtRegionName);
```

* **功能**
设置首选连接服务器的区域码.

* **参数说明**
`serverRegionId`：如果YOUME_RTC_SERVER_REGION定义的区域码不能满足要求，可以把这个参数设为 RTC_EXT_SERVER，然后通过后面的参数strExtServerRegionName 设置一个自定的区域值（如中国用 "cn" 或者 “ch"表示），然后把这个自定义的区域值同步给游密，我们将通过后台配置映射到最佳区域的服务器。
`strExtServerRegionName`：自定义的扩展的服务器区域名。不能为null，可为空字符串“”。只有前一个参数serverRegionId设为RTC_EXT_SERVER时，此参数才有效（否则都将当空字符串“”处理）。


### 设置服务器区域（全）
* **语法**

```
void SetServerRegion(string[] regionNames);
```

* **功能**
设置参与通话各方所在的区域,这个接口适合于分布区域比较广的应用。最简单的做法是只设定前用户所在区域。但如果能确定其他参与通话的应用所在的区域，则能使服务器选择更优。

* **参数说明**
`regionNames`：指定参与通话各方区域的数组，数组里每个元素为一个区域代码。用户可以自行定义代表各区域的字符串（如中国用 "cn" 或者 “ch"表示），然后把定义好的区域表同步给游密，游密会把这些定义配置到后台，在实际运营时选择最优服务器。

###  RestApi——支持主播相关信息查询
* **语法**

```
YouMeErrorCode  RequestRestApi( string command, string queryBody, ref int  requestID )
```

* **功能**
Rest API , 向服务器请求额外数据。支持主播信息，主播排班等功能查询。详情参看文档<RequestRestAPI接口说明>


* **参数说明**
`command`：请求的命令字符串，标识命令类型。
`queryBody`：请求需要的参数,json格式。
`requestID`：回传id,回调的时候传回，标识消息。不关心可以填NULL。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
void OnRequestRestApi(string strResult )
```

###  安全验证码设置

* **语法**

```
  void  SetToken( string strToken  )
```

* **功能**
设置身份验证的token，需要配合后台接口。

* **参数说明**
`strToken`：身份验证用token，设置为空字符串，清空token值，不进行身份验证。

###  查询频道用户列表
* **语法**

```
YouMeErrorCode  GetChannelUserList( string channelID,  int maxCount, bool notifyMemChange  );
```

* **功能**
查询频道当前的用户列表， 并设置是否获取频道用户进出的通知。（必须自己在频道中）


* **参数说明**
`channelID`：频道ID。
`maxCount`：想要获取的最大人数。-1表示获取全部列表。
`notifyMemChange`：当有人进出频道时，是否获得通知。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

* **异步回调**

```
void OnMemberChange( string strResult );
```



## 视频相关接口
视频的频道属性和语音是绑定的，可以单独控制是否开启/关闭音视频流。**以下接口的调用，必须是在进入频道之后。**

### 开启自己的视频

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

### 接收其他用户的视频
在收到 `OnEvent` 通知后，判断如果是 `YouMe.YouMeEvent.YOUME_EVENT_OTHERS_VIDEO_ON` 事件，接口调用如下代码接收视频：

``` c#
    public void BindRecivedUserVideo(string userId){
		// 根据用户id获取渲染id
		int videoRenderID = YouMeTexture.GetInstance ().CreateTexture (userId, gameObject.name);
		//获取显示对象
         YouMeTexture.GetInstance().SetVideoRenderUpdateCallback(videoRenderID, (Texture2D videoTexture)=>{
            // 把 videoTexture 这个Texture2D对象显示到游戏里即可
         });
	}
```

### 关闭摄像头
* **语法** *

``` c#
    YouMeErrorCode StopCapture();
```

* **功能**
关闭摄像头，关闭后仍然留在聊天频道，只是不再输出视频流。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 暂停渲染通知
* **语法** *

``` c#
    //注意，这个视频渲染类 YouMeTexture 的接口
    YouMeTexture.GetInstance().PauseVideoRender(string userid);
```

* **功能**
调用后，`SetVideoRenderUpdateCallback` 设置的回调就暂停通知，不会更新。

* **返回值**
bool，表示操作是否成功。

### 恢复渲染通知
* **语法** *

``` c#
    //注意，这个视频渲染类 YouMeTexture 的接口
    YouMeTexture.GetInstance().ResumeVideoRender(string userid);
```

* **功能**
调用后，`SetVideoRenderUpdateCallback` 设置的回调恢复正常通知（目前是每帧通知更新）。

* **返回值**
bool，表示操作是否成功。

### 释放视频渲染资源
* **语法** *

``` c#
    //注意，这个视频渲染类 YouMeTexture 的接口
    YouMeTexture.GetInstance().DeleteRender(string userid);
```

* **功能**
调用后，该用户绑定过的视频相关渲染组建就会被移除。如果删除后，又收到该用户的视频流，就会再通知 `YouMe.YouMeEvent.YOUME_EVENT_OTHERS_VIDEO_ON` 。

* **返回值**
bool，表示操作是否成功。


### 反初始化

* **语法**

``` c#
YouMeErrorCode UnInit ();
```

* **功能**
反初始化引擎，可在退出游戏时调用，以释放SDK所有资源。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置视频回调
* **语法**

```
 	YouMeErrorCode SetVideoCallback();
```

* **功能**
设置视频回调

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 开始camera capture
* **语法**

```
 	YouMeErrorCode StartCapture();
```

* **功能**
开始camera capture

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 停止camera capture
* **语法**

```
 	YouMeErrorCode StopCapture();
```

* **功能**
停止camera capture

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置本地视频渲染回调的分辨率
* **语法**

```
 	YouMeErrorCode SetVideoLocalResolution(int width, int height);
```

* **功能**
设置本地视频渲染回调的分辨率

* **参数说明**
`width`:宽
`height`:高

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置是否前置摄像头
* **语法**

```
 	YouMeErrorCode SetCaptureFrontCameraEnable(bool enable);
```

* **功能**
设置是否前置摄像头

* **参数说明**
`enable`:是否前置摄像头

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 获取视频数据
* **语法**

```
 int GetVideoFrame(int renderId, ref I420Frame frame);
```

* **功能**
获取视频数据

* **参数说明**
`renderId`:renderId
`frame`:frame

* **返回值**
成功返回len,否则返回0

### 设置是否是测试模式
* **语法**

```
 void SetTestConfig(int bTest);
```

* **功能**
是否是测试模式,测试模式使用测试服

* **参数说明**
`bTest`:是否是测试模式

* **返回值**
无

### 获取是否开启变声
* **语法**

```
 	bool GetSoundtouchEnabled();
```

* **功能**
获取是否开启变声

* **参数说明**
无

* **返回值**
是表示开启，否表示不开启

### 设置是否开启变声
* **语法**

```
 void SetSoundtouchEnabled(bool bEnable);
```

* **功能**
设置是否开启变声

* **参数说明**
`bEnable`:是否开启变声

* **返回值**
无。

### 获取变速
* **语法**

```
 float GetSoundtouchTempo();
```

* **功能**
获取变速，1为正常值

* **参数说明**
无

* **返回值**
变速，1为正常值

### 设置变速
* **语法**

```
 	void SetSoundtouchTempo(float nTempo);
```

* **功能**
设置变速，1为正常值

* **参数说明**
`nTempo`:变速，1为正常值

* **返回值**
无

### 获取节拍
* **语法**

```
 float GetSoundtouchRate();
```

* **功能**
获取节拍，1为正常值

* **参数说明**
无

* **返回值**
节拍，1为正常值

### 设置节拍
* **语法**

```
 void SetSoundtouchRate(float nRate);
```

* **功能**
设置节拍，1为正常值

* **参数说明**
`nRate`:节拍，1为正常值

* **返回值**
无。

### 获取变调
* **语法**

```
 float GetSoundtouchPitch();
```

* **功能**
获取变调，1为正常值

* **参数说明**
无

* **返回值**
变调，1为正常值


### 设置变调
* **语法**

```
 YouMeErrorCode SetSoundtouchPitch(float nPitch);
```

* **功能**
设置变调，1为正常值

* **参数说明**
`nPitch`:变调，1为正常值

* **返回值**
无。


### 设置是否开启远端语音音量回调
* **语法**

```
 YouMeErrorCode SetFarendVoiceLevelCallback(int maxLevel);
```

* **功能**
设置是否开启远端语音音量回调, 并设置相应的参数

* **参数说明**
`maxLevel`:音量最大时对应的级别，最大可设100。比如你只在UI上呈现10级的音量变化，那就设10就可以了。设 0 表示关闭回调。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置服务器模式
* **语法**

```
 void SetServerMode(int mode);
```

* **功能**
设置服务器模式

* **参数说明**
`mode`:服务器模式

* **返回值**
无。


### 进入房间后，切换身份
* **语法**

```
 YouMeErrorCode SetUserRole(YouMeUserRole userRole);
```

* **功能**
进入房间后，切换身份

* **参数说明**
`userRole`:用户在语音频道里面的角色，见YouMeUserRole定义。

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 获取身份
* **语法**

```
 YouMeUserRole GetUserRole();
```

* **功能**
获取身份

* **参数说明**
无。

* **返回值**
用户在语音频道里面的角色，见YouMeUserRole定义。

### 获取背景音乐是否在播放
* **语法**

```
 bool IsBackgroundMusicPlaying();
```

* **功能**
获取背景音乐是否在播放

* **参数说明**
无。

* **返回值**
true表示在播放，false表示不在播放。


### 获取是否初始化成功
* **语法**

```
 bool IsInited();
```

* **功能**
获取是否初始化成功

* **参数说明**
无。

* **返回值**
true表示初始化成功，false表示还未完成。

### 获取是否在某个语音房间内
* **语法**

```
 bool IsInChannel(string pChannelID);
```

* **功能**
获取是否在某个语音房间内

* **参数说明**
`pChannelID`：要查询的频道id

* **返回值**
true表示在，false表示不在。


### 切换前后摄像头
* **语法**

```
 YouMeErrorCode SwitchCamera();
```

* **功能**
切换前后摄像头

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 权限检测结束后重置摄像头
* **语法**

```
 int ResetCamera();
```

* **功能**
权限检测结束后重置摄像头

* **参数说明**
无

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置视频网络传输过程分辨率
* **语法**

```
 int SetVideoNetResolution( int width, int height );
```

* **功能**
设置视频网络传输过程的分辨率,高分辨率

* **参数说明**
`width`:视频图像宽
`height`:视频图像高

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置音视频统计数据时间间隔
* **语法**

```
 void SetAVStatisticInterval( int interval );
```

* **功能**
设置音视频统计数据时间间隔

* **参数说明**
`interval`:时间间隔

* **返回值**
无。


### 设置Audio的传输质量
* **语法**

```
 void SetAudioQuality( YOUME_AUDIO_QUALITY quality );
```

* **功能**
设置Audio的传输质量

* **参数说明**
`quality`:传输质量0: low 1: high

* **返回值**
无。

### 设置视频数据上行的码率的上下限
* **语法**

```
 void SetVideoCodeBitrate( uint maxBitrate,  uint minBitrate );
```

* **功能**
设置视频数据上行的码率的上下限。

* **参数说明**
`maxBitrate`:最大码率，单位kbit/s.  0无效
`minBitrate`:最小码率，单位kbit/s.  0无效

* **返回值**
无。


### 获取视频数据上行的当前码率。
* **语法**

```
 uint GetCurrentVideoCodeBitrate();
```

* **功能**
获取视频数据上行的当前码率。

* **参数说明**
无

* **返回值**
视频数据上行的当前码率。


### 设置视频数据是否同意开启硬编硬解
* **语法**

```
 void SetVideoHardwareCodeEnable( bool bEnable );
```

* **功能**
设置视频数据是否同意开启硬编硬解

* **参数说明**
`bEnable`:true开启,false不开启

* **返回值**
无。


### 获取视频数据是否同意开启硬编硬解
* **语法**

```
 void GetVideoHardwareCodeEnable();
```

* **功能**
获取视频数据是否同意开启硬编硬解

* **参数说明**
无

* **返回值**
true同意，false不同意。


### 设置视频无帧渲染的等待超时时间
* **语法**

```
 void SetVideoNoFrameTimeout(int timeout);
```

* **功能**
设置视频无帧渲染的等待超时时间，超过这个时间会给上层回调

* **参数说明**
`timeout`:超时时间，单位为毫秒

* **返回值**
无。


### 查询多个用户视频信息
* **语法**

```
 int QueryUsersVideoInfo( string userList);
```

* **功能**
查询多个用户视频信息（支持分辨率）

* **参数说明**
`userList`:用户ID列表的json数组

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。


### 设置多个用户视频信息
* **语法**

```
 int QueryUsersVideoInfo( string videoinfoList );
```

* **功能**
设置多个用户视频信息（支持分辨率）

* **参数说明**
`videoinfoList`:用户对应分辨率列表的json数组

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

### 设置未定义视频帧
* **语法**
* 
youme_setVideoFrameRawCbEnabled(true);

* **功能**
设置未定义视频帧,安卓环境下 必须在进入频道前调用开启

* **参数说明**
`true`:开启，false：不开启

* **返回值**
如果成功则返回YOUME_SUCCESS，其它具体参见[YouMeErrorCode类型定义](/doc/TalkStatusCode.html#youmeerrorcode类型定义)。

