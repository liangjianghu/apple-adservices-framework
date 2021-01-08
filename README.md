# AdServices



Attribute app-download campaigns that originate from the App Store, Apple News, or Stocks on iOS devices.



---



## Overview



The Apple Ads Attribution API is a solution that combines the AdServices framework on client devices and a RESTful API for server-side communication with Apple’s attribution server. The API retrieves attribution data from app downloads and redownloads from Apple Search Ads campaigns. Measure attribution data using specific Apple Search Ads campaign metadata against the performance of Apple Search Ads campaigns.

Some developers use a server-side integration with Mobile Measurement Providers (MMPs) for enhanced reporting. Developers also have the option to hand off attribution data to a MMP or manage their attribution data themselves. The following diagram illustrates using the AdServices framework in combination with a RESTful endpoint to retrieve attribution data.



![A diagram showing the sequence of interaction between the AdServices framework and RESTful API.](https://docs-assets.developer.apple.com/published/0e812f690f/rendered2x-1604339646.png)



- In step 1, the AdServices framework makes a call to request a token.
- In step 2, the AdServices framework generates a token. For more detail, see [attributionTokenWithError:](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc).
- In step 3, an MMP or developer uses the token in a RESTful API request to retrieve an attribution record from Apple’s attribution server. For more detail, see [Attribution Payload](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697463).
- In step 4, the attribution record that is returned has key value pairs that correspond to your campaigns in the Apple Search Ads Campaign Management API. For more detail, see [Attribution Payload Descriptions](https://developer.apple.com/documentation/adservices/aaattribution/3697093-attributiontokenwitherror?language=objc#3697458).



### Request Token

```objective-c
+ (NSString *)attributionTokenWithError:(NSError * _Nullable *)error;
```

The token that the framework returns is a string and has a 24-hour TTL. 



##### AAAttributionErrorCode

```objective-c
typedef enum AAAttributionErrorCode : NSInteger {
  AAAttributionErrorCodeNetworkError = 1,
  AAAttributionErrorCodeInternalError = 2,
  AAAttributionErrorCodePlatformNotSupported = 3
} AAAttributionErrorCode;
```



[`AAAttributionErrorCodeInternalError`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodeinternalerror?language=objc)

A token was not provided because an internal error occurred.

[`AAAttributionErrorCodeNetworkError`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodenetworkerror?language=objc)

A token was not provided because a network is not available.

[`AAAttributionErrorCodePlatformNotSupported`](https://developer.apple.com/documentation/adservices/aaattributionerrorcode/aaattributionerrorcodeplatformnotsupported?language=objc)

A token was not provided because the OS platform is unsupported.



### Request attribution

You can provide the token to a MMP or app developers can use the token to make a POST API call within the 24-hour TTL window to fetch attribution records. Use a single token in the request body:

```
POST https://api-adservices.apple.com/api/v1/

this_is_a_token_this_is_a_token_this_is_a_token_this_is_a_token_this_is_a_token_this_is_a_token_this_is_a_token_this_is_a_token_
```

### Response Codes
| Response                           | Description |
| --------------------------------------------------------- | --------------------------------------------------------- |
| 200 | Success. The API found a matching attribution record. The payload returns `attribution=true`. <br/>If the API doesn’t find a matching attribution record, `attribution=false`. In this case, the 200 response is an acknowledgement of the receipt of the data request.  |
| 400 | The token is invalid.  |
| 404 | Not found. The API is unable to retrieve the requested attribution record.<br/>Tokens have a TTL of 24 hours. If the POST API call exceeds 24 hours then a 404 response will result.<br/>If your token is valid, a best practice is to initiate retries at intervals of 5 seconds with a max retry of 3 attempts.  |
| 500 | The server is temporarily down or not reachable. The request may be valid, but you need to retry the request at a later point.|




### Attribution Payload

The API returns two types of attribution records: a standard response and a detailed response. The iOS 14 device level setting Allow Apps to Request to Track (AAtRtT) and details that are available from the attribution payload determine the type of response that the attribution server returns. The AAtRtT setting allows users to opt in or out of allowing apps to request user consent to access app-related data that can be used for both attribution and tracking the user or the device. The following table shows the combination of tracking interactions and expected attribution payload response.



| Allow Apps To Request to Track Setting (iOS 14 and above) | Per App Tracking Consent Status                           | AdServices Attribution Payload Response |
| --------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------- |
| On | Unknown<br/>ATTrackingManager<br/>AuthorizationStatusNot<br/>Determined | Standard                                |
|On	 | Denied or restricted<br/>ADClientErrorTracking<br/>RestrictedOrDenied | Standard|
|On | Authorized<br/>ATTrackingManager<br/>AuthorizationStatus<br/>Authorized | Detailed|
|Off  | Unknown<br/>ATTrackingManager<br/>AuthorizationStatusNot<br/>Determined | Standard|
|Off | Denied or restricted<br/>ADClientErrorTracking<br/>RestrictedOrDenied | Standard|
|Off | Authorized<br/>ATTrackingManager<br/>AuthorizationStatus<br/>Authorized | Detailed|



The attribution record is a data dictionary with key value pairs that correspond to your Apple Search Ads campaigns and app downloads on Apple News and Stocks, made from devices running iOS 14 and above.

Detailed Payload:

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



Standard Payload:

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



Run reports to review detailed campaign metadata in the [Apple Search Ads](https://developer.apple.com/documentation/apple_search_ads?language=objc) Campaigns Management API or [Apple Search Ads Advanced](https://searchads.apple.com/advanced/).



| Field | Data Type | Description |
| --------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------- |
| attribution|Boolean|Has a value of <code>true</code> if the user clicked an Apple Search Ads, News, or Stocks impression up to 30 days before an app download. If the API cannot find a matching attribution record, the attribution value will be <code>false</code>.|
|orgId|Integer|The ID of the organization that owns the campaign of which the corresponding ad was part. Obtain your <code>orgId</code> by calling <a href="/documentation/apple_search_ads/get_user_acl?language=objc">Get User ACL</a> in the Apple Search Ads Campaign Management API.  <br/>Use your <code>orgId</code> when creating campaigns and correlating to your campaigns throughout the Apple Search Ads Campaign Management API in payload requests and responses. |
|campaignId|Integer|The ID of the campaign of which the corresponding ad was part.  Use <a href="/documentation/apple_search_ads/get_a_campaign?language=objc">Get a Campaign</a> to correlate your attribution response by <code>campaignId</code>. When you create a campaign through the Apple Search Ads Campaign Management API, the API generates a <code>campaignId</code> for use in payload requests. Refer to the <a href="/documentation/apple_search_ads/campaign?language=objc">Campaign</a> object to correlate the key values in the payload responses of campaign endpoints. <br/>When you call <a href="/documentation/apple_search_ads/get_campaign_level_reports?language=objc">Get Campaign Level Reports</a>, refer to the metadata in the payload response and corresponding key value descriptions in the <a href="/documentation/apple_search_ads/reportingcampaign?language=objc">ReportingCampaign</a> object. |
|conversionType|String|The type of conversion will either be <code>newdownloads</code> or <code>redownloads</code>. Redownloads are downloads of an app by users who have previously installed the app.<br/>You will see conversion types in your campaigns reports in the Apple Search Ads Campaign Management API. |
|clickDate|Date/time string|The date and time when the user clicked an ad in a corresponding campaign. <br/>Note, this field only appears in the detailed response payload.|
|adGroupId|Integer|The ID of the ad group of which the corresponding ad was part.  When you create an ad group through the Apple Search Ads Campaign Management API, you assign it to a campaign.  <br/>Use <a href="/documentation/apple_search_ads/get_an_ad_group?language=objc">Get an Ad Group</a> to correlate your attribution response by <code>adGroupId</code> and its corresponding campaign. Refer to the <a href="/documentation/apple_search_ads/adgroup?language=objc">AdGroup</a> object to correlate the key values in the payload responses of ad group endpoints.  <br/>When you call <a href="/documentation/apple_search_ads/get_ad_group_level_reports?language=objc">Get Ad Group Level Reports</a>, refer to metadata in the payload response and its corresponding <a href="/documentation/apple_search_ads/reportingadgroup?language=objc">ReportingAdGroup</a> object for key value descriptions.|
|countryOrRegion|String|The country or region associated with the campaign that drove the install. The <code>countryOrRegion</code> field is within reporting requests in the Apple Search Ads Campaign Management API.|
|keywordId|Integer|The ID of the keyword that drove the ad impression. When you <a href="/documentation/apple_search_ads/create_targeting_keywords?language=objc">Create Targeting Keywords</a>, <a href="/documentation/apple_search_ads/create_campaign_negative_keywords?language=objc">Create Campaign Negative Keywords</a>, or <a href="/documentation/apple_search_ads/create_ad_group_negative_keywords?language=objc">Create Ad Group Negative Keywords</a>, the API generates a <code>keywordId</code> as a unique identifier for the keyword.  <br/>When adding keywords to ad groups, you set a match type as part of your campaign keyword strategy. <br/>When search match is enabled, the API doesn’t return <code>keywordId</code> in the attribution response.  <br/>Use <a href="/documentation/apple_search_ads/get_all_targeting_keywords_in_an_ad_group?language=objc">Get all Targeting Keywords in an Ad Group</a>, <a href="/documentation/apple_search_ads/get_all_campaign_negative_keywords?language=objc">Get all Campaign Negative Keywords</a>, and <a href="/documentation/apple_search_ads/get_all_ad_group_negative_keywords?language=objc">Get all Ad Group Negative Keywords</a> to correlate your attribution response by <code>keywordId</code>.  <br/>Refer to the <a href="/documentation/apple_search_ads/keyword?language=objc">Keyword</a> object for a description of match types and other key value pairs in the response payload.  <br/>Refer to metadata in the payload response for <a href="/documentation/apple_search_ads/get_keyword_level_reports?language=objc">Get Keyword Level Reports</a> and its corresponding <a href="/documentation/apple_search_ads/reportingkeyword?language=objc">ReportingKeyword</a> object for key value descriptions. |
|creativeSetId|Integer|The ID of the Creative Sets variation of which the corresponding ad was part. If a Creative Set is not specified, the API doesn’t return <code>creativeSetId</code> in the attribution response.  <br/>Use <a href="/documentation/apple_search_ads/get_a_creative_set?language=objc">Get a Creative Set</a> to correlate your attribution response by <code>creativeSetId</code>.  <br/>For more details on <code>creativeSetId</code>, see the Apple Search Ads Campaign Management API.  <br/>Refer to metadata in the payload response for <a href="/documentation/apple_search_ads/get_creative_set_level_reports?language=objc">Get Creative Set Level Reports</a> and its corresponding <a href="/documentation/apple_search_ads/reportingcreativeset?language=objc">ReportingCreativeSet</a> object for key value descriptions. |


