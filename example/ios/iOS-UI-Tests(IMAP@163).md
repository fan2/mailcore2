
编译 `mailcore2/example/ios/iOS UI Test/iOS UI Test.xcodeproj` 工程，启动真机或模拟器。

点击右上角按钮"Settings"，可设置账号、密码和 IMAP 服务器。

## QQ 邮箱
QQ 邮箱默认关闭了 `POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务`。

```
2016-03-15 16:36:07.640 iOS UI Test[4342:2551981] error loading account: Error Domain=MCOErrorDomain Code=5 "(null)" UserInfo={IMAPServerError=��ʹ����Ȩ���¼�������뿴: http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256}
```

如需开启，需要先对QQ号申请第二代密码保护。  
具体可参考：[什么是授权码，它又是如何设置？](http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256)

## Gmail
Gmail 邮箱默认开启了 `转发和 POP3/IMAP`，可进入【设置】|【转发和 POP3/IMAP】中查看设置。

```
2016-03-15 16:33:50.886 iOS UI Test[4336:2550856] error loading account: Error Domain=MCOErrorDomain Code=5 "(null)" UserInfo={IMAPServerError=Invalid credentials (Failure)}
```

可能由于调试 APP 的签名没有可行认证，被认定为非 `正规厂商的客户端`。尝试登陆会收到主题为 `阻止了登录尝试` 的安全提示邮件，可点击[了解详情](https://support.google.com/accounts/answer/6010255)，允许不够安全的应用访问您的帐户。  
点击进入[“不够安全的应用”部分](https://www.google.com/settings/security/lesssecureapps)，可启用 `不够安全的应用的访问权限` 。

## 163
### 开启 IMAP 服务
163 邮箱的 POP3/SMTP/IMAP 服务全部支持 SSL 连接。

登陆 http://mail.163.com/ 邮箱，【设置】|【POP3/SMTP/IMAP】。  
无法勾选“开启IMAP服务”，需要先设置客户端授权密码。  
通过手机短信验证后，即可设置客户端授权独立密码。  
然后，即可勾选“开启IMAP服务”。

点击右上角按钮"Settings"，设置 163 邮箱账户密码。

![iOS-UI-Tests(IMAP@163-Settings)](iOS-UI-Tests(IMAP@163-Settings).PNG)

**注意**：  
> 密码为设置的客户端授权独立密码！

### 收信请求被阻止

`IMAPSession::select->mailimap_select` 调用失败：

```
2016-03-15 16:43:33.161 iOS UI Test[4349:2554286] checking account
2016-03-15 16:43:33.162 [4349:9d27] MCOperationQueue.cpp:81: start thread
2016-03-15 16:43:33.171 [4349:9d27] MCIMAPSession.cpp:585: connect <mailcore::IMAPSession:0x14468ca40>
2016-03-15 16:43:34.817 [4349:9d27] MCIMAPSession.cpp:608: ssl connect imap.163.com 993 2
2016-03-15 16:43:35.647 [4349:9d27] MCIMAPSession.cpp:669: connect ok
2016-03-15 16:43:35.647 [4349:9d27] MCIMAPSession.cpp:709: login
2016-03-15 16:43:36.445 [4349:9d27] MCIMAPSession.cpp:962: login ok
2016-03-15 16:43:36.445 iOS UI Test[4349:2554286] finished checking account.
2016-03-15 16:43:36.448 [4349:9d27] MCIMAPSession.cpp:1049: select
2016-03-15 16:43:36.629 [4349:9d27] MCIMAPSession.cpp:1053: select error : 33
2016-03-15 16:43:37.717 [4349:main] MCOperationQueue.cpp:215: trying to quit 0x1447213a0
2016-03-15 16:43:37.717 [4349:9d27] MCOperationQueue.cpp:102: stopping 0x1447213a0
2016-03-15 16:43:37.717 [4349:main] MCOperationQueue.cpp:230: thread stopped 0x1447213a0
2016-03-15 16:43:37.718 [4349:9d27] MCOperationQueue.cpp:151: cleanup thread 0x1447213a0
2016-03-15 16:44:10.717 [4349:ab1b] MCOperationQueue.cpp:81: start thread
2016-03-15 16:44:11.767 [4349:main] MCOperationQueue.cpp:215: trying to quit 0x1447213a0
2016-03-15 16:44:11.767 [4349:ab1b] MCOperationQueue.cpp:102: stopping 0x1447213a0
2016-03-15 16:44:11.767 [4349:main] MCOperationQueue.cpp:230: thread stopped 0x1447213a0
2016-03-15 16:44:11.768 [4349:ab1b] MCOperationQueue.cpp:151: cleanup thread 0x1447213a0
```

重新进入 http://mail.163.com/ 邮箱，发现收到一封标题为 `网易邮箱提醒：阻止了一次不安全的收信请求` 的安全提醒邮件。

### 信任收信请求
可能由于调试 APP 的签名没有可行认证，被认定为非 `正规厂商的客户端`。

```
如您确认当前所用的邮件客户端为可信任客户端仍继续使用，并愿自行承担信息泄
露风险和损失，可前往（这里）设置。
```

点击 “这里” 链接，进入设置

> 网易邮箱安全提示  
>> 经检测，您正在使用的客户端来源不明。

勾选底部的

- [x] 已知悉以上信息

点击【下一步】按钮，通过手机短信验证即可信任 iOS UI Test.APP 客户端。

重新启动 iOS UI Test.APP 客户端，成功登录且拉取到邮件列表！

```
2016-03-15 16:50:49.567 iOS UI Test[4363:2557104] checking account
2016-03-15 16:50:49.569 [4363:5907] MCOperationQueue.cpp:81: start thread
2016-03-15 16:50:49.604 [4363:5907] MCIMAPSession.cpp:585: connect <mailcore::IMAPSession:0x146da7a90>
2016-03-15 16:50:57.116 [4363:5907] MCIMAPSession.cpp:608: ssl connect imap.163.com 993 2
2016-03-15 16:50:57.170 [4363:5907] MCIMAPSession.cpp:669: connect ok
2016-03-15 16:50:57.170 [4363:5907] MCIMAPSession.cpp:709: login
2016-03-15 16:50:57.468 [4363:5907] MCIMAPSession.cpp:962: login ok
2016-03-15 16:50:57.468 iOS UI Test[4363:2557104] finished checking account.
2016-03-15 16:50:57.470 [4363:5907] MCIMAPSession.cpp:1049: select
2016-03-15 16:50:58.419 [4363:5907] MCIMAPSession.cpp:1053: select error : 0
2016-03-15 16:50:58.419 [4363:5907] MCIMAPSession.cpp:1108: select ok
2016-03-15 16:50:58.424 [4363:5907] MCIMAPSession.cpp:2343: request flags
2016-03-15 16:50:58.424 [4363:5907] MCIMAPSession.cpp:2394: request envelope
2016-03-15 16:50:58.424 [4363:5907] MCIMAPSession.cpp:2416: request bodystructure
2016-03-15 16:50:59.881 iOS UI Test[4363:2557104] Progress: 1 of 2
2016-03-15 16:50:59.884 [4363:5907] MCIMAPSession.cpp:2148: parse envelope 1415701667
2016-03-15 16:50:59.899 [4363:5907] MCIMAPPart.cpp:138: attachment 1
2016-03-15 16:50:59.900 [4363:5907] MCIMAPPart.cpp:138: attachment 2
2016-03-15 16:50:59.902 iOS UI Test[4363:2557104] Progress: 2 of 2
2016-03-15 16:50:59.902 [4363:5907] MCIMAPSession.cpp:2148: parse envelope 1415701668
2016-03-15 16:50:59.903 [4363:5907] MCIMAPPart.cpp:138: attachment 1
2016-03-15 16:50:59.905 iOS UI Test[4363:2557104] fetched all messages.
2016-03-15 16:51:00.588 [4363:5907] MCIMAPSession.cpp:2818: had error : 0
2016-03-15 16:51:00.771 [4363:5907] MCIMAPSession.cpp:2818: had error : 0
2016-03-15 16:51:01.863 [4363:main] MCOperationQueue.cpp:215: trying to quit 0x146da7d60
2016-03-15 16:51:01.863 [4363:5907] MCOperationQueue.cpp:102: stopping 0x146da7d60
2016-03-15 16:51:01.863 [4363:main] MCOperationQueue.cpp:230: thread stopped 0x146da7d60
2016-03-15 16:51:01.864 [4363:5907] MCOperationQueue.cpp:151: cleanup thread 0x146da7d60
```

![iOS-UI-Tests(IMAP@163-Inbox)](iOS-UI-Tests(IMAP@163-Inbox).PNG)
