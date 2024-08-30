# 实现App内的视频/直播悬浮小窗播放

*   功能：使用webview的方案，在App中实现视频/直播的悬浮小窗播放效果，支持应用内小窗、系统小窗，在订单页小窗播放，回到直播间小窗消失

*   场景：适用于要在App中快速实现直播带货场景，或在App中需要点播视频小窗播放的场景

*   支持环境：安卓、iOS、鸿蒙

*   前置条件：使用[保利威](https://www.polyv.net/?src=wh "保利威")的webview SDK Demo，对方案有任何疑问，+V:wjc24680525，备注“小窗”

# 一、安卓对接说明

## 1.1 集成简介

### **1.1.1 项目架构图**

![Image](https://github.com/user-attachments/assets/765d3cd8-9ba6-4da0-8125-c2f5cf1b7d66)

### **1.1.2 使用源码项目集成**

当前将提供的源码包中的PolyvAndroidWebViewDemo、PLVWebViewSDK、PLVJsBridge、PLVFloatWindow四个项目放入到同一目录下，然后使用编译器打开PolyvAndroidWebViewDemo项目即可。

### **1.1.3 源码集成注意事项**

#### **功能对接说明**

参考 1.2 功能对接文档

#### **更换通信协议**

如果需要更换web-native通信协议的情况时，可以参考提供的  1.3 更换web-native通信规则注意事项[](https://j9k9c2q2qc.feishu.cn/docx/U7D5dyO4NoXlSBxu0emchu5AnSD#share-OCQMd5qbuoyuBFx0xDvccJ2xnXg "1.3 更换web-native通信规则注意事项")

## 1.2 功能对接文档

### **1.2.1 功能设置**

#### **设置Url和** **UA**

在启动PLVWebViewDemoActivity之前，我们需要设置加载的Url和添加上需要用到的Ua，这里可以通过`PLVWebViewConfig`来进行设置，示例代码如下：

    PLVWebViewConfig config = new PLVWebViewConfig(); 
    config.setUrl("需要加载的url") .setUa("需要添加的UA"); 
    PLVWebViewDemoActivity.startWebViewDemoActivity(PLVURLInputActivity.this, config);

#### **设置小窗边框颜色**

当前SDK支持设置小窗边框颜色，通过`PLVFloatWindowManager`进行设置，示例代码如下：

    PLVFloatWindowManager.getInstance().setSolidColor(Color.RED);

##### **1）设置小窗边框厚度**

当前SDK支持设置小窗边框颜色，通过`PLVFloatWindowManager`进行设置，示例代码如下：

    PLVFloatWindowManager.getInstance().setSolidWidth(20);

##### **2）设置自动开启小窗功能**

当进程退到后台时会自动唤出小窗(默认关闭)，通过`PLVFloatWindowManager`进行设置，示例代码如下：

    PLVWebViewConfig config = new PLVWebViewConfig(); 
    config.setSupportAutoFloating(true); // 开启自动开启小窗功能 
    PLVWebViewDemoActivity.startWebViewDemoActivity(PLVURLInputActivity.this, config);

##### **3）使用系统/应用小窗**

当前小窗支持系统小窗和应用窗(默认使用系统小窗)，如果开启系统小窗需要请求小窗悬浮权限，而应用小窗不需要请求权限，注意应用小窗依赖于Activity，当Activity被销毁时，应用小窗也会被关闭。

    PLVWebViewConfig config = new PLVWebViewConfig(); 
    config.isSystemFloatingWindow(true);// 为true时使用系统窗，false为使用应用小窗 
    PLVWebViewDemoActivity.startWebViewDemoActivity(PLVURLInputActivity.this, config);

##### **4）使用原生/web弹出请求权限窗**

当前sdk支持通过原生或web端来弹出请求悬浮权限弹窗（默认使用原生弹窗方式）。

注意如果是选择使用web端来弹窗请求悬浮权限弹窗，需要web端支持对应的事件。

    PLVWebViewConfig config = new PLVWebViewConfig(); 
    config.setUseWebRequestPermission(true);// 为true时web弹窗，false为使用原生弹窗 
    PLVWebViewDemoActivity.startWebViewDemoActivity(PLVURLInputActivity.this, config);

### **1.2.2 监听方法** **js** **事件**

SDK内部已经定义了一部分与web端通信的js事件，这些通信事件可以在demo层中重写某些方法进行监听和拦截处理，这里以监听onShare事件进行为例：

    public class PLVWebViewDemoActivity extends PLVWebViewBaseActivity { 
        @Override 
        public void onShare() { 
            // 监听到onShare事件的处理 
            ... 
            super.onShare(); 
        } 
    }

可以选择需要监听的js事件进行重写实现对事件的监听。

`注意:这些js通信事件都是与webview关联，开启系统小窗后,即使Activity被销毁依然能收到来自web端发送的消息，触发这些方法，所以重写这些方法时，尽量避免做与Activity生命周期关联的操作`

#### **允许的监听** **js** **事件**

当前支持通过重写方法来监听事件；

有以下这些方法：

        /**
         * 点击商品，将切换到小窗时会触发该回调
         *
         * @param width   小窗的宽
         * @param height  小窗的高
         * @param newPage 是否打开新的一页
         * @param link    打开新的一页加载的url
         * @param data    其他更多数据，开发者可根据需要自定义实现逻辑，数据为json格式
         */
        void changeToWindowForProduct(int width, int height, boolean newPage, String link, String data);

        /**
         * 用户主动调用切换到小窗的方法，在切换前会触发该回调
         *
         * @param width  小窗的宽
         * @param height 小窗的高
         */
        void changeToWindowForUser(int width, int height);

        /**
         * 点击小窗区域，从小窗中恢复到页面触发该回调
         **/
        void changeToNormal();

        /**
         * 点击小窗关闭按钮，关闭小窗触发该回调
         **/
        void closeWindow();

        /**
         * 点击返回事件
         */
        void onGobackPressed();

        /**
         * 横置屏幕事件
         */
        void onLandScreen();

        /**
         * 竖直屏幕事件
         */
        void onPortraitScreen();

        /**
         * 分享事件
         */
        void onShare();

        /**
         * 收藏事件
         */
        void onCollect();

        /**
         * 隐藏状态栏，进入沉浸模式
         */
        void hideNavigationBar();

        /**
         * 显示状态栏，退出沉浸模式
         */
        void showNavigationBar();

        /**
         * 打开悬浮窗权限页面
         */
        void requestFloatWindowPermissionByWeb();

        /**
         * 发送当前小窗的打开状态给前端
         */
        void getFloatWindowStatus();

        /**
         * 获取当前的网路状态
         */
        void getCurrentNetworkStatus();

        /**
         * 设置是否开启自动悬浮窗权限
         * @param enable true为打开，false为甘比
         */
        void setEnableBackground(boolean enable);

        /**
         * 获取是否开启了自动悬浮窗的权限
         */
        void getEnableBackground();

        /**
         * 处理其他event
         */
        void handleOtherEvent(String event);

### 1.2.**3 注册新的** **js** **通信事件**

SDK支持注册自定义js事件，可以通过下面的方式来注册自定义js事件，示例代码如下：

    @Override
    protected void initHandleForDemo() {
        //监听来自web端发送事件
       floatableLayout.getWebView().registerHandler("监听事件", new BridgeHandler() {
            @Override
            public void handler(String s, CallBackFunction callBackFunction) {}
        });
        
        //向web端发送对应事件和消息
       floatableLayout.getWebView().callHandler("发送事件", "发送消息", new CallBackFunction() {
            @Override
            public void onCallBack(String s
            }
        });
    }

`注意：注册新的js事件时不仅需要原生端做相应的操作，还需要web端有注册对应的事件，否则是原生端是不会接收到对应的事件消息`

## 1.3 **更换web-native通信规则注意事项**

### **1.3.1 说明**

当前SDK内部是通过Jsbridge该库来实现web端与原生端的通信，Jsbridge中定义了web端与原生的通信规则，如果集成项目是没有制定指定的web-native通信规则，那么可以直接使用SDK的通信规则即可，无需其余改动。

如果集成项目中有定制指定的web-native通信规则，可以参考下面的方式进行修改。

无论是使用哪一种通信规则，关键在于原生端发送/接收web端消息，web端发送/接收原生端消息，所以当需要更换SDK内部的web-native通信规则仅关注上述的两点即可。

### **1.3.2 更换native端通信规则**

因为sdk内部是依赖于jsbridge该模块实现web-native通信，当需要更换通信规则时，可以选择不依赖该模块。

### **1.3.3 更改PLVBaseWebView继承类**

    public class PLVBaseWebView extends BridgeWebView {
        ....
    }

可将PLVBaseWebView继承的BridgeWebView更换为集成项目中指定通信规则的webview。

### **1.3.4 更改WebViewClient**

WebViewClient是实现原生端接收web端消息的关键，也是定制修改webview的核心部份，所以在更换通信规则时也需要更换WebViewClient

    //替换自己需要用到的WebViewClient 
    webView.setWebViewClient(webviewClient);

可以通过setWebViewClient(new WebViewClient() 方法为webview设置指定WebViewclient。

### **1.3.5 更改原生端发送/接收web端消息的方法**

1.  更换发送消息方式

<!---->

    public void callMessage(String type, String message) {
        //当使用的新的web-native规则时，可以将下面的代码修改为webview使用新规则发送消息时的代码
        webview.callHandler(type, message, new CallBackFunction() {
            @Override
            public void onCallBack(String s) {
              ...
            }
        });
    }

当前SDK内部时通过webview\.callHandler()方法来实现消息的发送，当更换新的通信规则时，可以将这个webview\.callHandler()方法替换为新规则中对应原生端发送消息的方法。

2.  更换接收web端消息

<!---->

    webview.registerHandler("xxx", new BridgeHandler() {
        @Override
        public void handler(String s, CallBackFunction callBackFunction) {
            ...    
        }
    });

当前SDK内部是通过 webview\.registerHandler()方法来监听web端发送的消息，当更换新的通信规则时，可以将这个webview\.registerHandler()方法替换为新规则中对应原生端接收web消息的方法。

### **1.3.6 更换web端通信规则**

web-native通信规则是由web端和原生端两端定制的，所以当更换web-native通信规则不仅需要原生端更换，web端也需要进行更换。

#### **注意事项**

当前需要通信的 web页面 与 更换通信规则后的原生端 所使用的通信规则是否是对应，如果是对应的情况下无需做其他更替。

如果需要通信的web页面与 更换通信规则后的原生端 所使用的不对应，那么就需要去更替web端的通信规则。

`如web端页面需要使用保利威的web页面（当前保利威web页面与sdk内部所使用的通信规则一致，当原生端切换通信规则，那么web端页面也需要更换对应通信规则）`

### **1.3.7 更换web端发送/接收方法**

#### 1.更改web端发送消息方式

     window.bridge.callHandler(
                    'callAppEvent', message,
                    function(responseData) {
                       //发送消息
                        ....
                    }
                );

当前SDK内部与Web端对应的通信规则是通过bridge.callHandler()方法来进行发送消息，当替换新的通信规则后，可以使用新的发送方式来取代这个bridge.callHandler()方法

#### 2.更改web端接收消息方式

    // 监听来自xxx事件的消息
    bridge.registerHandler("xxx", function(data, responseCallback) {
                    document.getElementById("show").innerHTML = (data);
    });

当前SDK内部与Web端对应的通信规则是通过bridge.registerHandler()方法来接收消息，当替换新的通信规则后，可以使用新的发送方式来取代这个bridge.registerHandler()方法

# 二、iOS对接说明

## 2.1 集成简介

### 2.1.1 项目架构图

![Image](https://github.com/user-attachments/assets/00c2b420-fe58-4b9f-8cd7-8d94bf319b3e)

### 2.1.2 运行环境要求

| 名称·       | 要求      |
| --------- | ------- |
| iOS系统     | 9.0+    |
| CocoaPods | 1.11.3+ |
| Xcode     | 11.0+   |

### 2.1.3 API文档

详细接口文档请参见 API文档

| Demo 仓库 Tag | 依赖 SDK 版本 | API 文档                                                                                                  |
| ----------- | --------- | ------------------------------------------------------------------------------------------------------- |
| 3.0.0       | 3.0.0     | [v3.0.0 API](https://repo.polyv.net/ios/documents/PLVWebViewSDK/3.0.0-20240130/index.html "v3.0.0 API") |
| 2.1.2       | 2.1.2     | [v2.1.2 API](https://repo.polyv.net/ios/documents/PLVWebViewSDK/2.1.2-20221209/index.html "v2.1.2 API") |
| 2.1.1       | 2.1.1     | [v2.1.1 API](https://repo.polyv.net/ios/documents/PLVWebViewSDK/2.1.1-20221010/index.html "v2.1.1 API") |
| 2.1.0       | 2.1.0     | [v2.1.0 API](https://repo.polyv.net/ios/documents/PLVWebViewSDK/2.1.0-20220804/index.html "v2.1.0 API") |

## 2.2 快速集成

### 2.2.1 **项目配置**

#### **配置支持系统版本**

打开项目的 PROJECT - Deployment Target - iOS Deployment Target 改为 9.0 或更高。

打开项目的 TARGETS - General - Deployment Info，把 iOS 系统改为 9.0 或更高。

### **2.2.2 配置 App Transport Security (ATS)**

打开项目的 `info.plist `文件，添加如下内容：

    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
        <key>NSAllowsArbitraryLoadsInWebContent</key>
        <true/>
    </dict>

### 2.2.3 **配置设备旋转**

播放器支持全屏播放，需要在 TARGETS - General - Deployment Info 中，勾选支持的横屏旋转方向。

### 2.2.4 **配置后台播放和画中画**

打开项目的 TARGETS - Signings & Capabilities，点击 + Capability，选择 Background Modes，勾选 ‘Audio, AirPlay, and Picture in Picture’。

## **2.3 集成** **SDK**

### 2.**3.1 Pod方式集成**

#### **集成PLVWebViewSDK**

在 Podfile 文件中，添加以下内容：

    pod 'PLVWebViewSDK', '~> 1.0.0'

在终端执行 `pod install`

#### **集成系统小窗**

如果需要使用系统小窗功能，则需要在 Podfile 文件中另外添加以下内容

    # 包含系统画中画 -- 如果不使用系统小窗 则不需要下面配置
    pod 'PLVAliHttpDNS', '~>1.10.0'
    pod 'PLVFoundationSDK', '1.10.6', :subspecs => ['BaseUtils', 'NetworkUtils', 'ErrorCode', 'LogReporter', 'ConsoleLogger', 'Reachability', 'SafeModel', 'ProgressHUD']
    pod 'PLVBusinessSDK', '1.10.4', :subspecs => ['BaseBSH']
    pod 'PLVLiveScenesSDK', '1.10.6', :subspecs => ['Base', 'LogReporter', 'ConsoleLogger', 'ErrorManager', 'Network', 'Player', 'BasePlayer', 'LivePlayer', 'LivePlaybackPlayer', 'PictureInPicture']

并在终端执行 `pod install`，并且将对应的 `PLVLiveScenesSDK、PLVBusinessSDK` 进行替换。

### 2.**3.2 离线包方式集成**

#### **集成PLVWebViewSDK**

将 PLVWebViewSDK.framework 库添加到自己的项目中。如果是使用源码的方式集成，则需要将 PLVWebViewSDK 文件整个添加到自己的项目即可。

#### **集成系统小窗**

1.如果项目使用系统小窗功能则需要将以下库添加到项目中

    PLVAliHttpDNS、PLVBusinessSDK、PLVFoundationSDK、PLVIJKPlayer、PLVLiveScenesSDK

2.在General => Frameworks，Libraries，and Embedded Content中移除已添加的 PLVFoundationSDK.framework、PLVLiveScenesSDK.framework、PLVBusinessSDK.framework 库，并添加系统库 libresolv.tbd。

3.设置PLVIJKPlayer为Embed & Sign

4.Build Settings，Other Linker Flags中添加 -ObjC，如果项目已配置有，则不需要重复添加

5.运行报错

如果模拟器运行报错需要在Build Settings 中设置Excluded Architecture => Any iOS Simulator SDK 为 arm64

如果运行报错

    Building for iOS Simulator, but the linked and embedded framework '***' was built for iOS + tvOS Simulator

可在Build Settings 中 设置 VALIDATE\_WORKSPACE 为 YES即可。

## **2.4 WebView接入**

创建想要展示 WebView 的控制器页面 `DemoViewController` 继承于`PLVWebViewBaseViewController`，并通过初始化方法 `-initWithConfig:`创建控制器页面。示例代码如下：

    // PLVWebViewDemoViewController.h

    #import <PLVWebViewSDK/PLVWebViewSDK.h>

    @interface PLVWebViewDemoViewController : PLVWebViewBaseViewController

    @end


初始化时可通过重写 `initHandleForDemo` 进行自定义配置，同时可以对Bridge事件进行自定义处理。示例代码如下：

    // PLVWebViewDemoViewController.m
        
    @implementation PLVWebViewDemoViewController

    #pragma mark - [ Override ]

    - (void)viewDidLoad {
        [super viewDidLoad];
     
    }

    /// 初始化时，可以进行的自定义配置
    //- (void)initHandleForDemo {
    //
    //}

    #pragma mark - [ Delegate ]
    #pragma mark PLVFloatableWebViewBridgeDelegate
    /// 如需要监听 js 回调事件，可以通过Delegate事件进行相应的处理，例如webviewBridgeShare消息
    //- (void)webviewBridgeShare:(PLVFloatableWebViewBridge *)bridge {
    //
    //}

    @end


创建、配置、打开直播页面控制器

    PLVWebViewConfig *config = [[PLVWebViewConfig alloc] init];
    config.urlString = urlString;
    config.allowFloatingWindow = YES;
    config.isSystemFloatingWindow = NO;
    config.userAgent = self.uaTextView.text;
    config.enableAutoFloatWindow = YES;
        
    PLVWebViewDemoViewController *webVC = [[PLVWebViewDemoViewController alloc] initWithConfig:config];
    [self.navigationController pushViewController:webVC animated:YES];


## 2.5 高级功能

### **2.5.1 WebView 配置**

创建 PLVWebViewBaseViewController 时需要根据 PLVWebViewConfig 配置来进行初始化。

    PLVWebViewConfig *config = [[PLVWebViewConfig alloc] init];
    config.urlString = @"";
    config.allowFloatingWindow = YES;
    config.isSystemFloatingWindow = NO;
    config.userAgent = @"";
    config.enableAutoFloatWindow = YES;    
    PLVWebViewDemoViewController *floatingWebVC = [[PLVWebViewDemoViewController alloc] initWithConfig:config];


*   allowFloatingWindow 是否允许使用浮窗的功能
*   isSystemFloatingWindow 使用系统小窗或者应用内小窗
*   userAgent 浏览器的UA，可自行修改
*   enableAutoFloatWindow 开启自动浮窗，开启后 在退出页面或者退到后台时会自动开启小窗

### **2.5.2 应用内小窗配置**

我们还可以对浮窗的大小、默认位置、以及样式进行配置修改：

    // 配置浮窗大小
    [[PLVFloatWindowManager sharedManager] setFloatingWindowSize:size];
    // 配置浮窗初始位置
    [[PLVFloatWindowManager sharedManager] setFloatingWindowLocation:point];
    // 应用内小窗样式修改
    UIView *appWindowView = [PLVFloatWindowManager sharedManager].floatingWindow;
    appWindowView.layer.cornerRadius = 8.0f;
    appWindowView.layer.masksToBounds = YES;


其中，调用 `-moveContentViewToWindow:size:` 方法主动唤起的浮窗，默认宽度为屏幕的0.3倍，宽高比16:9。通过保利威的网页唤起浮窗，浮窗大小由网页告知。初始位置是指浮窗右下角距离屏幕右下角的相对位置，默认为(-10, -100)， 表示悬浮窗右边沿距离屏幕右边沿10pt，下边沿距离屏幕下边沿100pt。

### **2.5.3 画中画配置**

1.  如果要使用画中画功能，首先需要在 Podfile 文件中 添加支持画中画的SDK，然后执行 `pod install`；

        # 包含系统画中画 -- 如果不使用系统小窗 则不需要下面配置
        pod 'PLVAliHttpDNS', '~>1.10.0'
        pod 'PLVFoundationSDK', '1.10.6', :subspecs => ['BaseUtils', 'NetworkUtils', 'ErrorCode', 'LogReporter', 'ConsoleLogger', 'Reachability', 'SafeModel', 'ProgressHUD']
        pod 'PLVBusinessSDK', '1.10.4', :subspecs => ['BaseBSH']
        pod 'PLVLiveScenesSDK', '1.10.6', :subspecs => ['Base', 'LogReporter', 'ConsoleLogger', 'ErrorManager', 'Network', 'Player', 'BasePlayer', 'LivePlayer', 'LivePlaybackPlayer', 'PictureInPicture']


2.  替换对应的离线包

用提供的离线包`PLVLiveScenesSDK、PLVBusinessSDK` 将 Pods 文件夹下对应的SDK进行替换。

### **2.5.4** **UA** **和白名单配置**

当开启了 `enabelWhiteList` 后，只有添加了域名白名单的链接才会支持原生与前端的通信，才支持小窗的功能。

    [PLVUAConfigManager sharedManager].enabelWhiteList = YES;
    [[PLVUAConfigManager sharedManager] addHostWhitelist:@[@"live.polyv.cn"]];


同时支持 自定义配置 UA

    [[PLVUAConfigManager sharedManager] addCustomUserAgent:@""];


### **2.5.5 监听方法** **js** **事件**

在SDK内部已经对一些事件进行了处理，如果想要自定义处理某些事件可以在demo中进行监听拦截处理。示例如下：

    #pragma mark - [ Delegate ]
    #pragma mark PLVFloatableWebViewBridgeDelegate
    /// 如需要监听 js 回调事件，可以通过Delegate事件进行相应的处理，例如webviewBridgeShare消息
    - (void)webviewBridgeShare:(PLVFloatableWebViewBridge *)bridge {

    }


### **2.5.6 注册和调用自定** **js** **事件**

可以通过Demo层注册和调用自定义事件，示例代码如下：

    - (void)initHandleForDemo {
        [self.contentAreaView.mainWebView.bridge registerHandler:@"" handler:^(id  _Nonnull data, PLVWVJBResponseCallback  _Nonnull responseCallback) {
                
        }];
        [self.contentAreaView.mainWebView.bridge callHandler:@"" data:nil responseCallback:^(id  _Nonnull responseData) {
            
        }];
    }


## 2 **.6 更换web-native通信规则注意事项**

本项目中使用的是开源库 WebViewJavascriptBridge 来实现web 和 native之间的通信。如果接入的项目中没有用到web-native通信，则不需要做任何调整。如果接入的项目中也用到web-native通信，则可以参考下面的方式进行修改。

### **2.6.1 更换native端通信规则**

SDK中使用的 WebViewJavascriptBridge 来实现通信的，在 PLVFloatableWebViewBridge 中对其进行了封装处理，如果需要更换native端通信规则只需要修改 PLVFloatableWebViewBridge 这个类即可

#### 更换注册消息方式:

    - (void)registerHandler:(NSString*)handlerName handler:(WVJBHandler)handler {
          //当使用的新的web-native规则时，可以将下面的代码修改为webview使用新规则注册消息时的代码
        [self.bridge registerHandler:handlerName handler:handler];
    }


#### 更换发送web端消息

    - (void)callHandler:(NSString*)handlerName data:(id _Nullable)data responseCallback:(WVJBResponseCallback _Nullable)responseCallback {
          //当使用的新的web-native规则时，可以将下面的代码修改为webview使用新规则发送消息时的代码
        [self.bridge callHandler:handlerName data:data responseCallback:responseCallback];
    }


### **2.6.2 更换web端通信规则**

当native端通信规则改变时，web端则要根据是否与原来的通信方式原理相同来判断是否需要修改。

#### 当前web端注册消息方式

    bridge.registerHandler("testJavascriptHandler",
              function (data, responseCallback) {
                console.log(data);
                responseCallback(data);
              }
     );


#### 当前web端发送消息方式

    bridge.callHandler("callAppEvent", { },
              function responseCallback(responseData) {
                console.log(responseData);
              }
    );


当前web-native的通信是通过 bridge 这种方式来发送和接收消息的，如果需要调整可用集成项目web端接收、发送消息的方式来进行替换。

## **2.7** **iOS** **系统版本使用系统小窗功能说明**

iOS因为系统限制想实现系统小窗只能通过画中画的功能，同时这个功能会有版本限制，在iOS14+开始支持iPhone的画中画功能。同时直播如果使用系统播放器 AVPlayer 会有14s+的高延迟，IJKPlayer可以实现低延迟，但是iOS15以上画中画才支持SampleBufferLayer，因此如果直播低延迟的话需要iOS15+。

*   回放视频系统小窗支持iOS14+
*   直播视频系统小窗支持iOS15+

# 三、鸿蒙next

## 3.1 简介

PLVWebSDK 项目从属于易方信息科技股份有限公司，对保利威云直播、云点播系列产品的直播、回放观看做了良好的适配，极大优化了用户的观看体验，并支持浮窗播放等扩展功能，也可作为其他网页的展示容器。

本项目包含功能如下：

*   支持基础Web功能
*   支持应用内浮窗播放
*   支持系统级浮窗播放
*   支持悬浮窗手势拖动
*   支持视频全屏播放

## 3.2 下载安装

    ohpm install plvwebsdk


OpenHarmony ohpm 环境配置等更多内容，请参考如何安装 OpenHarmony ohpm 包

## 3.3 需要权限

    ohos.permission.INTERNET
    ohos.permission.GET_NETWORK_INFO


## 3.4 使用示例

### 3.4.1 绑定WindowStage

首先需要在Ability中为PLVWebViewWindowManager绑定用到windowStage

     onWindowStageCreate(windowStage: window.WindowStage): void {// Main window is created, set main page for this abilityPLVLogger.info(TAG, 'Ability onWindowStageCreate');// demo核心代码PLVWebViewWindowManager.setWindowStage(windowStage)
        windowStage.loadContent(CONTENT, new LocalStorage(), (err, data) => {if (err.code) {PLVLogger.info(TAG, 'Failed to load the content. Cause: ' + JSON.stringify(err) ?? '');
            return;
          }PLVLogger.info(TAG, 'Succeeded in loading the content. Data: ' + JSON.stringify(data) ?? '');
        });
      }


### 3.4.2 使用PLVWeb组件

    @Entry@Component
    struct PLVWebPage {@StorageLink('inputWebConfig') config: PLVWebViewConfig = new PLVWebViewConfig().setUrl("")@State controller: PLVWebController = new PLVWebController({ config: this.config!! })
      aboutToAppear(): void {this.controller.registerHandler("自定义事件", (data: string) => {// 监听自定义事件的处理
        })this.controller.callHandle("发送原生事件", "xxx", () => {//发送原生事件
        })this.controller.registerCustomerContainEvent({
          onShare: () => {// 监听到onShare事件时的操作return true // return true表示拦截掉分享事件 false就继续由sdk往下执行
          }
        })
      }
      build() {
        Column() {
          PLVWeb({ controller: this.controller })
        }
        .width(PLVCommonConstants.FULL_PERCENT)
      }
      onBackPress(): boolean | void {if (this.controller.accessBackward()) {this.controller.backward()return true
        }return false
      }
    }


`注意该组件需要放在一个单独的pages页面中，因为创建的Web窗口需要一个单独page页面`

### 3.4.3 Demo使用

    @Entry(storage)
    @Component
    struct PLVInputPage {
      inputUrl: string = 'https://www.polyv.net'
      config: PLVWebViewConfig = new PLVWebViewConfig().setUa("Android" + PLVUAConfig.defaultUA)
        .setUrl(this.inputUrl)
        .setAllowFloatingWindow(true)

      build() {
        Stack({ alignContent: Alignment.Top }) {
            ...
            Button("打开Web")
              .margin(24)
              .onClick(() => {
                if (!PLVWebViewWindowManager?.getWebViewWindow()) {
                  this.config.setUrl(this.inputUrl)AppStorage.setOrCreate('inputWebConfig', this.config);
                  // 为webWindow绑定一个page页面，用于窗口的显示// true表示初始化后立刻打开webWindow
                  PLVWebViewWindowManager?.initWindow(true, "pages/PLVWebPage")
                  return
                }
              })
          }
            ....
      }
    }


为PLVWebViewWindowManager绑定了WindowStage后，可以通过`PLVWebViewWindowManager?.initWindow(true, "pages/PLVWebPage")` 来进行初始化化，如果传入参数true时，当初始化后立刻展示WebWindow，也可以选择传入false，初始化后不立刻展示WebWindow 再通过`PLVWebViewWindowManager.showWebViewWindow()`来进行展示

## 3.5 功能说明

### 3.5.1 PLVWebViewConfig

1.设置UA

当前可以通过下面的方式来设置需要的UA

`PLVWebViewConfig().setUa("Android" + PLVUAConfig.defaultUA)`

注意：如果需要到Saas页面的小窗功能的情况，必须在UA中带上Android和PLVUAConfig.defaultUA字段

2.设置url

可以通过下面的方式来设置需要加载的url

PLVWebViewConfig().setUrl("需要加载的url")

3.设置是否允许小窗悬浮

可以通过下面的方式来设置是否允许开启小窗功能，默认是开启的，如不需要可以设置为false

`PLVWebViewConfig().setAllowFloatingWindow(true)`

### 3.5.2 自定义注册事件

当前SDK内部已经注册好与前端页面通信的事件，当接入sdk后就能直接使用这些事件。

SDK也支持注册自定义事件，可以通过下面的方式来进行注册

    this.controller.registerHandler("自定义事件", (data: string) => {// 监听自定义事件的处理
    })
    this.controller.callHandle("发送原生事件", "xxx", () => {//发送原生事件
    })


如果需要监听/拦截SDK内部已经实现了的事件，可以通过下面的方式继续监听/拦截，这里以onShare事件为例子，如果需要监听/拦截其他事件可以仿照这里来完成

    this.controller.registerCustomerContainEvent({
      onShare: () => {// 监听到onShare事件时的操作return true // return true表示拦截掉分享事件 false就继续由sdk往下执行
      }
    })


# 四、自定义事件

    // 小窗播放事件
    webviewBridage?.sendData('clickProduct', webviewData);

    // 没有小窗播放的情况： 监听自定义事件
    webviewBridage?.sendData('clickProductCustom', webviewData);

    // 数据格式
    const webviewData = {
            width: plvWebviewDataSize.width,
            height: plvWebviewDataSize.height,
            newPage: true,
            link:  “跳转的自定义 url”,
            data: {
                    type : out | inner | stock
                    link : “跳转的自定义 url” ( 同上 link )
                    market : ‘5053’,
                    code : 789465,
                    name : 名称
            },
    }

    out 外部
    inner 内部
    stock 基金 


clickProduct事件 和 clickProductCustom自定义 webviewData ---> data 下面根据 type 和 其他参数完成 app 内链接跳转功能
