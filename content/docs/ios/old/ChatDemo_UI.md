---
title: 环信
sidebar: iossidebar
secondnavios: true
---

# EaseMob SDK 集成

## 特别说明
**username：环信IM用户唯一标识符**

**SDK的username不能是中文,且不区分大小写（[用户名规则](http://www.easemob.com/docs/rest/userapi/#eid)）**

**只要知道对方username，不需要互为好友即可聊天**

**SDK支持arm64位**

**SDK需要使用xcode6以上编译**

**因为iOS静态库问题，SDK较大，但不影响打包ipa后大小，打包ipa后仅增加2MB左右**

## 下载环信Demo及SDK

1. 下载环信Demo及SDK： [下载](http://www.easemob.com/sdk/)

2. 解压缩iOSSDK.zip后会得到以下目录结构：

![alt text](/iOS_Example_layout_IOS.png "Title")


## 将EaseMobSDK拖入到项目中 

![alt text](/iOS_Import.png "Title")

## 添加SDK依赖库 

![alt text](/iOS_AddLibs.png "Lib")

## 设置Linker 

![alt text](/iOS_OtherLinker.png "link")

向Other Linker Flags 中添加 -ObjC(如果已有，则不需要再添加)`注意O,C大写`

# SDK的使用，以demo源码为示例

## UIDemo依赖库 
![alt text](/iOS_AddUIDemoLibs.png "UIDemoLib")

## 初始化EaseMobSDK ，见AppDelegate

###在AppDelegate中注册SDK

<pre class="hll"><code class="language-objective_c">
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary 	*)launchOptions
{

    //注册 APNS文件的名字, 需要与后台上传证书时的名字一一对应
    NSString *apnsCertName = @"chatdemo";
    [[EaseMob sharedInstance] registerSDKWithAppKey:@"easemob-demo#chatdemo" apnsCertName:apnsCertName];
    // 需要在注册sdk后写上该方法
    [[EaseMob sharedInstance] application:application didFinishLaunchingWithOptions:launchOptions];

    return YES;
}
</code></pre>


关于EASEMOB_APPKEY，请登录或注册[环信开发者后台(https://console.easemob.com),申请APPKEY后，进行相关配置。

### 配置apns相关函数（需要在真机上运行）

<pre class="hll"><code class="language-objective_c">
//自定义方法
- (void)registerRemoteNotification
{
#if !TARGET_IPHONE_SIMULATOR
    UIApplication *application = [UIApplication sharedApplication];

    //iOS8 注册APNS
    if ([application respondsToSelector:@selector(registerForRemoteNotifications)]) {
    [application registerForRemoteNotifications];
    UIUserNotificationType notificationTypes = UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert;
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:notificationTypes categories:nil];
    [application registerUserNotificationSettings:settings];
    }else{
    UIRemoteNotificationType notificationTypes = UIRemoteNotificationTypeBadge |
    UIRemoteNotificationTypeSound |
    UIRemoteNotificationTypeAlert;
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:notificationTypes];
    }

#endif
}

//系统方法
-(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    //SDK方法调用
    [[EaseMob sharedInstance] application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

//系统方法
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
    //SDK方法调用
    [[EaseMob sharedInstance] application:application didFailToRegisterForRemoteNotificationsWithError:error];
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"注册推送失败"
    message:error.description
    delegate:nil
    cancelButtonTitle:@"确定"
    otherButtonTitles:nil];
    [alert show];
}

//系统方法
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    //SDK方法调用
    [[EaseMob sharedInstance] application:application didReceiveRemoteNotification:userInfo];
}

//系统方法
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
    //SDK方法调用
    [[EaseMob sharedInstance] application:application didReceiveLocalNotification:notification];
}
</code></pre>


### 实现其他方法

<pre class="hll"><code class="language-objective_c">
- (void)applicationWillResignActive:(UIApplication *)application
{
    [[EaseMob sharedInstance] applicationWillResignActive:application];
}

- (void)applicationDidEnterBackground:(UIApplication *)application
{
    [[EaseMob sharedInstance] applicationDidEnterBackground:application];
}

- (void)applicationWillEnterForeground:(UIApplication *)application
{
    [[EaseMob sharedInstance] applicationWillEnterForeground:application];
}

- (void)applicationDidBecomeActive:(UIApplication *)application
{
    [[EaseMob sharedInstance] applicationDidBecomeActive:application];
}

- (void)applicationWillTerminate:(UIApplication *)application
{
    [[EaseMob sharedInstance] applicationWillTerminate:application];
}
</code></pre>



## 如果某个类想监听SDK的回调方法，该类需要符合协议<IChatManagerDelegate>，并且需要注册为listener ,如 MainViewController

<pre class="hll"><code class="language-objective_c">
[[EaseMob sharedInstance].chatManager addDelegate:self
delegateQueue:nil];
</code></pre>


## 登录 ，见 LoginViewController

<pre class="hll"><code class="language-objective_c">
[[EaseMob sharedInstance].chatManager asyncLoginWithUsername:username
password:@"123456"
completion:
^(NSDictionary *loginInfo, EMError *error) {
    if (!error) {
    NSLog(@"登录成功");         
    }
} onQueue:nil];
</code></pre>


## 退出登录：见SettingsViewController 

<pre class="hll"><code class="language-objective_c">
[[EaseMob sharedInstance].chatManager asyncLogoffWithUnbindDeviceToken:YES];
</code></pre>


## 发送消息：工具类 ChatSendHelper 

<pre class="hll"><code class="language-objective_c">
EMChatText *text = [[EMChatText alloc] initWithText:message];
EMTextMessageBody *body = [[EMTextMessageBody alloc] initWithChatObject:text];

EMMessage *msg = [[EMMessage alloc]
initWithReceiver:@"bot"
bodies:[NSArray arrayWithObject:body]];
//msg.ext = ...;

[[EaseMob sharedInstance].chatManager sendMessage:msg
progress:nil
error:nil];
</code></pre>


## 接收聊天消息并显示：见ChatViewController 

<pre class="hll"><code class="language-objective_c">
 -(void)didReceiveMessage:(EMMessage *)message 
 {
     id\<\IEMMessageBody\>\ body = [message.messageBodies firstObject];
     if (body.messageBodyType == eMessageBodyType_Text) {
     NSString *msg = ((EMTextMessageBody *)body).text;
     NSLog(@"收到的消息---%@",msg);
 }
</code></pre>


## 其他说明

### 回调方法：监测网络状态

在MainViewController类中有体现，监测以下方法

<pre class="hll"><code class="language-objective_c">
//IChatManagerDelegate 登录状态变化
- (void)didConnectionStateChanged:(EMConnectionState)connectionState
{

}
</code></pre>

![alt text](/iOS_ChatUIDemoNetwork.png "Demo") 

### 回调方法：账号在其它设备登录

账号在其它设备登录时, 当前设备会自动断开连接(收到该回调时, 当前客户端已不能收发消息了, 当前客户端必须处理该回调, 退出到登录页面, )

<pre class="hll"><code class="language-objective_c">
- (void)didLoginFromOtherDevice
{
//退出到登录页面代码
}
</code></pre>

### 回调方法：账号在后台被删除

<pre class="hll"><code class="language-objective_c">
- (void)didRemovedFromServer
{

}
</code></pre>

## Demo演示流程
  
### 运行demo

账号不支持中文

 ![alt text](/iOS_ChatUIDemoLogin.png "Demo")
 
### 登录成功进入首页

会话：聊天的会话列表

通讯录：申请通知，群组，好友列表

设置：退出登录

 ![alt text](/iOS_ChatUIDemoHome.png "Demo")
 
### 添加好友

运行程序并登录账号2。点击“通讯录”页面的“+”

 ![alt text](/iOS_ChatUIDemoOther.png "Demo")
 
输入好友用户名（账号1），进行搜索添加
 
 ![alt text](/iOS_ChatUIDemoAddFriend.png "Demo")
 
在账号1接收账号2的好友申请
 
 ![alt text](/iOS_ChatUIDemoApplyList.png "Demo")
 
### 账号1和账号2互发消息

 ![alt text](/iOS_ChatUIDemoChatList.png "Demo") 
