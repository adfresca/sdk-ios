## Contents
- [Introduction](#introduction)
- [Quick Start](#quick-start)
    - [Installation](#installation)
    - [Code](#code)
- [Test Device ID](#test-device-id) 
- [Custom Parameter](#custom-parameter)
- [Marketing Event](#marketing-event)
- [In-App-Purchase Count](#in-app-purchase-count) 
- [Push Notification](#push-notification)
- [Custom URL](#custom-url)
- [CPI Identifier](#cpi-identifier)
- [Reward Item](#reward-item)
- [Advanced Features](#advanced-features)
    - [AdFrescaViewDelegate](#adfrescaviewdelegate) 
    - [Timeout Interval](#timeout-interval) 
    - [IFV Only Option](#ifv-only-option)
- [Trouble Shooting](#trouble-shooting)
- [Release Notes](#release-notes)

* * *

## Introduction

AD fresca는 게임 운영자나 마케터가 앱 내 사용자 특성을 실시간으로 파악하여  더 자주, 더 오래 플레이하고, 더 많이 결제하도록 유도하는 라이브 서비스 운영 툴을 제공합니다

게임 운영자나 마케터는 [Dashboard](https://admin.adfresca.com) 사이트를 통해 실시간으로 타겟팅한 사용자에게 메시지를 전달할 수 있으며, 이를 실제 앱에 적용하기 위하여 게임 개발팀에서는 아래 제공되는 SDK를 손쉽게 설치하고 가이드에 따라 코드를 적용합니다.

* * *

## Quick Start

### Installation

아래 링크를 통해 SDK 파일을 다운로드 합니다.

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) (v1.3.5)

SDK를 프로젝트에 추가하기 위해 아래의 절차가 필요합니다.

1. 제공되는 AdFresca 폴더를 Xcode 프로젝트에 Drag & Drop 하여 추가합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2. System Configuration.framework, StoreKit.framework, AdSupport.framework(선택)를 Xcode 프로젝트에 추가합니다.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - AdSupport.framework를 추가할 경우, SDK는 [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) 값을 수집하여 디바이스(=앱 사용자) 구분에 사용합니다. AD fresca SDK는 IFA 값을 사용하여 크로스 프로모션 캠페인 기능을 제공하고 캠페인 노출 이후 사용자의 앱 설치 및 액션 트랙킹을 위해 사용하고 있습니다. 
  - AdSupport.framework를 제외할 경우, [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) 값을 사용합니다. 이 경우 크로스 프로모션 캠페인 기능을 이용할 수 없으며 IFV의 특성상 사용자가 앱을 삭제하고 재설치할 때 새로운 디바이스(=앱 사용자)로 인식될 수 있습니다. 

  만약, 앱 업데이트 과정에서 AdSupport.framework를 제외하거나 새로 추가하는 경우 [IFV Only Option](#ifv-only-option) 항목의 내용을 참고하여 주시기 바랍니다.

3. Build Setting의 Other Linker Flags 값을 –ObjC로 설정 혹은 추가합니다. 

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4. Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정합니다. (Push Notification 적용 시 반드시 확인해주시기 바랍니다.)

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  만약 앱이 가로 방향만을 지원한다면 'Initial interface orientation' 값을 'Landscape (right home button)' 으로 설정합니다.

아무런 에러 없이 빌드가 성공헀다면 모든 설치가 정상적으로 완료된 것입니다. 만약 Duplicate Symbol 등의 Linking Error 가 발생하였다면 아래의 '[Trouble Shooting](#trouble-shooting)' 항목을 확인해주시기 바랍니다

### Code

AD fresca SDK 통해 사용자에게 메시지를 전달하기 위한 주요 코드를 적용합니다. 아래의 코드만으로도 게임 운영자 / 마케터가 지정한 캠페인의 콘텐츠를 화면에 표시하고, 푸시 메시지를 전송할 수 있습니다.

```objective-c
// AppDelegate.m
#import <AdFresca/AdFrescaView.h>
 
// 앱이 최초로 실행되는 이벤트에서 API KEY 설정을 합니다.

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];

  [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];   // Push Notification 기능을 등록     
} 

// 아래와 같이 코드를 적용하여 매칭되는 캠페인의 콘텐츠를 내려받고 표시합니다.

- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca loadAd]; 
  [fresca showAd]; 
} 

// Push Notification 기능을 사용하기 위해 아래 코드를 삽입합니다. 자세한 내용은 '[Push Notification](#push-notification)' 항목을 참고해 주세요.

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [AdFrescaView registerDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
    [AdFrescaView handlePushNotification:userInfo];
  }
}
```

`[AdFrescaView startSession:@"YOUR_API_KEY"];` API Key 를 설정하며 앱의 시작을 알립니다. API Key는 [Dashboard](https://admin.adfresca.com) 사이트에서 앱 추가 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다. 이 메소드는 반드시 didFinishLaunchingWithOptions 이벤트에서 실행합니다.

`[fresca loadAd];` 서버로부터 매칭되는 캠페인의 콘텐츠를 내려받습니다. 

`[fresca showAd];` 내려받은 콘텐츠를 화면에 표시합니다.

앱이 실행 되면 다음과 같은 화면이 보여집니다. 정상적으로 콘텐츠 뷰가 화면에 표시되고, 터치 시 앱스토어 페이지로 이동하는지 확인합니다.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

* * *

## Test Device ID

AD fresca는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 지정한 캠페인의 콘텐츠를 화면에 표시하고 푸시 메시지를 전송할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.
 
1. testDeviceId Property를 사용하여 로그로 출력하는 방법
  - 테스트에 사용할 기기를 개발PC에 연결한 후 로그를 통해 해당 아이디 값을 출력하여 확인 합니다. 

2. printTestDeviceId Property를 설정하여 뷰에 기기 아이디를 화면에 표시하는 방법
  - 개발자가 기기를 직접 연결할 수 없는 경우, 설정을 활성화 한 상태로 앱 빌드를 전덜하여 설치합니다. 화면에 표시된 기기 아이디를 직접 기록하여 등록할 수 있습니다.
  - 담당 마케터가 원격에서 근무하는 경우 해당 기능을 유용하게 사용할 수 있습니다.
  - 설정이 활성화된 상태로 앱이 배포되지 않도록 주의해야 합니다.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];
NSLog(@"AD fresca Test Device ID = %@", fresca.testDeviceId);  // 로그로 기기 ID를 출력. 
fresca.printTestDeviceId = YES; // 콘텐츠 뷰에 테스트 기기 ID가 함께 표시되도록 설정
[fresca loadAd];
[fresca showAd];
```

아이디 값을 확인한 후 [Dashboard](https://admin.adfresca.com) 사이트에서 테스트 기기를 등록하고, 테스트 모드 기능을 이용하기 위해서는 [테스트 기기 및 테스트 모드 관리하기](https://adfresca.zendesk.com/entries/21921047) 가이드 내용에 따라 작업을 진행합니다.

* * *

## Custom Parameter

커스텀 파라미터는 캠페인 진행 시, 타겟팅을 위해 사용할 사용자의 상태 값을 의미합니다.

AD fresca SDK는 기본적으로 '국가, 언어, 앱 버전, 실행 횟수 등'의 디바이스 고유 데이터를 수집하며, 동시에 각 앱 내에서 고유하게 사용되는 특수한 상태 값들(예: 캐릭터 레벨, 보유 포인트, 스테이지 등)을 커스텀 파라미터로 정의하고 수집하여 분석 및 타겟팅 기능을 제공합니다.

커스텀 파라미터 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 커스텀 파라미터의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

Integer, Boolean 형태의 데이터를 상태 값으로 설정할 수 있으며, *setCustomParameterWithValue** 메소드를 사용하여 각 인덱스 값에 맞게 상태 값을 설정합니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];                    
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.stage] forIndex:CUSTOM_PARAM_INDEX_STAGE];
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forIndex:CUSTOM_PARAM_INDEX_FACEBOOK];   
}

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];  // 사용자 level 정보를 가장 최신으로 업데이트 
}   
```

**주의:** 모든 커스텀 파라미터의의 초기 설정은 항상 didFinishLaunchingWithOptions 이벤트에서 적용하며, 이후에 값이 변경되는 경우 각 위치에서 업데이트 합니다.

만약 불가피하게 didFinishLaunchingWithOptions() 이벤트에서 커스텀 파라미터 값을 설정할 수 없는 경우, 앱을 최초로 실행한 사용자의 프로파일은 업데이트되지 않으며 해당 사용자의 2회째 앱 실행부터 SDK가 로컬에 캐싱해둔 값이 전달됩니다. 최초로 실행된 사용자의 프로파일까지 통계 및 타겟팅하기 위해서는 아래와 같이 초기 값 설정을 진행합니다. 또한, 사용자의 로그인 이벤트 이후 모든 커스텀 파라미터의 값을 설정할 수 있도록 구현합니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  if (isUserFirstRun) {
    [fresca setCustomParameterWithValue:[NSNumber numberWithInt:defaultLevel] forIndex:CUSTOM_PARAM_INDEX_LEVEL];                    
    [fresca setCustomParameterWithValue:[NSNumber numberWithInt:defaultStage] forIndex:CUSTOM_PARAM_INDEX_STAGE];
    [fresca setCustomParameterWithValue:[NSNumber numberWithBool:defaultFacebookFlag] forIndex:CUSTOM_PARAM_INDEX_FACEBOOK];  
  } 
}

// 유저 로그인 성공

- (void)userDidSignIn {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];                    
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.stage] forIndex:CUSTOM_PARAM_INDEX_STAGE];
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forIndex:CUSTOM_PARAM_INDEX_FACEBOOK];   
}   
```

* * *

## Marketing Event

마케팅 이벤트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 이벤트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 이벤트 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Events 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 이벤트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 이벤트 발생 시, loadAd() 메소드에 원하는 이벤트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(loadAd() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

```objective-c
- (void)viewDidLoad {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca loadAd:EVENT_INDEX_MAIN_PAGE]; // 메인 페이지에 설정한 캠페인 노출       
  [fresca showAd];
} 

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forIndex:CUSTOM_PARAM_INDEX_LEVEL];  // 사용자 level 정보를 가장 최신으로 업데이트 
  [fresca loadAd:EVENT_INDEX_LEVEL_UP]; // 레벨업 이벤트에 설정한 캠페인 노출  
  [fresca showAd];
}  
```

* * *

## In-App Purchase Count

앱에서 IAP 기능을 사용하는 경우, 현재까지 사용자가 구매한 누적 횟수를 SDK에 설정하여 분석 및 타겟팅에 이용할 수 있습니다.

**numberOfInAppPurchases** Property 값을 설정하여 현재까지 사용자가 구매한 누적 횟수 값을 SDK에 설정합니다. 커스텀 파라미터와 마찬가지로 앱 실행 혹은 사용자 로그인 이후에 값을 지정하고, IAP 결제가 일어난 직후에 갱신된 누적 구매 횟수 값을 설정합니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ......

  AdFrescaView *fresca = [AdFrescaView sharedAdView];  
  fresca.numberOfInAppPurchases = user.numberOfInAppPurchases; 
  ......
}

- (void)userDidPurchase:(int)numberOfTotalInAppPurchases {
  user.numberOfInAppPurchases = numberOfTotalInAppPurchases;

  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  fresca.numberOfInAppPurchases = user.numberOfInAppPurchases; 
}  
```

* * *

## Push Notification

AD fresca를 통해 Push Notification 메시지를 보내고 받을 수 있습니다.

SDK를 적용하기 이전에 애플의 [Local and Push Notification Programming Guide](http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008194-CH1-SW1) 가이드 문서를 읽어보시길 권장합니다. 

(현재 AD fresca의 Push Notification 서비스는 APNS의 Production 환경만을 지원하며, 추후 업데이트를 통해 Development 환경을 추가로 지원할 예정입니다.)

1. Push Notification 인증서 파일을 생성하고 Dashboard 사이트에 등록합니다.
  - [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드 를 따라 Production용 Push Notification Certificate를 생성하고 [Dashboard](https://admin.adfresca.com) 사이트에 등록합니다.

2. Info.plast 확인하기 / Provision 확인하기
  - Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정합니다. 
  - App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드해야 합니다.

3. AppDelegate 클래스의 이벤트 추가하기.
  - AppDelegate.m 파일을 열어 아래와 같은 내용을 추가합니다.
  ```objective-c
  #import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AdFrescaView startSession:@"YOUR_API_KEY"];
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];   // Push Notification 기능을 이용할 경우 등록.      
  } 

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Push Notification을 수신 받기 위한 고유 토큰을 AD fresca에 등록합니다.
    [AdFrescaView registerDeviceToken:deviceToken];
  }

  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    // AD fresca를 통해 전달된 Notification만 사용하며 앱이 이미 실행 중인 경우, 무시합니다.
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      // 만약 Push Notification에 URL Schema가 설정되어 있을 경우, 현재 위치에서 URL을 실행합니다.
      [AdFrescaView handlePushNotification:userInfo];
    }  
  } 
  ```

* * *

## Custom URL

Announcement 캠페인의 Click URL, Push Notification 캠페인의 URL Schema 설정 시에 자신의 앱 URL Schema를 사용할 수 있습니다.

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

## CPI Identifier

Incentivized CPI 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://admin.adfresca.com) 사이트에서의 설정 방법은 [Incentivized CPI 캠페인 관리하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

#### Advertising App 설정하기:

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  (현재 Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 URL Schema 설정만 진행되면 됩니다. 하지만 이후 지원할 CPA 캠페인을 위해서 미리 SDK를 설치하는 것을 권장합니다.)

  Xcode 프로젝트의 Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다. 
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 CPI 캠페인을 위해서는 최대한 Unique한 값을 선택해야 합니다.

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Reward Item](#reward-item) 항목의 내용을 구현합니다.

* * *

## Reward Item

Reward Item 기능을 적용하여 현재 사용자에게 지급 가능한 보상 아이템이 있는지 검사하고, 보상 아이템을 사용자에게 지급할 수 있습니다.

Annoucnement 캠페인의 'Reward Item' 항목을 설정했거나, Incentivized CPI 캠페인의 'Incentive Item' 을 설정한 경우 사용자에게 보상 아이템이 지급됩니다.

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

Incentivized CPI 캠페인의 경우는 사용자의 앱 설치가 확인된 후 itemRewarded 이벤트가 발생하며, Annoucnement 캠페인의 경우는 캠페인이 앱 사용자에게 매칭되어 노출될 때 itemRewarded 이벤트가 발생합니다. 만일 디바이스의 네트워크 단절이 발생한 경우 SDK는 데이터를 로컬에 보관하여 다음 앱 실행에서 아이템 지급이 가능하도록 구현되어 있기 때문에 항상 100% 지급을 보장합니다.

(기존의 getAvailableRewardItems 메소드는 Deprecated 상태로 변경되었지만, 호환성을 보장하여 정상적으로 동작하고 있습니다.)

* * *

## Advanced Features

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
  [fresca loadAd];
  [fresca showAd];
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

loadAd() 메소드의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 데이터가 로딩되지 못한 경우, 사용자에게 콘텐츠를 노출하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];  
fresca.timeoutInterval = 3 // # secs  
[fresca loadAd];
[fresca showAd];
```

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

## Trouble Shooting
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

- **v1.3.5 (2014/04/06 Updated)**
  - SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [Installation](#installation) 항목을 참고하여 주세요.
  - Announcement 캠페인을 통한 Reward Item 지급 기능을 지원합니다. 
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
  - Event 기능을 지원합니다. loadAd() 메소드에 Event Index 값을 설정할 수 있습니다. 자세한 내용은 '7. Event 지정하기'를 참고해주세요.
  - AD Slot 기능이 Deprecated 되었습니다. 기존의 Default Slot은 '1'번 이벤트 인덱스,  AD Only Slot은 '2'번 이벤트 인덱스로 적용됩니다.
  - 콘텐츠 데이터를 요청하는 중에 새로 loadAd()가 호출된 경우, 가장 최근에 요청된 콘텐츠 화면에 표시됩니다. (기존에는 콘텐츠 데이터를 요청 중에 새 요청을 할 수 없었습니다.)
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
  - loadAd() 메서드에서 에러가 발생 시, frescaClosed 이벤트가 강제로 발생하던 문제를 해결 하였습니다. frescaClosed 이벤트는 항상 showAd 메서드가 호출된 이후에 발생 됩니다.
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
