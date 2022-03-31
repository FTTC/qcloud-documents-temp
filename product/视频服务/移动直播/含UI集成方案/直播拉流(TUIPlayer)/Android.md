## TUIPlayer简介
`TUIPlayer` 组件是一套开源的、完整的视频解决方案，它基于腾讯云 `MLVB SDK` 和` IM SDK`，可以实现直播拉流，直播连麦等功能，通过 `TUIPlayer` 组件您可以轻松实现直播视频拉流，而无需自己实现复杂的UI与逻辑功能
<table>
<tr>
   <th>观看直播</th>
   <th>发起连麦</th>
 </tr>
<tr>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/9cbd0d20c12b86a87436fc72e0b4950d.jpg"/></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/4125d61586106e87a414d738848ad0c6.jpg"/></td>
</tr>
</table>

[](id:model)
## 二. 组件集成
[](id:model.step1)
### 步骤一：下载并导入 TUIPlayer 组件
点击进入 [Github](https://github.com/LiteAV-TUIKit/TUIPlayer) ，选择克隆/下载代码，然后拷贝 Android/tuiplayer 和 Android/TUICore 目录到您的工程中，并完成如下导入动作：
在项目的 `setting.gradle` 文件中添加我们的tuiplayer模块

```
   include ':tuiplayer'
   include ':tuicore'
```
在使用我们 TUIPlayer 的module的 `build.gradle` 文件中添加依赖

```
   api project(":tuiplayer")
```

[](id:model.step2)
### 步骤二：配置权限及混淆规则

在 AndroidManifest.xml 中配置 App 的权限，SDK 需要以下权限（6.0以上的 Android 系统需要动态申请相机、麦克风权限等）：

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />
```

在 proguard-rules.pro 文件，将 SDK 相关类加入不混淆名单：

```
-keep class com.tencent.** { *; }
```

[](id:model.step3)
### 步骤三：登录 TUI 组件库
在您初始化之前，请您完成以下操作
1. [开通直播服务并绑定域名](https://console.cloud.tencent.com/live/livestat) 如果还没开通，点击申请开通，之后在域名管理中配置推流域名和拉流域名
2. [获取SDK的测试License](https://console.cloud.tencent.com/live/license) 
3. [配置推拉流域名](https://console.cloud.tencent.com/live/domainmanage)

```java
TXLiveBase.getInstance().setLicence(this, "您的LICENSEURL", "您的LICENSEURLKEY");
```

#### 3.1 初始化SDK参数说明
- **LICENSEURL**：**TRTC 应用LICENSEURL**，通过上述步骤[获取SDK的测试License](https://console.cloud.tencent.com/live/license)获取；
- **LICENSEURLKEY**：**TRTC 应用LICENSEURLKEY**，通过上述步[获取SDK的测试License](https://console.cloud.tencent.com/live/license)获取；

```java
  // 2.组件登录，
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
```
#### 3.2 登录组件参数说明
- **SDKAppID**：**TRTC 应用ID**，如果您未开通腾讯云 TRTC 服务，可以点击 [这里](https://console.cloud.tencent.com/trtc/app)，进入腾讯云实时音视频控制台，创建一个新的的TRTC应用后，点击应用信息，SDKAppID信息如下图所示；
 <img src="https://liteav.sdk.qcloud.com/app/doc/app_manager_sdk_secretkey.png" width="700">
	
- **Secretkey**：**TRTC 应用密钥**，和SDKAppId对应，进入 [TRTC 应用管理](https://console.cloud.tencent.com/trtc/app) 后，SecretKey信息如上图所示；

- **userId**：当前用户的 ID，字符串类型，长度不超过32字节，不支持使用特殊字符，建议使用英文或数字，可结合业务实际账号体系自行设置。

- **userSig**：根据SDKAppId、userId，Secretkey等信息计算得到的安全保护签名，您可以点击 [这里](https://console.cloud.tencent.com/trtc/usersigtool) 直接在线生成一个调试的userSig，也可以参照我们的[TUIPlayer示例工程](https://github.com/LiteAV-TUIKit/TUIPlayer/app/src/main/java/com/tencent/qcloud/tuikit/tuiplayer/demo/debug/GenerateTestUserSig.java#L74)自行计算，更多信息见 [如何计算及使用 UserSig](https://cloud.tencent.com/document/product/647/17275)。

[](id:model.step4)
### 步骤四：初始化组件并使用相关功能
1. 创建 `TUIPlayerView`

```
   TUIPlayerView mTUIPlayerView = new TUIPlayerView(Context);
```

2. 为 `TUIPlayerView` 设置事件回调
```
    mTUIPlayerView.setTUIPlayerViewListener(new TUIPlayerViewListener() {
        ...
    }
```

3. 使用 `TUIPlayerView` 相关功能

### 开始拉流
拉流URL的生成请参考[这里](https://cloud.tencent.com/document/product/454/7915)
```
    mTUIPlayerView.startPlay(String url);
```

### 停止拉流
```
    mTUIPlayerView.stopPlay();
```

[](id:model.step5)
### 步骤五：在TUIPlayer中集成其他TUIKit组件（可选）
我们在自己的[小直播](https://github.com/tencentyun/XiaoZhiBo)工程中使用了该TUIPlayer组件并集成了其他 `TUIKit` 组件，您可以以此为参考自行实现。

## 三. 常见问题

>? 更多帮助信息，详见 [TUI 场景化解决方案常见问题](https://cloud.tencent.com/developer/article/1952880)，欢迎加入 QQ 群：592465424，进行技术交流和反馈~
