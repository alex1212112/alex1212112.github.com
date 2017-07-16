---
layout: post
title: "Touch ID 的使用"
date: 2015-01-31 19:41:53 +0800
comments: true
categories: iOS
---
![](/images/201501312106.png)


### 目录

1. 序
2. 思路
3. 具体实现
4. 参考资料

### 序

iOS8，苹果开放了 Touch ID 的 SDK，苹果自身的 Connect、 Health 应用都使用了 Touch ID 来进行解锁登录，一些第三方应用比如 1Password 也使用了Touch ID 的登录方式，那么对于我们自己的应用，如何来使用 Touch ID 这个优秀的工具呢？下面是个人的一些想法和实践。

### 思路

既然使用 Touch ID 来自动登录，那么一定要把用户的的账号密码信息存储起来，然后在通过 Touch ID 验证通过的时候来获取到存储的用户密码，进行登录。因为密码的安全性很重要，所以可以把密码存储到 Keychain 里，如果觉得单独的存储字符串还是不够安全，可以在存储之前做一层 AES 的加密。流程分两部分，一部分为配置Touch ID,一部分为使用 Touch ID。流程如下

#### 配置 Touch ID

1. 用户正常登录
2. 登陆成功之后，通过 Keychain 把用户名和密码存储起来
3. 在设置里面面增加一项，启用 Touch ID，当用户选择启用 Touch ID 的时候，把该用户名存入到 NSUserDefaults 里。

#### 使用Touch ID

1. 用户输入用户名
2. 用户点击密码输入框
3. 先检查设备是否具备 Touch ID 的功能，然后通过 NSUserDefaults 查询该用户是否启用了 Touch ID 以及在 Keychain 中是否能获取到该用户信息。
4. 如果以上条件都满足，就弹出 Touch ID 验证的视图
5. 用户 通过 Touch ID 的验证，根据用户输入的用户名在 Keychain里 得到对应的密码，填充到密码框里
6. 登录


### 具体实现

#### 检查设备是否具备 Touch ID 功能

```objc
- (BOOL)canEvaluatePolicy
{
    LAContext *context = [[LAContext alloc] init];
    NSError *error = nil;
    BOOL success;

    // test if we can evaluate the policy, this test will tell us if Touch ID is available and enrolled
    success = [context canEvaluatePolicy: LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error];

    return success;
}
```

注意 添加 `@import LocalAuthentication;`


#### 显示 Touch ID 的 UI视图

```objc
- (void)evaluatePolicyWithTitle:(NSString *)title successHandler:(void(^)())successHandler
{
    LAContext *context = [[LAContext alloc] init];

    // show the authentication UI with our reason string

    [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:title reply:
     ^(BOOL success, NSError *authenticationError) {

         dispatch_async(dispatch_get_main_queue(), ^{

             if (success)
             {
                 if (successHandler)
                 {
                     successHandler();
                 }
             }
         });

     }];
}
```
其中 title 是用来描述我们使用 Touch ID来做什么的，比如 “解锁 APP” 之类的，successHandler 是 Touch ID 验证通过的回调，我们可以在这里来获取 Keychain 里的用户密码，然后进行登录，比如

```objc
 [self evaluatePolicyWithTitle:NSLocalizedString(@"Unlock APP", nil) successHandler:^{

                    self.pwdTextField.text = [self passwordFromKeychainWithUsername:_userNameTextField.text];
                    [self login];
                }];
```

#### 用户名密码的存储

这里用的 Keychain 是 Justin Williams 封装的 SGKeychain

存储到 Keychain

```objc
- (void)saveToKeyChainWithUsername:(NSString *)username password:(NSString *)password
{
    SGKeychainItem *item = [[SGKeychainItem alloc] init];
    item.account = username;
    item.service = @"APP";
    item.secret = password;

    [SGKeychain storeKeychainItem:item completionHandler:^(NSError *error) {
        if (!error)
        {
            NSLog(@"Password successfully created");
        }
        else
        {
            NSLog(@"Password failed to be created with error: %@", error.description);
        }
    }];
}
```

从 Keychain 获取密码

```objc
- (NSString *)passwordFromKeychainWithUsername:(NSString *)username
{
    SGKeychainItem *item = [[SGKeychainItem alloc] init];
    item.account = username;
    item.service = @"APP";

    // Fetch the password
    [SGKeychain populatePasswordForItem:item completionHandler:^(NSError *error){

        NSLog(@"error == %@",error.description);
    }];

    NSLog(@"password ==%@",item.secret);

    return item.secret;
}

```


### 参考资料

1. [SGKeychain](https://github.com/secondgear/SGKeychain)
2. [在iOS 8 SDK中使用Touch ID API](http://www.cocoachina.com/ios/20141114/10222.html)
3. [ iOS8 Touch ID api接口调用](http://blog.csdn.net/johnson_puning/article/details/36188255)
