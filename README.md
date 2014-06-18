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

 Download SDK on the following link:

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) (v1.4.1)

To add SDK into your Xcode project, please follow the instructions below:

1. Drag & Drop AdFresca folder into the framework folder on your Xcode project.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2. Add System Configuration.framework and AdSupport.framework, StoreKit.framework into your target if these frameworks are not added yet.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - If you add AdSupport.framework, SDK collects [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distinguish the user's device. We use this value to provide the cross-promotion campaign with install and action tracking.
  - If you do not add AdSupport.framework, SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distinguish user's device. In this case, you can't use any cross promotion feature. Also, as IFV's policy, your user may be recognized as a new user after re-installing app.

  If you'd like to add AdSupport.framework or remove the framework from existing xcode project with our SDK, please refer to the [IFV Only Option](#ifv-only-option) section to migrate your users.

3. Add -ObjC to Other Linker Flag on your target's build setting.

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4. In Info.plst, set 'aps-environment' value as 'production'. It is necessary to use a push notification feature.

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  Also, set your own URL Scheme value. the example below shows how to set URL Scheme with "myapp" value. It will be used in the cross promotion feature.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

AD fresca SDK has been successfully installed without any build error. If you have a 'Duplicate Symbol' error, please refer to the [Troubleshooting](#troubleshooting) section.

### Start Session

Now, start to put some simple SDK codes in your app. You first need to call startSession() method with your API Key. To get your API Key, go to our [Dashboard](https://admin.adfresca.com) and then click 'Settings - API Keys' button in your app's 'Overview' page.

startSession() will start to detect when user starts app and resumes from the background.

```objective-c
// AppDelegate.m
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 
```

### In-App Messaging

With the in-app messaging feature, you can deliver a message to your in-app users in real time. Simply put 'loadAd' and 'showAd' methods where you want to deliver a message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when your user matches the in-app messaging campaign's target logics. We will discuss more details of the in-app messaging's dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca loadAd]; 
  [fresca showAd]; 
} 
```

When you first call in-app messaging methods, you will see the test message below. If you tap on the image, it will redirect to the product page of the app on the app store. You will hide this test message by chagning the test mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can also deliver your push messages anytime you want. Follow the steps below to configure the push notification settings in your app.

1. Upload your APNS Certificate file (.p12) to our Dashboard
  - You can export your .cer file to .p12 file using Keychain. Please refer [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/21714780) to generate .p12 and upload to [Dashboard](https://admin.adfresca.com)

2. Check your provisioning
  - AD fresca only supports APNS production environment. So, you should build your app with App Store or Ad Hoc Provisioning file to enable production mode

3. Add some codes to AppDelegate 
  ```objective-c
  #import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ....
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];   // Push Notification 기능을 이용할 경우 등록.      
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

AD fresca supports a test mode feature. With the test mode feature, you can deliver your test message to only registred test devices. 

To register your test device to our dashboard, you need to know your test device ID from our SDK. SDK provodies two ways to show test device ID.
 
1. Using testDeviceId Property
  - After connecting your device with Xcode, you can simply print out test device ID with a logger.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  NSLog(@"AD fresca Test Device ID = %@", fresca.testDeviceId); 
  [fresca loadAd];
  [fresca showAd];
```

2. Displaying test device ID on your app screen using printTestDeviceId property
  - When you are not able to connect tester's device in your office, you have to set printTestDeviceId to true, and then let them install this app build. They can see their own test device ID on the app screen. 
  - It is useful when testers are working remotely. 
  - printTestDeviceId property must be set to false when you distribute your app on the store. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca loadAd];
  [fresca showAd];
  ```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). You can register your device in the 'Test Device' menu.

* * *

## IAP & Reward

### In-App Purchase Tracking (Beta)

With In-App-Purchase Tracking , you can analyze all the purchases of your users, and use it for targeting specific user segment to display your campaigns. (targeting feature is coming soon)

There are two types of purchases you can track with our SDK.

1. **Actual Item Purchase Tracking:**  the purchases made by real money. For example, user purchased 'USD 1.99' to get 'Gold 100' cash item.
2. **Virtual Item Purchase Tracking:** the purchases made by virtual money. For example, user purchased 'Gold 10' to get 'Rocket Launcher' item 

You don't need to write down any item list manually. All the Items tracked by SDK are automatically added to our dashboard. To see the list of item, go to 'Overview > Settings > In-App Items' page in our dashboard.

Let's get started to implement SDK codes with examples below. 

#### Actual Item Tracking

In iOS, the purchase of 'Actual Item' is made with Apple's Storekit framework. When your user purchased the item successfully, simply create AFPurchase object and use logPurchase() method.

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
}
```

For more details of AFPurchase object with the actual item , check the table below.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
currencyCode(string) | Set the current code of IOS 4217 standard. You may use SKProduct's's value or manually set the value form your server.
price(double) | Set the item price. you may use SKProduct's value or manually set the value from your server.
purchaseDate(date) | Set the date of purchase. You may use SKPaymentTransaction.transactionDate value. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.
transactionReceiptData(nsdata) | Set the receipt property of SKPaymentTransaction object. We will use it to verify the receipt in the future.

#### Virtual Item Tracking

When users purchased your virtual item in the app, you can also create AFPurchase object and call logPurchase() method.

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

For more details of AFPurchase object with the virtual item, check the table below.. You don't need to set transactionReceiptData property in the virtual item tracking.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
currencyCode(string) | Set the item's virtual currency code. (ex: 'gold', 'gas')
price(double) | Set the item price. You may get this value from your server. (ex: 100 of gold)
purchaseDate(date) | Set the date of purchase. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.
transactionReceiptData(nsdata) | Set nil for AFPurchaseTypeVirtualItem

#### IAP Troubleshooting

After you call logPurchase() method, the purchase data is updated to our dashboard in real-time. You can see the list of updated item in 'Overview > Settings > In-App Items' menu.

If you can't see any data in our dashboard, your AFPurchase object may be invalid. To check it, you can implement  AFPurchaseDelegate and call log logPurchase(purchase, delegate) method. 

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

When you set 'Reward Item' section of the announcement campaign or 'Inventive item' section of the incentivized CPI & CPA campaign, you should implement this 'reward item' code to give an reward item to your users.

Implementing reward item codes, you can check if your user has any reward to receive, and then will be noticed with an reward item info.

To implement codes, we use two codes below:

- checkRewardItems method: this method is to check if any item is available to receive. we recommend to put this code when app becomes active. 
- AFRewardItemDelegate implementation: when the reward condition is completed with current user, itemRewarded event is automatically called with AFRewardItem object from our SDK. you can give an item to the user with AFRewardItem object.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFRewardItemDelegate> {
  ...
}


// AppDelegate.m

- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setRewardDelegate:self];
  [fresca checkRewardItems];
}

- (void)itemRewarded:(AFRewardItem *)item 
{
  NSString *logMessage = [NSString stringWithFormat:@"You got the reward item! (%@)", item.name];
  NSLog(@"%@", logMessage);
  
  // Using uniqueValue property, you can give an item to users.  
  [self sendItemToUser:item.uniqueValue];
}
```

You will implement your own 'sendItemToUser' method. This method may send the current user info and item's uniqueValue to your server. Then server gives the item to the user.

itemRewarded event is called when each type of campaign's reward condition is completed.

- Announcement Campaign: the event is called when your user see the campaign contents
- Incentivized CPI Campaign: the event is called when SDK checks Advertising App's install
- Incentivized CPA Campaign: the event is called after SDK checks Advertising App's install and the user called the targeted marketing event in Advertising App
 
If your users have any network disconnection or loss in theirs device, our SDK stored the reward data in the app's local storage, and then re-check in the next app session. So, we guarantee users will always get a reward from our SDK.

(getAvailableRewardItems is deprecated, but we still support this method for backward compatibility)

* * *

## Dynamic Targeting

### Custom Parameter

Our SDK can collect user specific profiles such as level, stage, maximum score and etc. We use it to deliver a personalized and targeted message in real time to specific user segment that you can define.

To implement codes, simply call setCustomParameterWithValue method with passing parameter's index and value. You can get the custom parameter's index in our [Dashboard](https://admin.adfresca.com): 1) Select a App 2) In 'Overview' menu, click 'Settings - Custom Parameters' button.

You will call the method after your app is launched and the values have changed. 

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

In some cases, you may not able to set some custom parameters in didFinishLaunchingWithOptions event since you may need to get the values from you server. If so, you will need to set the custom parameters right after the user sign in.

* * *

### Marketing Moment

Marketing Moment means the moment you want to engage with your users. For example, you may need to deliver the message when the user completes a quest or enters an item store. You will be able to use it with the [custom parameters](#custom-parameter) so you can deliver the personalized and targeted message in specific moment in real time.

To implement codes, simply call loadAd method with passing marketing moment's index. You can get the marketing moment's index in our [Dashboard](https://admin.adfresca.com): 1) Select a App 2) In 'Overview' menu, click 'Settings - Marketing Moment' button. 

You will call the method after the moment has happened in the app.

```objective-c
- (void)userDidEnterItemStore {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca loadAd:EVENT_INDEX_STORE_PAGE];    
  [fresca showAd];
} 

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forIndex:CUSTOM_PARAM_INDEX_LEVEL]; 
  [fresca loadAd:EVENT_INDEX_LEVEL_UP]; 
  [fresca showAd];
}  
```

## Advanced

### AdFrescaViewDelegate

With implementing AdFrescaViewDelegate in your code, you can check all the events on the SDK 

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

//This event occurs when SDK will start to receive contents
- (void)adViewWillReceiveAd:(AdFrescaView *)theAdView {}

//This event occurs when SDK receive contents successfully
- (void)adViewDidReceiveAd:(AdFrescaView *)theAdView {}

// This event occurs when content was not received properly with error information.
- (void)adView:(AdFrescaView *)view didFailToReceiveAdWithException:(AdException *)error {}

// This event occurs when the user closes view. it's also called when content is not loaded with an error after calling showAd(). So this is endpoint of our process
- (void)adViewClosed:(AdFrescaView *)adView {}
```

There are many good practices implementing AdFrescaViewDelegate.

1. Scenarios 1: Display overlay contents in the bootup screen
  - Display Boot-up screen (Logo, etc)
  - Contents will be displayed over the boot-up screen
  - If the user closes the view, the main page will be loaded.
2. Scenarios 2: Insert Contents between each stages
  - When the user touch 'Next Stage' button on your game, Contents will be displayed.
  - If the user closes the view and the page will be redirected to the next stage.
  - In these case, you need to use adViewClosed()

```objective-c
// Example: FirstViewController.m
#pragma mark – AdFrescaViewDelegate
- (void)adViewClosed:(AdFrescaView *)adView {
  // Move to the next page
  NextViewController *vc = [[NextViewController alloc] init];
  [self.navigationController pushViewController:vc animated:YES];  
  [vc release];
}
```

Caution:

If user clicked contents that opens App Store or other applications, user will leave our of your app screen.

In this case, if you implemented adViewClosed() event like above example, user may see unnatural paging animation since app was temporarily paused by another application.

To fix this issue, follow the steps below:
  1. In dashboard, you should change 'Close mode' to 'Override' in your marketing moment settings. (it will prevent to close view when user clicked)
  2. In app codes, Implement  applicationWillEnterForeground() event of AppDelegate like below:

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

You can set a timeout interval for marketing moment request. If the message is not loaded within this time interval, the message won't be displayed to users and SDK will return the control to your app.

Default is 5 seconds and you can set from 1 seconds to 5 seconds.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];  
fresca.timeoutInterval = 3 // # secs  
[fresca loadAd];
[fresca showAd];
```

* * *

## Reference

### Custom URL Schema

You can set your own URL Schema as 'Click URL' of the campaigns. So, you can navigate your users to the specific page or do some custom actions when user clicked the image message. 

1. Set your custom url schemes in Info.plst as follows

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2. In AppDelegate.m, implement handleOpenURL method. You may call a new app view controller depending on the url.
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

In this example, ItemViewController will be displayed when you set the campaign click url as 'myapp://item'

* * *

### Cross Promotion Configuration

IUsing Incentivized CPI & CPA Campaign, your users in 'Media App' can get an incentive item when they install 'Adverting App' from the campaigns.

- Medial App: the media app which displays the promotion image and gives an incentive item to users
- AdvertisingApp: the promotion app which is displayed with an image in the media app's screen.

For more details of Incentivized campaigns and configuration guide in dashboard, please refer 'Understanding Cross-promotion (Korean)'  guide.

To integrate SDK with this feature, you should set URL Schema value for the adverting app and implement codes to give an incentive item to users in the media app.

#### Configuration for Advertising App.:

  Check your url schemes to check app install in Info.plst as follows

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  In this case, you should set CPI Identifier value of advertising app to "myapp://" in our dashboard.

  For iOS, url schema value may be duplicated with other apps, so be careful to choose unique value.

  For Incentivized CPI Campaign, SDK Installation of the advertising app is not required. You can only set URL Schema to use app's install.

  However, If you use Incentivized CPA Campaign, SDK installation is required and you should also implement 'Marketing Event' feature to check a reward condition. For example, when you set the reward condition to check 'Tutorial Complete' event, you should call the marketing event method to inform your user achieved the goal.
    
  ```objective-c
  - (void)didTutorialComplete {
    AdFrescaView *fresca = [AdFrescaView sharedAdView];   
    [fresca loadAd:EVENT_INDEX_TUTORIAL_COMPLETE];  
    [fresca showAd];
  }  
  ```

#### SDK implementation for Media App:

  To give an incentive item to the media app's users, please refer to the [Give Reward](#give-reward) section.

* * *

### IFV Only Option

As [SDK Installation](#installation) describes, SDK uses [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distinguish the user's device if you add AdSupport.framework. In other hands, SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distinguish user's device if you do not add AdSupport.framework.

If you are adding the framework or remove it while you're updating your app which already exists in the app store. The following issues may happen to you.

1. When you are going to remove the framework that already used:
  - AD fresca serve will automatically migrate your users' identifier to IFV since we already know both IFA and IFV values. There won't be any issue. (The migration is only available with SDK version higher than 1.3.3)
2. When you are going to add the framework:
  - Since our SDK does not have users' previous IFA values, we can't do the migration process. To solve this issue, we provide **setUseIFVOnly** method. If you set 'YES' value to the method, our SDK will try to match users with IFV value even though the framework is added. If you don't use this method while adding the frameowkr, your exstitng users will be recognized as new users. Please be careful to read this section.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [AdFrescaView startSession:API_KEY];
  [[AdFrescaView shardAdView] setUseIFVOnly:YES];
}
```

Please contact us if you have any concern or issue of this section.

* * *

## Troubleshooting

Duplicated Symbol Error of SBJson may occur if you already have SBJson in your project.

<img src="https://adfresca.zendesk.com/attachments/token/ikafbcqjnj9jbak/?name=6666.png">

In this case, compiling will fail with the errors above.

You need to remove 'SBJson' folder in our SDK folder to solve this issue. The latest AD fresca SDK uses [3.1 release](https://github.com/stig/json-framework/tree/v3.1) version of SBJson. You may have a problem when you use older versions in your project.

In other case, if you cannot see any message or get other errors, you can debug by implementing didFailToReceiveAdWithException event method of  AdFrescaViewDelegate 

```objective-c
- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes

- 1.4.1 (2014/06/18 Updated)
  - Support 64-bit configuration of Xcode
  - Include IAP Beta features to 1.4.1
  - Rename some methods (loadAd -> load, showAd -> show, closeAd -> close)
- 1.4.0-beta1
  - 'In-App-Purchase Tracking' feature is now added to iOS SDK. Please refer to In-App-Purchase Tracking (Beta) section.
- 1.3.5
  - SDK supports 'Reward Item' feature of the announcement campaign.
  - SDK supports 'Incentivized CPA Campaign'. Please refer to 'CPI Identifier' section for detail. 
  - AFRewardItemDelegate is added for easier implementation of reward item. Please refer to 'Reward Item' section for detail 
- 1.3.4 
  - Fix testDeviceId property to support all iOS versions
- 1.3.3 
  - When APNS device token is registered or changed, SDK now updates  a token value to our service in real-time. (previous SDKs only updated the value when app session started). 
- 1.3.2
  - Update custom parameters to set long long type.
  - customParameterWithIndex method will return nil value if no data was set
- 1.3.1 
  - Added 'Close mode' feature. You can control the closing action of an interstitial view on our dashboard.
  - SDK starts to cache n-App-Purchase Count, Custom Parameter information.
  - Fix a minor bug that landscape view was not normally shown on iOS 4.3
  - Update custom parameters to set long type.
- 1.3.0
  - Added Incentivized Campaign feature. Check '10. Reward Item' section.
- v.1.2.1
  - numberOfInAppPurchases property is added. You can set how many times user purchased in-app items, and then use it for campaign targeting options. (See 'In-App Purchased User Management' for detail)
isInAppPurchasedUser property is deprecated. Please use numberOfInAppPurchases instead.
- v.1.2.0
  - Event feature is available. (See 'Event Setting' for detail)
  - AD Slot feature is deprecated and replaced into Event. Default slot will be set to '1' event index and AD Only Slot will be set to '2' event index.
  - When loadAd() is called again while previous request is not completed, the old request will be canceled and the latest one will show the contents. (In previous versions, you could not call loadAd() while requesting)
  - If contents have app store link, SDK will show app store page inside of app. So user wil no longer leave to your app. (Please add StoreKit.framework for this feature)
  - When user clicked image contents, view will be automatically closed as a default.
  - Auto Close Timer feature is available. You can set on Dashboard.
- v1.1.0 
  - SDK supports 'Push Notification' feature (See '8. Push Notification Setting' for detail)
- v1.0.1 
  - SDK supports 'Custom Parameter' feature (See 'Custom Parameter Management' for detail)
- v.1.0.0
  - HTML5 View is added (There is no need to change any SDK code in your app!)
  - SDK supports Advertising Identifier', which is recently added in iOS 6. For this reason, AdSupport framework is required to be added in your Xcode project (See 'SDK Installation')
- v0.9.9 
  - SDK supports an official iOS 6 and iPhone 5 with ARMV7s
- v0.9.8
  - testDeviceId, printTestDeviceId properties are added to support a test mode (Please, see 'Checking Test Device ID)
- v0.9.7
  - AD Slot feature added as an announcement feature added (See 'AD Slot Setting')
  - adViewClosed bug fixed. adViewClosed event must be called after showAd() requested.
  - The AD Caching feature is optimized for better performance.
  - Typos in some methods are fixed.
- v0.9.6 
  - The AD Caching feature is optimized for better performance.
- v0.9.5 
  - Now, SDK use the AD Caching feature for faster ad display. If the cached AD exists, the cached AD will be shown up automatically.
  - timeoutInterval property is added. You can set a timeout interval for AD request. If AD is not loaded within the time interval, AD won't be displayed to users.
  - testModeEnabled property is deprecated . All the test mode control will be proceed on our admin website from now on.
  - required method list is modifed for AdFrescaViewDelegate
  - SDK supports iOS 6
- 0.9.4
  - startSession method is added. For accurate session logging, add startSession method in didFinishLaunchingWithOptions()
  - isInAppPurchasedUser property is added. You can manage your in-app purchased users with our service.
- v0.9.3
  - the session logging feature is added. SDK will automatically log user session data.
- v0.9.2
  - Performance optimized
  - Bug fix for some click events.
- v0.9.1
  - SDK UI improvement.
- v0.9.0
  - AD fresca iOS SDK is now released! basic AD feature is included.

