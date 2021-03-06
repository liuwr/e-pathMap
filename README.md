# e-pathMap
e-pathMap 是一套基于 iOS 9.0 及以上版本的室内地图应用程序开发接口，供开发者在自己的iOS应用中加入室内地图相关的功能，包括：地图显示（多楼层、多栋楼）、室内导航、模拟导航、语音播报等功能。

## 获取AppKey
请填写
**[申请表](https://github.com/epath/EpathmapSDK-iOS/blob/master/Readme.asset/释途客户应用申请表.xlsx)** 
给liuwr@e-path.cn邮件联系获取appKey、mapId、scheme（分享用）

## 使用CocoaPods部署
在Podfile中使用命令如下：
```bash
pod 'e-pathMap','1.0.0'
```
然后运行以下命令

```bash
$ pod install
```

## 手动集成
1. 将SDK文件中包含的EpathmapSDK.framework、~~EpathmapSDK.xcassets两个文件~~添加到工程中。
2. 开发者需要在工程中链接上：CoreTelephony.framework、SystemConfiguration.framework、CFNetwork.framework、libz.tbd、libc++.tbd、libsqlite3.tbd。
3. 接入微信分享SDK可
**[点击](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417694084&token=&lang=zh_CN)** 
查看

### 注意
导入EpathmapSDK后需要
- Build Settings中将Enable Bitcode 改成 NO。
- 在Info.plist中添加授权申明
```bash
<key>NSBluetoothPeripheralUsageDescription</key>
<string>开启蓝牙以使用室内导航服务</string>
<key>NSBluetoothAlwaysUsageDescription</key>
<string>开启蓝牙以使用室内导航服务</string>
<key>NSCameraUsageDescription</key>
<string>释途App需要您的同意,才能访问相机</string>
<key>NSContactsUsageDescription</key>
<string>App需要您的同意,才能访问通讯录</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>释途会在室外定位及导航等服务中使用您的位置信息</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>释途会在室外定位及导航等服务中使用您的位置信息</string>
<key>NSMicrophoneUsageDescription</key>
<string>释途会在语音搜索中使用您的麦克风</string>
```
## 使用说明  

### SDK初始化
在 didFinishLaunchingWithOptions里面添加以下代码即可，appKey为邮件中获取的
```objective-c
//初始化Ipsmap
[EpathmapServices setAppKey:appKey];
[[EpathmapServices sharedInstance] application:application didFinishLaunchingWithOptions:launchOptions];
```


### 显示室内地图
```objective-c
EpathMapViewController *vc = [[EpathMapViewController alloc] initWithMapId:@"lGaWCUtqoj" withFrame:CGRectMake(0, 20, ScreenWidth, ScreenHeight-24)];
[self.navigationController pushViewController:vc animated:YES];
```
### 导航至具体地址
```objective-c
EpathMapViewController *vc = [[EpathMapViewController alloc] initWithMapId:@"lGaWCUtqoj" targetName:@"高交会" targetId:@"200191" withFrame:CGRectMake(0, 20, ScreenWidth, ScreenHeight-24)];
[self.navigationController pushViewController:vc animated:YES];
```
### 是否在地图范围内的接口
```objective-c
- (void)viewDidLoad {
[super viewDidLoad];
// Do any additional setup after loading the view.
self.view.backgroundColor = [UIColor whiteColor];

self.manger = [IpsmapLocationManger new];
self.manger.locationTimeOut = 20;
self.manger.delegate = self;

[self.manger startLocationEngine:@"lGaWCUtqoj"];
}

- (void)EpathmapLocationManager:(IpsmapLocationManger *_Nullable)manager didUpdateLocation:(IpsLocation *_Nullable)location 
{
if (location.inThisMap) {
NSLog(@"在建筑物里面");
}
}
```

### 位置共享功能

## 使用粘贴板实现

1、在 didFinishLaunchingWithOptions里面创建好window之后添加
```objective-c
//检测位置共享口令是否存在
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(EpathReceiveShareInfo:) name:EpathReceiveShareInfoNotification object:nil];
   [[EpathmapServices sharedInstance] application:application didFinishLaunchingWithOptions:launchOptions];
   ```
   2、在applicationWillEnterForeground里添加
   ```objective-c
   [[EpathmapServices sharedInstance] applicationWillEnterForeground:application];
   ```
   3、通知响应处理
   ```objective-c
   - (void)EpathReceiveShareInfo:(NSNotification *)noti {
   if (noti.object == nil || ![noti.object isKindOfClass:[UIView class]]) {
   return;
   }
   
   UIView *viewJoin = (UIView *)noti.object;
   //有分享口令
   dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
   [viewJoin performSelector:@selector(showInView:) withObject:[UIApplication sharedApplication].keyWindow];
   });
   }
   ```
   
   ## 使用微信分享SDK实现
   1、开发者可在自己的工程中接入微信SDK并完成register
   ```objective-c
   //例如在didFinishLaunchingWithOptions实现register
   [WXApi registerApp:wxAppID];
   ```
   2、将获取到的scheme配置到自己的工程中
   
   <img alt="EpathmapSDK" src="Readme.asset/scheme.jpg">
   
   3、在每个EpathMapViewController创建完之后加上locationShareDelegate处理分享按钮点击的回调
   ```objective-c
   //设置需要分享的平台
   [EpathShareConfig showSharePlatforms:@[@(IpsShareTypeWeChat), @(IpsShareTypeQQ), @(IpsShareTypeSMS)]];
   
   EpathMapViewController *vc = [[EpathMapViewController alloc] initWithMapId:@"lGaWCUtqoj"];
   vc.locationShareDelegate = [IpsLocationShareHandle sharedInstance];
   [self.navigationController pushViewController:vc animated:YES];
   ```
   4、分享按钮点击的回调处理可以用demo中的IpsLocationShareHandle也可以自己写
   
   5、点击分享链接App的处理在Appdelegate中的handleOpenURL方法加处理事件,delegate 需要支持IpsLocationShareProtocol协议
   ```objective-c
   - (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
   if ([url.scheme isEqualToString:wxAppID]) {
   return [WXApi handleOpenURL:url delegate:self];
   } else if ([url.scheme isEqualToString:EpathScheme])
   return [[EpathmapServices sharedInstance] application:application openURL:url delegate:[EpathLocationShareHandle sharedInstance]];
   return YES;
   }
   ```
   
   6、为了方便位置共享可以默认填写用户昵称，可在登录成功或者切换用户等获取到用户信息后传递用户信息
   ```objective-c
   EpathUserInfo *userInfo = [EpathUserInfo new];
   userInfo.name = @"张三";
   userInfo.phoneNumber = @"13888888888";
   [EpathmapServices sharedInstance].userInfo = userInfo;
   ```
   
