
#iOS 推送证书设置指南


## 创建应用程序 ID

* 登陆 [iOS Dev Center](https://developer.apple.com/devcenter/ios/index.action) 选择进入iOS Provisioning Portal。

![Enable push](images/ios_cert/login.png)

* 在 [iOS Provisioning Portal](https://developer.apple.com/ios/manage/overview/index.action)中，点击App IDs进入App ID列表。

![Enable push](images/ios_cert/appid.png)

* 创建 App ID，如果 ID 已经存在可以直接跳过此步骤

![Enable push](images/ios_cert/appid2.png)

* 为 App 开启 Push Notification 功能。如果是已经创建的 App ID 也可以通过设置开启 Push Notification 功能。

![Enable push](images/ios_cert/appservice.png)

* 根据实际情况完善 App ID 信息并提交,注意此处需要指定具体的 Bundle ID 不要使用通配符。

![Enable push](images/ios_cert/appid3.png)

## 配置和下载证书

* 如果你之前没有创建过 Push 证书或者是要重新创建一个新的，请在证书列表下面新建。

![Enable push](images/ios_cert/cer0.png)

* 新建证书需要注意选择证书种类（开发证书用于开发和调试使用，生产证书用于 App Store 发布）

![Enable push](images/ios_cert/cer1.png)

* 点击 Continue 后选择证书对应的应用ID，然后继续会出现“About Creating a Certificate Signing Request (CSR)”。

![Enable push](images/ios_cert/cer2.png)

* 根据它的说明创建 Certificate Signing Request。

![Enable push](images/ios_cert/cer3.png)

* 然后点击 Continue ，上传刚刚生成的 .certSigningRequest 文件 生成 APNs Push  Certificate。
* 下载并双击打开证书，证书打开时会启动“钥匙串访问”工具。
* 在“钥匙串访问”中你的证书会显示在“我的证书”中，注意选择“My Certificates” 和"login" （左上角选择`login`也就是`登录`，左下角选择`My Certificates`也就是`我的证书`。）

![Enable push](images/ios_cert/keychain_cert.png)

## 导出 .p12 证书文件

* 在“钥匙串访问”中，选择刚刚加进来的证书，选择右键菜单中的“导出“...””。

![Enable push](images/ios_cert/export_p12.png)

* 将文件保存为Personal Information Exchange (.p12)格式。

![Enable push](images/ios_cert/export_filename.png)

**保存p12文件时，必须密码为空**。

如果`Personal Information Exchange (.p12)`格式是灰色的，请确保选中`My Certificates`（我的证书）。如果还是不行，请确保是选择的是钥匙串里的`login`的证书（左上角`登录`）。或者你也可以尝试将证书文件拖拽到`My Certificates`下。

## 上传证书

在 LeanCloud的[应用设置 -> 推送设置](/app.html?appid={{appid}}#/push)里，上传上面步骤得到 .p12 证书文件。这是 iOS SDK 能够接收到 LeanCloud 推送消息的必要步骤。

![Enable push](images/ios_cert/upload_p12.png)









