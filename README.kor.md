## Contents
- [Basic Integration](#basic-integration)
    - [Installation](#installation)
    - [Start Session](#start-session)
    - [In-App Messaging](#in-app-messaging)
    - [Push Messaging](#push-messaging)
    - [Test Device Registration](#test-device-registration)
- [IAP & Reward](#iap--reward)
  - [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta)
  - [Give Reward](#give-reward)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [AdFrescaViewDelegate](#adfrescaviewdelegate) 
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Custom URL Schema](#custom-url-schema)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [IFV Only Option](#ifv-only-option)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

아래 링크를 통해 SDK 파일을 다운로드 합니다.

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) 

SDK를 프로젝트에 추가하기 위해 아래의 절차가 필요합니다.

1) 제공되는 AdFresca 폴더를 Xcode 프로젝트에 Drag & Drop 하여 추가합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2) System Configuration.framework, StoreKit.framework, AdSupport.framework(선택)를 Xcode 프로젝트에 추가합니다.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - AdSupport.framework를 추가할 경우, SDK는 [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) 값을 수집하여 디바이스(=앱 사용자) 구분에 사용합니다. AD fresca SDK는 IFA 값을 사용하여 크로스 프로모션 캠페인 기능을 제공하고 캠페인 노출 이후 사용자의 앱 설치 및 액션 트랙킹을 위해 사용하고 있습니다. 
  - AdSupport.framework를 제외할 경우, [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) 값을 사용합니다. 이 경우 크로스 프로모션 캠페인 기능을 이용할 수 없으며 IFV의 특성상 사용자가 앱을 삭제하고 재설치할 때 새로운 디바이스(=앱 사용자)로 인식될 수 있습니다. 

  만약, 앱 업데이트 과정에서 AdSupport.framework를 제외하거나 새로 추가하는 경우 [IFV Only Option](#ifv-only-option) 항목의 내용을 참고하여 주시기 바랍니다.

3) Build Setting의 Other Linker Flags 값을 –ObjC로 설정 혹은 추가합니다. 

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4) Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정합니다. (Push Notification 적용 시 반드시 확인해주시기 바랍니다.)

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  만약 앱이 가로 방향만을 지원한다면 'Initial interface orientation' 값을 'Landscape (right home button)' 으로 설정합니다.

  마지막으로, URL Scheme 값을 지정합니다. 아래의 예제는 'myapp' 이라는 스키마 값을 지정한 예제입니다. 해당 값은 크로스 프로모션 기능을 이용하기 위하여 사용됩니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

아무런 에러 없이 빌드가 성공헀다면 모든 설치가 정상적으로 완료된 것입니다. 만약 Duplicate Symbol 등의 Linking Error 가 발생하였다면 아래의 '[Troubleshooting](#troubleshooting)' 항목을 확인해주시기 바랍니다

### Start Session

이제 SDK 적용을 시작하기 위해 몇 가지 간단한 코드를 적용합니다. 첫 번째로 API Key를 설정하고 앱의 실행을 기록하는 startSession() 메소드를 적용합니다. API Key는 [Dashboard](https://admin.adfresca.com) 사이트에서 등록한 앱을 선택한 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다.

startSession() 메소드를 적용하면 앱이 최초로 실행되거나, 백그라운드에서 재실행될 때 자동으로 앱의 실행을 기록합니다.

```objective-c
// AppDelegate.m
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 
```

### In-App Messaging

인-앱 메시징 기능을 이용하여, 사용자에게 원하는 메시지를 실시간으로 전달할 수 있습니다. 메시지를 전달하고자 하는 시점에 load(), show() 메소드만을 호출하여 적용이 가능합니다. 메시지는 전면 interstitial 이미지, 텍스트, 혹은 iframe 웹페이지 형태로 화면에 표시될 수 있습니다. 메시지는 현재 플레이 중인 사용자가 인-앱 메시징 캠페인의 조건과 매칭된 경우에만 화면에 표시됩니다. 조건에 만족하는 캠페인이 없다면 사용자는 아무런 화면을 보지 않고 자연스럽게 플레이를 이어갑니다. 매칭과 관련한 인-앱 메시징의 다이나믹 타겟팅 기능은 아래의 [Dynamic Targeting](#dynamic-targeting) 항목에서 보다 자세히 설명하고 있습니다.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca load]; 
  [fresca show]; 
} 
```

첫 번째로 인-앱 메시징 코드를 적용한 경우, 아래와 같이 테스트 이미지 메시지가 표시됩니다. 해당 이미지를 터치하면 앱스토어 페이지로 이동합니다. 현재 보고 있는 테스트 메시지는 이후 테스트 모드 설정을 변경하여 더이상 보이지 않도록 설정하게 됩니다.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

푸시 메시징 기능을 이용하여 사용자가 앱을 실행하지 않을 때에도 언제든 메시지를 전달할 수 있습니다. 아래의 과정을 통하여 푸시 메시징 기능을 적용합니다.

1) APNS 인증서 파일(.p12)을 Dashboard에 등록하기
  - Keychain 툴을 이용하여 .cer 인증서 파일을 .p12로 변환하고 [Dashboard](https://admin.adfresca.com) 사이트에 등록합니다.
  - 보다 자세한 설명은 [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드를 통하여 확인이 가능합니다.

2) Info.plast 확인하기 / Provision 확인하기
- AD fresca는 APNS의 Production 환경만을 지원합니다. 때문에 빌드가 production으로 빌드되어야 정상적인 서비스 이용이 가능합니다.
- Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정되어 있어야 합니다.
- App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드되어야 합니다.

3) AppDelegate 코드 적용하기 

  ```objective-c
  #import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ....
    if ([application respondsToSelector:@selector(registerUserNotificationSettings:)]) {
      UIUserNotificationType types = (UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound);
      UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:types categories:nil];
      [application registerUserNotificationSettings:notificationSettings];
      [application registerForRemoteNotifications];
    } else {
      [application registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound)];
    }

    NSDictionary* userInfo = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
	if (userInfo != nil) {
	  [self application:application didReceiveRemoteNotification:userInfo];
	}
  } 
  
  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Register user's push device token to our SDK
    [AdFrescaView registerDeviceToken:deviceToken];
  }

  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    /// Check a push notification is form AD fresca. Also, ignore a notification received when app is already running 
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }  
  } 
  ```

### Test Device Registration

AD fresca는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 원하는 메시지를 전달할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달하여 테스트할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.
 
1. testDeviceId Property를 사용하여 로그로 출력하는 방법
  - 테스트에 사용할 기기를 개발PC에 연결한 후 로그를 통해 해당 아이디 값을 출력하여 확인 합니다. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  NSLog(@"AD fresca Test Device ID = %@", fresca.testDeviceId); 
  [fresca load];
  [fresca show];
```

2. printTestDeviceId Property를 설정하여 뷰에 기기 아이디를 화면에 표시하는 방법
  - 개발자가 기기를 직접 연결할 수 없는 경우, 설정을 활성화 한 상태로 앱 빌드를 전덜하여 설치합니다. 화면에 표시된 기기 아이디를 직접 기록하여 등록할 수 있습니다.
  - 담당 마케터가 원격에서 근무하는 경우 해당 기능을 유용하게 사용할 수 있습니다.
  - 설정이 활성화된 상태로 앱이 배포되지 않도록 주의해야 합니다.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca load];
  [fresca show];
  ```

테스트 디바이스 아이디를 확인한 이후에는, [Dashboard](https://admin.adfresca.com)를 접속하여 'Test Device' 메뉴를 통해 디바이스 등록이 가능합니다.

* * *

## IAP & Reward

### In-App Purchase Tracking (Beta)

_In-App-Purchase Tracking_ 기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

AD fresca의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Actual Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Virtual Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

#### Actual Item Tracking

Actual Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. iOS의 경우 Storekit 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다.

적용 예제: 
```objective-c
- (void)completeTransaction:(SKPaymentTransaction *)transaction
{
  // productInfo is NSMutableDictionary object to store fetched SKProduct object with its productIdentifier as a key.
  SKProduct *product = [productInfo objectForKey:transaction.payment.productIdentifier];

  NSString *itemId = transaction.payment.productIdentifier;
  NSString *currencyCode = [product.priceLocale objectForKey:NSLocaleCurrencyCode];
  NSNumber *price = product.price;
  NSDate *transactionDate = transaction.transactionDate;
  NSData *transactionReceiptData = transaction.transactionReceipt;

  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeActualItem
                                                    itemId:itemId
                                              currencyCode:currencyCode
                                                     price:[price doubleValue]
                                              purchaseDate:transactionDate
                                    transactionReceiptData:transactionReceiptData];

  [[AdFrescaView shardAdView] logPurchase:purchase];
  ......
}
```

Actual Item을 위한 AFPurchase 객체 생성의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
itemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
currencyCode(string) | ISO 4217 표준 코드를 설정합니다. SKProduct 객체의 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
price(double) | 아이템의 가격을 설정합니다. SKProduct 객체의 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
purchaseDate(date) | 결제된 시간을 NSDate 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
transactionReceiptData(nsdata| SKPaymentTransaction 객체의 transactionReceipt 값을 지정합니다. 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 

#### Virtual Item Tracking

Virtual Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다.

적용 예제: 
```objective-c
- (void)didPurchaseVirtualItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeVirtualItem
                                                    itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil]; 

  [[AdFrescaView shardAdView] logPurchase:purchase];
}
```

Virtual Item을 위한 AFPurchase 객체 생성의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
itemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
currencyCode(string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
price(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
purchaseDate(date) | 결제된 시간을 NSDate 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
transactionReceiptData(nsdata| Virtual 아이템의 경우는 값을 지정하지 않습니다.

#### IAP Trouble Shooting

logPurchase() 메소드를 통해 기록된 AFPurchase 객체는 AD fresca 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, AFPurchaseDelegate를 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. 

만약 AFPurchase 객체의 값이 제대로 설정되지 않은 경우, didFailToLogWithException 이벤트를 통하여 에러 메시지를 표시하고 있으니 아래와 같이 코드를 적용하여 로그를 확인합니다.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPurchaseDelegate> {
  ...
}

// AppDelegate.m
- (void)didPurchaseVirtualItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeVirtualItem
                                               itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil];
  [[AdFrescaView shardAdView] logPurchase:purchase, self];
}

- (void)purchase:(AFPurchase *)purchase didFailToLogWithException:(AdFrescaException *)exception {
  NSLog(@"AFPurchase didFailToLogWithException :: purchase = %@, exception = %@", [purchase JSONRepresentation], [exception description]);
}
```

* * *

### Give Reward

Reward Item 기능을 적용하여 현재 사용자에게 지급 가능한 보상 아이템이 있는지 검사하고, 보상 아이템을 사용자에게 지급할 수 있습니다.

Annoucnement 캠페인의 'Reward Item' 항목을 설정했거나, Incentivized CPI & CPA 캠페인의 'Incentive Item' 을 설정한 경우 사용자에게 보상 아이템이 지급됩니다.

SDK 적용을 위해서는 아래 2가지 코드를 이용합니다.
- checkRewardItems 메소드 호출: 현재 지급 가능한 보상 아이템이 있는지 검사합니다. 사용자가 앱을 실행할 호출하는 것을 권장합니다.
- AFRewardItemDelegate 구현: 아이템 지급 조건이 만족되면 itemRewarded 이벤트가 발생됩니다. 인자로 넘어온 아이템 정보를 이용하여 사용자에게 아이템을 지급합니다.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFRewardItemDelegate> {
  ...
}

```

```objective-c
// AppDelegate.m

- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setRewardDelegate:self];
  [fresca checkRewardItems];
}

- (void)itemRewarded:(AFRewardItem *)item {
  NSString *logMessage = [NSString stringWithFormat:@"You got the reward item! (%@)", item.name];
  NSLog(@"%@", logMessage);
  
  // 아이템 고유 값 'uniqueValue'을 이용하여 사용자에게 아이템 지급
  [self sendItemToUser:item.uniqueValue];
}
```
캠페인 종류에 따라 itemRewarded 이벤트의 발생 조건이 다릅니다.

- Annoucnement 캠페인: 캠페인이 앱 사용자에게 매칭되어 노출될 때 이벤트가 발생합니다
- Incentivized CPI 캠페인: 사용자의 Advertising App 설치가 확인된 후 이벤트가 발생합니다.
- Incentivized CPA 캠페인: 사용자의 Advertising App 설치가 확인되고 보상 조건으로 지정된 마케팅 이벤트가 호출된 후에 발생합니다.

만일 디바이스의 네트워크 단절이 발생한 경우 SDK는 데이터를 로컬에 보관하여 다음 앱 실행에서 아이템 지급이 가능하도록 구현되어 있기 때문에 항상 100% 지급을 보장합니다.

(기존의 getAvailableRewardItems 메소드는 Deprecated 상태로 변경되었지만, 호환성을 보장하여 정상적으로 동작하고 있습니다.)

* * *

## Dynamic Targeting

### Custom Parameter

커스텀 파라미터는 캠페인 진행 시, 타겟팅을 위해 사용할 사용자의 상태 값을 의미합니다.

AD fresca SDK는 기본적으로 '국가, 언어, 앱 버전, 실행 횟수 등'의 디바이스 고유 데이터를 수집하며, 동시에 각 앱 내에서 고유하게 사용되는 특수한 상태 값들(예: 캐릭터 레벨, 보유 포인트, 스테이지 등)을 커스텀 파라미터로 정의하고 수집하여 분석 및 타겟팅 기능을 제공합니다.

커스텀 파라미터 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 커스텀 파라미터의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

Integer, Boolean 형태의 데이터를 상태 값으로 설정할 수 있으며, **setCustomParameterWithValue** 메소드를 사용하여 각 인덱스 값에 맞게 상태 값을 설정합니다.

앱이 실행되는 시점에 한 번 값을 설정하고, 이후에는 값이 새로 갱신되는 이벤트마다 새로운 값을 설정합니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{
  ...
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];                    
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.stage] forIndex:CUSTOM_PARAM_INDEX_STAGE];
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forIndex:CUSTOM_PARAM_INDEX_FACEBOOK];   
}

- (void)levelDidChange:(int)level 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];
}   

- (void)stageDidChange:(int)stage 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:stage] forIndex:CUSTOM_PARAM_INDEX_STAGE];
}
....
```

특정 커스텀 파라미터의 경우, 사용자의 로그인 작업 이후 설정이 가능할 수 있습니다. 해당 경우는 사용자의 로그인 이후에 필요한 커스텀 파라미터를 모두 설정할 수 있도록 합니다.

* * *

### Marketing Moment

마케팅 모멘트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 모멘트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 모멘트 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Moments 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 모멘트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 모멘트 발생 시, load() 메소드에 원하는 모멘트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(load() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

```objective-c
- (void)userDidEnterItemStore {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca load:EVENT_INDEX_STORE_PAGE];    
  [fresca show];
} 

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forIndex:CUSTOM_PARAM_INDEX_LEVEL]; 
  [fresca load:EVENT_INDEX_LEVEL_UP]; 
  [fresca show];
}  
```

## Advanced

### AdFrescaViewDelegate
AdFrescaViewDelegate 를 직접 구현함으로써, 콘텐츠 뷰에서 발생하는 이벤트를 확인 할 수 있습니다. 

```objective-c
// ViewController.h
@interface MainViewController : UIViewController<AdFrescaViewDelegate> {
  .......
@end

// ViewController.m

- (void)viewDidLoad {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.delegate = self;
  [fresca load];
  [fresca show];
}

#pragma mark – AdFrescaViewDelegate

// 콘텐츠를 요청하기 직전에 호출됩니다.
- (void)frescaWillReceiveAd:(AdFrescaView *)theAdView {}

// 콘텐츠를 정상적으로 불러온 후 발생하는 이벤트입니다.
- (void)frescaDidReceiveAd:(AdFrescaView *)theAdView {}

// 콘텐츠를 불러오지 못한 경우 발생됩니다. 에러 정보를 확인 할 수 있습니다.
- (void)fresca:(AdFrescaView *)view didFailToReceiveAdWithException:(AdException *)error {}

// 사용자가 뷰를 종료한 이후 발생하는 이벤트입니다. 콘텐츠 불러오지 못해 에러가 발생한 경우에도 해당 이벤트가 발생됩니다.
- (void)frescaClosed:(AdFrescaView *)fresca {}
```

위의 이벤트 메소드 내용을 직접 구현함으로써 다양한 응용이 가능해집니다. 

예를 들면

- 앱의 인트로 화면에서 콘텐츠를 표시한 후, 사용자가 콘텐츠 뷰를  닫으면 메인 페이지로 이동하고 싶은 경우
- 게임 도중 ‘Next Stage” 버튼을 눌러 콘텐츠를 표시한 후, 사용자가 콘텐츠를  닫으면 스테이지가 넘어가는 경우  
위 경우는 frescaClosed 함수 내용을 구현함으로써 적용이 가능합니다.

```objective-c
// Example: FirstViewController.m
#pragma mark – AdFrescaViewDelegate

- (void)frescaClosed:(AdFrescaView *)fresca {
  // 다음 페이지로 이동

  NextViewController *vc = [[NextViewController alloc] init];
  [self.navigationController pushViewController:vc animated:YES];  
  [vc release];
}
```

주의사항:

사용자가 마켓이나 다른 애플리케이션의 URI가 설정된 콘텐츠를 클릭한 경우, 화면이 다른 애플리케이션으로 이동할 수 있습니다. 
이 때 frescaClosed 에 다른 페이지로 이동하도록 구현하였다면, 사용자가 다른 화면에 있는 동안 앱의 페이지가 가 미리 이동해버리거나, 페이지 애니메이션이 비정상적으로 실행될 수 있습니다.
아래와 같은 방법으로 해당 문제를 해결할 수 있습니다.

1. Dashboard 에서 해당 Event 의 Close Mode 를 Override 로 변경 합니다.(콘텐츠 이미지를 클릭해도 뷰가 닫히지 않습니다..)
2. AppDelegate의 applicationWillEnterForeground() 이벤트를 아래와 같이 구현합니다.

```objective-c
#pragma mark – AdFrescaViewDelegate

- (void)applicationWillEnterForeground:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView shardAdView];
  if (!fresca.hidden && fresca.userClicked) {
    [fresca closeAd];
  }
}
```

### Timeout Interval

load() 메소드의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 데이터가 로딩되지 못한 경우, 사용자에게 콘텐츠를 노출하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];  
fresca.timeoutInterval = 3 // # secs  
[fresca load];
[fresca show];
```

* * *

## Reference

### Deep Link

캠페인의 Click URL 설정 시에 Custom URL Schema를 지정할 수 있습니다.

이를 통해 사용자가 콘텐츠를 클릭할 경우, 자신이 원하는 특정 앱 페이지로 이동하는 등의 액션을 지정할 수 있습니다.

1. Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2. AppDelegate.m 파일을 열어 handleOpenURL 메소드를 구현합니다. 호출되는 URL 값에 따라 다른 페이지를 호출하도록 설정할 수 있습니다. 
  ```objective-c
  - (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {  
    if ([url.scheme isEqualToString:@"myapp"]) {
      if ([url.host isEqualToString:@"item"]) {
        ItemViewController *vc = [[ItemViewController alloc] init];
        [navigationController pushViewController:vc animated:YES];
        [vc release];
        return YES;
      }
    }
    return NO;
  }
```
  위와  같이 구현한 경우, 캠페인의 Click URL을 'myapp://item' 으로 설정하여 전송하면, ItemViewController 페이지가 실행됩니다.

* * *

### Cross Promotion Configuration

Incentivized CPI & CPA 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI & CPA 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://admin.adfresca.com) 사이트에서의 설정 방법은 [크로스 프로모션 이해하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

#### Advertising App 설정:

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  (현재 Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 URL Schema 설정만 진행되면 됩니다. 하지만 Incentivized CPA 캠페인을 진행할 경우 반드시 SDK 설치 및 [Marketing Moment](#marketing-moment) 기능이 적용되어야 합니다.)

  먼저 Xcode 프로젝트의 Info.plst 파일을 열어 사용할 URL Schema 정보를 확인합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다.
  
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 캠페인 진행을 위해서는 고유한 값을 사용해야 합니다.

  마지막으로, Incentivized CPA 캠페인을 진행할 경우는 보상 조건으로 지정한 마케팅 모멘트가 발생되어야 합니다. 사용자가 보상 조건을 완료한 이후 아래와 같이 지정한 마케팅 모벤트 코드를 실행합니다.
  
  ```objective-c
  // 튜토리얼 완료 모멘트를 보상 조건으로 지정한 경우
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca load:MOMENT_INDEX_TUTORIAL];     
  [fresca show];
  ```

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Give Reward](#give-reward) 항목이 적용되어 있어야 합니다. 앞서 리워드 지급 기능을 적용하지 않았다면, 가이드에 따라 기능을 적용합니다.

* * *

### IFV Only Option

[SDK 설치 과정](#installation)에서 AdSupport.framework 를 추가한 경우 SDK는 IFA 값을 이용하여 디바이스를 구분하며, AdSupport.framework를 제외한 경우 IFV 값을 이용하여 디바이스를 구분하게 됩니다.

만약 앱스토어 출시 이후 앱을 업데이트 하는 과정에서 AdSupport.framework를 제외시키거나, 추가하는 경우 아래와 같은 상황이 발생합니다.

1. 기존에 사용하던 AdSupport.framework를 제외시키는 경우:
  - AD fresca API 서버는 기존에 함께 수집한 IFV 값을 이용하여, 기존의 앱 사용자들이 새로운 사용자로 인식 되지 않도록 자동으로 처리합니다. 따라서 아무런 문제가 발생하지 않습니다. 단, iOS SDK 1.3.3 (2013년 11월 26일 출시) 이상의 버전이 탑재되었던 앱에 한해서만 처리됩니다.
2. AdSupport.framework를 새로 추가하는 경우:
  - 이 경우는 SDK가 기존에 IFA 값을 수집하지 못하였기 때문에, 앱을 그대로 릴리즈하면 기존 사용자들이 모두 새로운 사용자로 인식되는 문제가 발생합니다. iOS SDK에서는 이 문제를 해결하기 위하여 **setUseIFVOnly** 메소드를 제공합니다. didFinishLaunchingWithOptions 이벤트에서 **setUseIFVOnly** 메소드에 'YES' 값을 설정하면 AdSupport.framework가 추가되었더라도 기존의 IFV 값을 가지고 디바이스를 구분하도록 합니다. 이로 인해 기존의 IFV 값을 이용하여 등록된 사용자들이 새로운 사용자로 인식되는 것을 방지할 수 있습니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [AdFrescaView startSession:API_KEY];
  [[AdFrescaView shardAdView] setUseIFVOnly:YES];
}
```

위 내용이 제대로 확인되지 않을 시에는 사용자 통계나 타겟팅 기능에 큰 문제를 일으킬 수 있습니다. 개발팀에서는 해당 내용을 자세히 확인하고 대응해야 하며, 보다 자세한 문의는 support@adfresca.com 메일을 통해 연락을 부탁드립니다.

* * *

## Troubleshooting

SDK 설치시에 SBJson의 Duplicate Symbol 에러가 발생하여 빌드가 되지 않을 수 있습니다.

<img src="https://adfresca.zendesk.com/attachments/token/ikafbcqjnj9jbak/?name=6666.png">

위와 같은 에러가 발생하며 빌드가 실패하게 됩니다.

현재 개발 중인 프로젝트내에 이미 SBJson을 사용중인 경우에 발생할 수 있으며, AdFresca SDK에 포함된 SBJson을 제거함으로써 해결이 가능합니다. 현재 SDK에 포함된 SBJson은 [3.1 release](https://github.com/stig/json-framework/tree/v3.1) 버전이며, 프로젝트에서 이보다 하위 버전을 사용할 시에 문제가 발생할 수 있습니다.

그 외에 콘텐츠가 제대로 출력되지 않거나, 에러가 발생한다면 AdFrescaViewDelegate의 didFailToReceiveAdWithException 이벤트 함수를 구현하여, 에러 정보를 확인 할 수 있습니다. 

```objective-c
- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes

- v1.4.2
	- 1개의 마케팅 모멘트에서 복 수 개의 캠페인이 매칭되어 표시가 가능하도록 변경되었습니다.
- v1.4.1
  - Xcode의 64-bit 아키텍쳐 설정을 지원합니다.
  - 1.4.0-beta에서 지원하는 [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta) 기능을 통합하여 제공합니다.
  - 몇몇 메소드의 이름이 변경되었습니다. (load -> load, show -> show, closeAd -> close) 기존에 제공하던 메소드도 호환성을 위하여 정상적으로 지원합니다.
- 1.4.0-beta1
  - 앱 내에서 발생하는 In-App Purchase 데이터를 트랙킹할 수 있는 기능이 추가되었습니다. 자세한 내용은 [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta) 항목을 참고하여 주세요. [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta) 항목을 참고하여 주세요.
- v1.3.5
  - SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [Installation](#installation) 항목을 참고하여 주세요.
  - Announcement 캠페인을 통한 Reward Item 지급 기능을 지원합니다. 
  - Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요.
  - AFRewardItemDelegate가 구현 기능이 추가되어, 지급 가능한 아이템이 발생할 시에 자동으로 itemRewarded 이벤트가 발생합니다. 보다 자세한 내용은 [Reward Item](#reward-item) 항목을 참고하여 주세요.
- v1.3.4
  - testDeviceId property 값이 각 iOS  버전에 맞는 값으로 출력되도록 변경되었습니다. 
- v1.3.3 
  - APNS 디바이스 토큰이 새로 생성되거나 변경 시, SDK가 토큰 값을 실시간으로 AD fresca  서비스에 업데이트하도록 개선되었습니다. (기존에는 앱 실행 시에만 업데이트하였습니다.)
- v1.3.2 
  - 커스텀 파라미터 설정 시 'long long' 타입까지 확장하여 지원합니다.
customParameterWithIndex 호출 시 설정된 값이 없는 경우 nil 값을 리턴하도록 변경되었습니다.
- v1.3.1 
  - 'Close Mode' 기능을 지원합니다. Dashboard에서 Interstitial View의 닫힘 설정을 제어할 수 있습니다.
  - In-App-Purchase Count, Custom Parameter 정보를 로컬에 캐싱하여 사용합니다. 
  - iOS 4.3 버전에서 가로모드의 뷰가 비정상적으로 표시되는 문제를 해결하였습니다.
  - 커스텀 파라미터 설정 시 long 타입을 지원합니다.
- v1.3.0 
  - Reward 아이템 지급 기능을 지원합니다. '10. Reward Item 지급하기' 항목을 참고하여 주세요.
- v1.2.1
  - numberOfInAppPurchases 설정 값이 추가 되었습니다. In-App Purchase를 구매한 횟수를 관리 할 수 있습니다. (적용 방법은 5. In-App Purchased Count 관리 항목 참고하여 주세요.)
  - isInAppPurchasedUser property가 Deprecated 되었습니다. 새로 추가된 numberOfInAppPurchases property를 사용하여 주세요.
- v1.2.0
  - Event 기능을 지원합니다. load() 메소드에 Event Index 값을 설정할 수 있습니다. 자세한 내용은 '7. Event 지정하기'를 참고해주세요.
  - AD Slot 기능이 Deprecated 되었습니다. 기존의 Default Slot은 '1'번 이벤트 인덱스,  AD Only Slot은 '2'번 이벤트 인덱스로 적용됩니다.
  - 콘텐츠 데이터를 요청하는 중에 새로 load()가 호출된 경우, 가장 최근에 요청된 콘텐츠 화면에 표시됩니다. (기존에는 콘텐츠 데이터를 요청 중에 새 요청을 할 수 없었습니다.)
  - 앱스토어로 연결되는 콘텐츠 경우, 앱스토어 페이지를 앱 안에서 표시합니다. 더이상 앱 밖으로 나가지 않습니다. (해당 기능을 위해서 반드시 StoreKit. framework를 추가하여 주세요.)
  - 콘텐츠 이미지 클릭 시 콘텐츠 뷰가 닫히도록 변경되었습니다.
  - 콘텐츠를 일정 시간 후 자동으로 닫을수 있는 Auto Close Timer 기능이 추가 되었습니다. Dashboard 에서 설정할 수 있습니다.
- v1.1.0 
  - Push Notification 기능이 적용되었습니다. 자세한 내용은 '8. Push Notification 설정하기'를 참고해주세요.
- v1.0.1 
  - Custom Parameter를 지원합니다.  (자세한 내용은 'Custom Parameter 관리하기'를 참고해주세요)
- v1.0.0
  - HTML5 형태의 View를 지원합니다. (SDK 적용 코드는 전혀 변경하지 않아도 됩니다.)
  - iOS6에서 추가된 'Advertising Identifier'를 추가로 수집 및 사용합니다. 이와 관련하여 AdSupport framework를 Xcode 프로젝트에 추가해야 합니다. (자세한 내용은 'SDK 설치'를 참고해주세요)
- v0.9.9
  - iOS 6  정식 버전 및 iPhone 5 모델을 지원합니다.
- v0.9.8
  - 테스트 모드 기능 지원을 위한 테스트 기기 ID 확인 기능을 지원 합니다. (자세한 내용은 '테스트 기기 ID 확인하기'를 참고해 주세요)
- v0.9.7
  - 공지사항 기능이 추가 되면서 AD Slot 관리 기능이 추가 되었습니다. (자세한 내용은 'AD Slot 관리하기' 를 참고해 주세요)
  - load() 메서드에서 에러가 발생 시, frescaClosed 이벤트가 강제로 발생하던 문제를 해결 하였습니다. frescaClosed 이벤트는 항상 show 메서드가 호출된 이후에 발생 됩니다.
  - 캐시 기능 및 퍼포먼스가 향상 되었습니다.
  - 몇몇 메서드 이름의 오타를 수정하였습니다. (sharedAdView, frescaClosed) SDK의 호환성 유지를 위하여 잘못된 이름의 메서드는 삭제되지 않았으며 추후 Depreciated 설정 될 예정 입니다.
- v0.9.6 
  - 콘텐츠 캐싱 기능이 향상 되었습니다.
- v0.9.5 
  - SDK가 콘텐츠 데이터를 캐싱하여 보여 줍니다. 콘텐츠를 1회 이상 노출 시 캐시가 자동으로 적용되어 빠른 노출이 가능하여 졌습니다.
  - timeoutInterval 설정 값이 추가 되었습니다. 지정된 시간 내에 데이터를 로딩하지 못한 경우, 사용자에게 콘텐츠를 노출하지 않습니다. 최소 1초 이상 지정이 가능하며 기본 값은 기존의 5초로 설정 됩니다.
  - testModeEnabled 설정 값이 deprecated 되었습니다. 이후 모든 테스트 모드의 제어는 웹 Admin 페이지에서 가능합니다.
  - AdFrescaViewDelegate의 required 메소드 목록이 변경 되었습니다.
  - iOS 6 버전을 지원합니다.
- v0.9.4
  - startSession: 메소드가 추가 되었습니다. 보다 정확한 세션로깅을 위해 startSession 메소드를 didFinishLaunchingWithOptions 델리게이트 메소드에 구현해 주세요. (적용 방법은 4. Session Logging 항목 참고)
  - isInAppPurchasedUser 설정 값이 추가 되었습니다. In-App Purchase를 구매한 사용자들을 분류하여 관리 할 수 있습니다. (적용 방법은 5. In-App Purchased User 관리 항목 참고)
- v0.9.3
  - 세션 로깅 기능을 지원 합니다.  SDK가 자동으로 앱의 실행 이벤트를 감지하여 세션 정보를 기록 합니다. 
- v0.9.2
  - Performance가 개선 되었습니다.
  - 콘텐츠 클릭 시 앱스토어 이동 관련하여 일부 발생하던 버그를 수정 하였습니다.
- v0.9.1
  - UI가 개선 되었습니다.
- v0.9.0
  - AD fresca iOS SDK가 출시 되었습니다. 기본적인 콘텐츠 출력 기능이 포함 됩니다.
