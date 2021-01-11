# 苹果 Apple Search Ads 全新归因框架 AdServices Framework

*更新日期：2021-01-11*



Attribute app-download campaigns that originate from the App Store, Apple News, or Stocks on iOS devices.



归因来自 Apple Search Ads 广告系列的安装（包括 App Store、Apple News 以及 Stocks）。

---



## 注意事项

- 此框架是专门用于归因 Apple Search Ads (ASA) 的安装，未来将替代 [iAd](https://developer.apple.com/documentation/iad)，而不是替代 [SKAdNetwork](https://developer.apple.com/documentation/storekit/skadnetwork)
- 接入此框架，无论用户是否授权许可“应用追踪透明度框架（ATT）”，来自 Apple Search Ads 的安装均可被正确归因。
- 涉及到 app 内代码实施，完成此框架接入后 app 需发布新版本
- 获取令牌必须在 app 内实施，获取归因数据包可以在 app 内或者 server 端实施
- 采用三方 SDK 的开发者，与您的服务商沟通，并关注其 SDK 版本更新动态
- 自行实施归因 API 的开发者，可以当前开始实施该 API
  - 注意版本兼容性，此框架仅支持 iOS 14.3 以上版本。旧版本还需使用 [iAd归因框架](https://developer.apple.com/documentation/iad)
- 令牌（Token）的有效时间为 24 小时，拿到后需及时请求归因数据
- 对于经验较少的开发人员，需注意网络请求的成功率和容错性



## Overview 概要

The Apple Ads Attribution API is a solution that combines the AdServices framework on client devices and a RESTful API for server-side communication with Apple’s attribution server. The API retrieves attribution data from app downloads and redownloads from Apple Search Ads campaigns. Measure attribution data using specific Apple Search Ads campaign metadata against the performance of Apple Search Ads campaigns.

苹果广告归因 API 是一种解决方案，结合了客户端设备上的 AdServices 框架和用于与 Apple 的归因服务器进行服务器端通信的 RESTful API。该 API 从来自 Apple Search Ads 活动带来的应用下载及重新下载中检索归因数据。归因数据使用给定的 Apple Search Ads 广告系列元数据来衡量效果。



Some developers use a server-side integration with Mobile Measurement Providers (MMPs) for enhanced reporting. Developers also have the option to hand off attribution data to a MMP or manage their attribution data themselves. The following diagram illustrates using the AdServices framework in combination with a RESTful endpoint to retrieve attribution data.

一些开发者使用服务器端与移动归因服务商（MMP）的集成来增强报告功能。开发者可以选择将归因数据交给 MMP ，也可以自己管理。下图说明了结合使用 AdServices 框架和 RESTful API 来检索归因数据。

![A diagram showing the sequence of interaction between the AdServices framework and RESTful API.](https://docs-assets.developer.apple.com/published/0e812f690f/rendered2x-1604339646.png)



- In step 1, the AdServices framework makes a call to request a token.
- In step 2, the AdServices framework generates a token. For more detail, see [attributionTokenWithError:](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc).
- In step 3, an MMP or developer uses the token in a RESTful API request to retrieve an attribution record from Apple’s attribution server. For more detail, see [Attribution Payload](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697463).
- In step 4, the attribution record that is returned has key value pairs that correspond to your campaigns in the Apple Search Ads Campaign Management API. For more detail, see [Attribution Payload Descriptions](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697458).



* 第一步，AdServices 框架发起调用请求生成token；
* 第二步，AdServices 框架生成 token。更多详细信息，参考[attributionTokenWithError:](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc) ；
* 第三步，MMP 或开发人员使用 token 发起 RESTful API 请求，苹果的归因服务器响应请求，并从返回的数据中检索归因数据。更多详细信息，参考 [Attribution Payload](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697463) ；
* 第四步，返回的归因数据为字段格式的键值对，这些键值对数据与 Apple Search Ads 广告系列管理API中的广告系列相对应。更多详细信息，参考  [Attribution Payload Descriptions](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697458) 。



### Request Token 请求token

```objective-c
+ (NSString *)attributionTokenWithError:(NSError * _Nullable *)error;
```

The token that the framework returns is a string and has a 24-hour TTL. 

AdServices 框架返回的 token 为字符串类型，并且只有 **24 小时**有效期。



##### AAAttributionErrorCode 归因错误枚举值

```objective-c
typedef enum AAAttributionErrorCode : NSInteger {
  AAAttributionErrorCodeNetworkError = 1,
  AAAttributionErrorCodeInternalError = 2,
  AAAttributionErrorCodePlatformNotSupported = 3
} AAAttributionErrorCode;
```



[`AAAttributionErrorCodeInternalError`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodeinternalerror?language=objc)

A token was not provided because an internal error occurred.

没有返回 token，发生了内部错误。

[`AAAttributionErrorCodeNetworkError`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodenetworkerror?language=objc)

A token was not provided because a network is not available.

没有返回 token，网络不可用。

[`AAAttributionErrorCodePlatformNotSupported`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodeplatformnotsupported?language=objc)

A token was not provided because the OS platform is unsupported.

没有返回 token，操作系统平台不支持。



### Request attribution 发起归因请求

You can provide the token to a MMP or app developers can use the token to make a POST API call within the 24-hour TTL window to fetch attribution records. Use a single token in the request body:

您可以将 token 提供给 MMP，或者使用该 token 在 24 小时内进行 POST API 调用，以获取归因数据。在请求正文中带上 token：

```bash
POST https://api-adservices.apple.com/api/v1/

yourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtokenyourtoken
```

### Response Codes 返回状态码

| Response状态码                        | Description描述 |
| --------------------------------------------------------- | --------------------------------------------------------- |
| 200 | Success. The API found a matching attribution record. The payload returns `attribution=true`. <br/>If the API doesn’t find a matching attribution record, `attribution=false`. In this case, the 200 response is an acknowledgement of the receipt of the data request.<br/>成功。API 找到了匹配的归因记录，返回值包含 `attribution=true`。<br/>如果API没有找到对应的归因记录，返回值为 `attribution=false`。在这种情况，状态码 200 仅表示服务器有数据返回。 |
| 400 | The token is invalid.<br/>token 无效。 |
| 404 | Not found. The API is unable to retrieve the requested attribution record.<br/>Tokens have a TTL of 24 hours. If the POST API call exceeds 24 hours then a 404 response will result.<br/>If your token is valid, a best practice is to initiate retries at intervals of 5 seconds with a max retry of 3 attempts.<br/>没有找到。API 无法获取到请求的归因记录。<br/>Tokens 只有 24 小时的有效期。如果请求超过了 24 小时，苹果会返回 404 状态码。<br/>如果 token 是有效的，一个最佳实践：最多重试 3 次，每次间隔 5 秒钟。 |
| 500 | The server is temporarily down or not reachable. The request may be valid, but you need to retry the request at a later point.<br/>服务器暂时关闭或无法访问。请求可能是有效的，但是你需要选择合适的时间点进行重试。 |

### 示例

#### 获取token：

```objective-c
#import <AdServices/AdServices.h>

- (void) methodToGetToken {
    if (@available(iOS 14.3, *)) {
        NSError *error;
        NSString *token = [AAAttribution attributionTokenWithError:&error];
        if (token != nil) {
          // 发送POST请求归因数据
        }
    } else {
        // Fallback on earlier versions
    }
}
```



#### 获取归因数据

##### Objective-C 请求示例

```objective-c
- (void)attributionWithToken:(NSString *)token {
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];
    NSURL *url = [NSURL URLWithString:@"https://api-adservices.apple.com/api/v1/"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url
                                                           cachePolicy:NSURLRequestUseProtocolCachePolicy
                                                       timeoutInterval:60.0];
    [request addValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setHTTPMethod:@"POST"];
    NSData* postData = [token dataUsingEncoding:NSUTF8StringEncoding];
    [request setHTTPBody:postData];
    NSURLSessionDataTask *postDataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSError *resError;
        NSMutableDictionary *resDic = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:&resError];
    }];
    [postDataTask resume];
}
```

##### cURL 请求示例：

```bash
curl -vvv https://api-adservices.apple.com/api/v1/ \
-H 'Content-Type:application/json'\
-d 'yourtokenyourtokenyourtokenyourtoken'

```

##### 返回归因数据包示例：

```json
{
	"orgId":1234567890,
	"campaignId":1234567890,
	"conversionType":"Download",
	"clickDate":"2021-01-11T06:41Z",
	"adGroupId":1234567890,
	"countryOrRegion":"US",
	"keywordId":12323222,
	"creativeSetId":1234567890,
	"attribution":true
}
```



### Attribution Payload 归因信息

The API returns two types of attribution records: a standard response and a detailed response. The iOS 14 device level setting Allow Apps to Request to Track (AAtRtT) and details that are available from the attribution payload determine the type of response that the attribution server returns. The AAtRtT setting allows users to opt in or out of allowing apps to request user consent to access app-related data that can be used for both attribution and tracking the user or the device. The following table shows the combination of tracking interactions and expected attribution payload response.

API 返回两种类型的归因记录：标准响应和详细响应。 iOS 14 设备级别的设置允许应用程序请求跟踪（AAtRtT）以及归因负载中可用的详细信息，决定了归因服务器返回的响应类型。 AAtRtT 设置允许用户选择启用或退出允许应用程序请求用户同意访问与应用程序相关的数据，这些数据可用于归因和跟踪用户或设备。下表显示了跟踪交互和预期的归因负载响应的组合。

| Allow Apps To Request to Track Setting (iOS 14 and above)<br/>允许应用程序请求跟踪设置 | Per App Tracking Consent Status<br/>单个应用程序跟踪同意状态 | AdServices Attribution Payload Response<br/>AdServices 归因数据包 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| On                                                           | Unknown<br/>ATTrackingManager<br/>AuthorizationStatusNot<br/>Determined | Standard<br/>标准数据包                                      |
| On                                                           | Denied or restricted<br/>ADClientErrorTracking<br/>RestrictedOrDenied | Standard<br/>标准数据包                                      |
| On                                                           | Authorized<br/>ATTrackingManager<br/>AuthorizationStatus<br/>Authorized | Detailed<br/>详细数据包                                      |
| Off                                                          | Unknown<br/>ATTrackingManager<br/>AuthorizationStatusNot<br/>Determined | Standard<br/>标准数据包                                      |
| Off                                                          | Denied or restricted<br/>ADClientErrorTracking<br/>RestrictedOrDenied | Standard<br/>标准数据包                                      |
| Off                                                          | Authorized<br/>ATTrackingManager<br/>AuthorizationStatus<br/>Authorized | Detailed<br/>详细数据包                                      |



The attribution record is a data dictionary with key value pairs that correspond to your Apple Search Ads campaigns and app downloads on Apple News and Stocks, made from devices running iOS 14 and above.

归因记录是一个包含键值对的数据字典，这些数据与来自 Apple Search Ads（包括 App Store、Apple News 以及 Stocks）的设备（由运行 iOS 14 及更高版本的设备）安装行为相对应。

##### Detailed Payload（详细数据包）

```json
{
  "attribution": true,
  "orgId": 55555,
  "campaignId": 12345678,
  "conversionType": "Download",
  "clickDate": "2020-10-01T17:17Z",
  "adGroupId": 12345678,
  "countryOrRegion": "US",
  "keywordId": 12345678,
  "creativeSetId": 12345678
}
```

##### Standard Payload（标准数据包）

```json
{
  "attribution": true,
  "orgId": 55555,
  "campaignId": 12345678,
  "conversionType": "Download",
  "adGroupId": 12345678,
  "countryOrRegion": "US",
  "keywordId": 12345678,
  "creativeSetId": 12345678
}
```



#### 归因数据包字段说明

| Field<br/>**字段** | Data Type<br/>**类型** | Description<br/>**说明** |
| --------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------- |
| attribution|Boolean|Has a value of <code>true</code> if the user clicked an Apple Search Ads, News, or Stocks impression up to 30 days before an app download. If the API cannot find a matching attribution record, the attribution value will be <code>false</code>.<br/>如果用户在应用下载前 30 天点击了 App Store、Apple News 以及 Stocks，则其值为 `true`。如果 API 找不到匹配的归因记录，则为`false`。|
|orgId|Integer|The ID of the organization that owns the campaign of which the corresponding ad was part.<br/>广告系列所属的账户 ID。 |
|campaignId|Integer|The ID of the campaign of which the corresponding ad was part.<br/>广告系列 ID。 |
|conversionType|String|The type of conversion will either be <code>newdownloads</code> or <code>redownloads</code>. Redownloads are downloads of an app by users who have previously installed the app.<br/>表明是否首次下载。"Redownload" 说明用户在本设备下载/卸载过，或者用同一账户在其他设备下载过。 |
|clickDate|Date/time string|The date and time when the user clicked an ad in a corresponding campaign. <br/>Note, this field only appears in the detailed response payload.<br/>用户点击相应广告的日期和时间。此字段仅出现在详细归因数据包中。|
|adGroupId|Integer|The ID of the ad group of which the corresponding ad was part. <br/>广告组 ID。|
|countryOrRegion|String|The country or region associated with the campaign that drove the install. <br/>国家或地区。|
|keywordId|Integer|The ID of the keyword that drove the ad impression.<br/>关键字的 ID。 |
|creativeSetId|Integer|The ID of the Creative Sets variation of which the corresponding ad was part. <br/>广告素材集的 ID。 |



## 参考

- [AdServices Framework - Apple Developer Documentation](https://developer.apple.com/documentation/adservices?language=objc)
- [AppsFlyer SDK 6.1.3 +](https://support.appsflyer.com/hc/en-us/articles/360016711377/ ) 已支持
- Adjust
- Branch
- Kochava
- Singular
- Tenjin


