---
layout: default
title: Fullter
nav_order: 15
parent: 语言
---

关联：
~~~


- (void)viewDidLoad
{
    [super viewDidLoad];
       
    [self.flutterEngine runWithEntrypoint:nil initialRoute:@"message"];

    [self addChildViewController:self.messageFlutterVC];
    [self.view addSubview:self.messageFlutterVC.view];
    
}

- (FlutterViewController *)messageFlutterVC
{
    if (!_messageFlutterVC) {
        _messageFlutterVC = [[FlutterViewController alloc] initWithEngine:self.flutterEngine nibName:nil bundle:nil];
        _messageFlutterVC.view.frame = self.view.bounds;
    }
    return _messageFlutterVC;
}

- (FlutterEngine *)flutterEngine
{
    if (!_flutterEngine) {
        _flutterEngine = [[FlutterEngine alloc] initWithName:"flutterEngineName"];
    }
    return _flutterEngine;
}

- (FlutterMethodChannel *)flutterMethodChannel
{
    if (!_flutterMethodChannel) {
        _flutterMethodChannel = [[FlutterMethodChannel methodChannelWithName:@"com.jfshare.ipxmall/message" binaryMessenger:self.flutterEngine.binaryMessenger] init];
    }
    return _flutterMethodChannel;
}
~~~

交互通过channel：
~~~
//注册Flutter调用OC回调
 @weakify(self);
[self.flutterMethodChannel setMethodCallHandler:^(FlutterMethodCall * _Nonnull call, FlutterResult  _Nonnull result) {
    @strongify(self);
    if ([call.method isEqualToString:@"finish"])
    {
        [self onBack];
    } else if ([call.method isEqualToString:@"getToken"])
    {
        result([NSString stringWithFormat:@"Bearer %@", curModel.accessToken]);
        
    } else if ([call.method isEqualToString:@"getUrl"])
    {
        result(kAppBaseURL);
        
    } else if ([call.method isEqualToString:@"getRefreshToken"])
    {
        result(curModel.refreshToken);
        
    } else if ([call.method isEqualToString:@"saveToken"])
    {
        if ([call.arguments isKindOfClass:NSString.class]) {
            curModel.accessToken = (NSString *)call.arguments;
            curModel.refreshToken = (NSString *)call.arguments;
        }
        
    } else if ([call.method isEqualToString:@"ReLogin"])
    {
        [MGJRouter openURL:XH_ROUTE_ACCOUNT_LOGOUT];
        [MGJRouter openURL:XH_ROUTE_ACCOUNT_LOGIN];
        
    } else if ([call.method isEqualToString:@"link"])
    {
        if ([call.arguments isKindOfClass:NSString.class]) {
            [MGJRouter openURL:call.arguments];
        }
        
    } else if ([call.method isEqualToString:@"getUserId"])
    {
        result(@(curModel.userId));
        
    } else if ([call.method isEqualToString:@"getDeviceId"])
    {
        result(PkgInfo.deviceId);
        
    } else if ([call.method isEqualToString:@"Toast"])
    {
        if ([call.arguments isKindOfClass:NSString.class]) {
            [SVHUD showMessage:call.arguments];
        }
        
    } else if ([call.method isEqualToString:@"getAppVersion"])
    {
        result(PkgInfo.appVersion);
        
    } else if ([call.method isEqualToString:@"startCommentView"])
    {
        
        NSString *feedId = call.arguments;
        if (feedId && [feedId isKindOfClass:NSString.class]) {
            
        }
            
    } else if ([call.method isEqualToString:@"startVisitor"])
    {
        NSString *userId = call.arguments;
        
    }else {
        result(FlutterMethodNotImplemented);
    }
}];


 //调用flutter
 [self.flutterMethodChannel invokeMethod:@"canBack" arguments:nil result:^(id  _Nullable result) {
     if ([result isKindOfClass:FlutterError.class]) {
        FlutterError *error = (FlutterError *)result;
        XHLog(@"%@", error.message);
         [self.navigationController popViewControllerAnimated:YES];
     } else {
         [self.navigationController popViewControllerAnimated:YES];
     }
    
}];
~~~