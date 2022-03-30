## 组件介绍

TUICalling 是一个开源的音视频 UI 组件，通过在项目中集成 TUICalling 组件，您只需要编写几行代码就可以为您的 App 添加“一对一音视频通话”，“多人音视频通话”等场景，并且支持离线唤起能力。TUICalling 包含 Android、iOS、Web、小程序、Flutter、UniApp 等平台的源代码，基本功能如下图所示：

<table class="tablestyle">
<tbody><tr>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/c792c6fa94e4c4dd3151003b0f28eab7.png" width="250"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/a75f7d5f2911f2e21d3e4b3b9dfc4db5.png" width="500"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/3bb1a748b590518e2be09326cc30dc0b.png" width="250"></td>
</tr>
</tbody></table>


## 二. 组件集成

### 步骤一：下载并导入 TUICalling 组件

点击进入 [Github](https://github.com/tencentyun/TUICalling) ，选择克隆/下载代码，然后拷贝 Android/tuicalling 和 Android/TUICore 目录到您的工程中，并完成如下导入动作：
- 在 `setting.gradle` 中完成导入，参考如下：
```
include ':tuicalling'
include ':TUICore'
```

- 在 app 的 build.gradle 文件中添加对 tuicalling 的依赖：
```
api project(':tuicalling')
```

### 步骤二：配置权限及混淆规则

在 AndroidManifest.xml 中配置 App 的权限，SDK 需要以下权限（6.0以上的 Android 系统需要动态申请相机、麦克风、读取存储权限等）：

```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />        // 使用场景：悬浮窗、应用在后台时拉起通话界面时需要此权限；
<uses-permission android:name="android.permission.INTERNET" />              
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.BLUETOOTH" />                  // 使用场景：使用蓝牙耳机时需要此权限；
<uses-permission android:name="android.permission.READ_PHONE_STATE" />           // 使用场景：判断是否是系统来电打断时需要此权限；
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera"/>
<uses-feature android:name="android.hardware.camera.autofocus" />
```

在 proguard-rules.pro 文件，将 SDK 相关类加入不混淆名单：

```
-keep class com.tencent.** { *; }
```

### 步骤三：创建并初始化 TUI 组件库

```java
  // 1.组件登录，
  TUILogin.init(this, "您的SDKAppId", config, new V2TIMSDKListener() {
            @Override
            public void onKickedOffline() {  // 登录被踢下线通知（示例：账号在其他设备登录）
            }
            @Override
            public void onUserSigExpired() { // userSig过期通知
            }
  });
  TUILogin.login("您的userId", "您的userSig", new V2TIMCallback() {
            @Override
            public void onError(int code, String msg) {
                Log.d(TAG, "code: " + code + " msg:" + msg);
            }
            @Override
            public void onSuccess() {
            }
  });
  
  // 2.初始化TUICalling实例
  TUICalling callingImpl = TUICallingImpl.sharedInstance(context);
  
```
#### 3.1 参数说明
- **SDKAppID**：**TRTC 应用ID**，如果您未开通腾讯云 TRTC 服务，可以点击 [这里](https://console.cloud.tencent.com/trtc/app)，进入腾讯云实时音视频控制台，创建一个新的的TRTC应用后，点击应用信息，SDKAppID信息如下图所示；
 <img src="https://liteav.sdk.qcloud.com/app/doc/app_manager_sdk_secretkey.png" width="700">
	
- **Secretkey**：**TRTC 应用密钥**，和SDKAppId对应，进入 [TRTC 应用管理](https://console.cloud.tencent.com/trtc/app) 后，SecretKey信息如上图所示；

- **userId**：当前用户的 ID，字符串类型，长度不超过32字节，不支持使用特殊字符，建议使用英文或数字，可结合业务实际账号体系自行设置。

- **userSig**：根据SDKAppId、userId，Secretkey等信息计算得到的安全保护签名，您可以点击 [这里](https://console.cloud.tencent.com/trtc/usersigtool) 直接在线生成一个调试的userSig，也可以参照我们的[TUICalling示例工程](https://github.com/tencentyun/TUICalling/blob/main/Android/App/src/main/java/com/tencent/liteav/demo/LoginActivity.java#L74)自行计算，更多信息见 [如何计算及使用 UserSig](https://cloud.tencent.com/document/product/647/17275)。


### 步骤四：实现音视频通话

#### 4.1. 实现1对1视频通话/音频通话
```java
// 发起1对1视频通话，假设userId为：1111；
callingImpl.call(["1111"], TUICalling.Type.VIDEO);
```

#### 4.2. 实现多人视频通话：
```java
// 发起多人视频通话，假设userId分别为：1111、2222、3333；
callingImpl.call(["1111", "2222", "3333"], TUICalling.Type.VIDEO);
```

>? 当接收方完成步骤三后，即登录成功后，再收到通话请求后，TUICalling组件会自动启动相应的接听界面。    
如果您想发起语音通话，更改类型为`TUICalling.Type.AUDIO`即可。

### 步骤五：实现离线唤醒（可选）

完成以上四个步骤，就可以实现音视频通话的拨打和接通，但如果您的业务场景需要在`App 的进程被杀死后`或者`APP 退到后台后`，还可以正常接收到音视频通话请求，就需要为 TUICalling 组件增加离线推送功能，详情见 [**Android离线推送接入指引.md**](https://github.com/tencentyun/TUICalling/blob/main/Android/Android%E7%A6%BB%E7%BA%BF%E6%8E%A8%E9%80%81%E6%8E%A5%E5%85%A5%E6%8C%87%E5%BC%95.md)。

### 步骤六：监听通话状态（可选）
如果您的业务需要监听通话的状态，例如通话开始、结束等，可以监听以下事件：
```
callingImpl.setCallingListener(new TUICalling.TUICallingListener() {
    @Override
    public boolean shouldShowOnCallView() {
        return false;
    }

    @Override
    public void onCallStart(String[] userIDs, TUICalling.Type type, TUICalling.Role role, View tuiCallingView) {

    }

    @Override
    public void onCallEnd(String[] userIDs, TUICalling.Type type, TUICalling.Role role, long totalTime) {

    }

    @Override
    public void onCallEvent(TUICalling.Event event, TUICalling.Type type, TUICalling.Role role, String message) {
        Log.d(TAG, "onCallEvent: event = " + event + " ,message = " + message);
    }
  });
```

### 步骤七：开启悬浮窗功能（可选）
如果您的业务需要开启悬浮窗功能，您可以在TUICalling组件初始化时调用`callingImpl.enableFloatWindow(true)`开启该功能；
  
目前组件支持以下两种情况悬浮窗：
  - 系统悬浮窗(点击home键退到后台)：需开启悬浮窗权限
  - 应用内悬浮窗(最小化退到上一层界面)：需开启悬浮窗权限，并设置要返回的上一层界面

开启悬浮窗权限方法：打开手机设置，找到应用管理，找到您的应用，点击权限，点击悬浮窗并允许(手机厂商、平台不同，该位置可能有差异)。
>? 没有权限会进行提示。

设置要返回的上一层界面，在`AndroidManifest.xml`为您需要跳转的界面配置跳转动作 `com.tencent.trtc.tuicalling`，例如：
```
 <activity
    android:name="{packageName}.MainActivity"
    android:launchMode="singleTop">
    <intent-filter>
        <action android:name="com.tencent.trtc.tuicalling" /> 
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

>? 通话中不支持直接返回。

## 三. 常见问题

**Q: TUICalling 组件支持自定义头像和昵称吗？**

A: 支持，调用`setUserNickname/setUserAvatar`即可；

**Q: TUICalling 组件支持自定义铃声吗？**

A: 支持，调用`setCallingBell`即可；

>? 更多帮助信息，详见 [TUI 场景化解决方案常见问题](https://cloud.tencent.com/developer/article/1952880)，欢迎加入 QQ 群：592465424，进行技术交流和反馈~
