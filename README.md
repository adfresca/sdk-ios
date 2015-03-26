## Contents
- [Basic Integration](#basic-integration)
  - [Installation](#installation)
  - [Start Session](#start-session)
  - [In-App Messaging](#in-app-messaging)
  - [Push Messaging](#push-messaging)
  - [Test Device Registration](#test-device-registration)
  - [Test Mode](#test-mode)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Stickiness Custom Parameter](#stickiness-custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [AdFrescaViewDelegate](#adfrescaviewdelegate) 
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [IFV Only Option](#ifv-only-option)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

 Download our SDK with the following link:

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) 

To add our SDK into your Xcode project, please follow the instructions below:

1) Drag & Drop AdFresca folder into the framework folder on your Xcode project.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2) Add System Configuration.framework, StoreKit.framework, and AdSupport.framework (optional) into your target if these frameworks are not added yet.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - If you add AdSupport.framework, our SDK collects [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distinguish the user's device. We use this value to provide the cross-promotion campaign with install and action tracking.
  - If you do not add AdSupport.framework, our SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distinguish user's device. In this case, you can't use any cross promotion feature. Also, as IFV's policy, your user may be recognized as a new user after re-installing app.

  If you'd like to add AdSupport.framework or remove the framework from existing xcode project with our SDK, please refer to the [IFV Only Option](#ifv-only-option) section to migrate your users.

3) Add -ObjC to Other Linker Flag on your target's build setting.

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4) In Info.plst, set 'aps-environment' value as 'production'. It is necessary to use a push notification feature.

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  Also, if your orientation of the app is landscape only, set 'Initial interface orientation' to 'Landscape (right home button)'

  Finally, set your own URL Scheme value. the example below shows how to set URL Scheme with "myapp" value. It will be used in the cross promotion feature.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

Nudge SDK has been successfully installed without any build error. If you have a 'Duplicate Symbol' error, please refer to the [Troubleshooting](#troubleshooting) section.

### Start Session

Now, let’s start to put some simple SDK codes in your app. You first need to call startSession() method with your API Key. To get your API Key, go to our [Dashboard](https://admin.adfresca.com) and then click 'Settings - API Keys' button in your app's 'Overview' page.

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

With the in-app messaging feature, you can deliver a message to your in-app users in real time. Simply put 'load' and 'show' methods where you want to deliver a message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when your user matches the in-app messaging campaign's target logics. We will discuss more in detail about the in-app messaging's dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca load]; 
  [fresca show]; 
} 
```

When you call in-app messaging methods, you will see the test message below. If you tap on the image, it will redirect to the product page of the app on the app store. You will hide this test message by changing the test campaign mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can send push messages using Nudge. Follow the steps below to configure the push notification settings in your app.

1. Upload your APNS Certificate file (.p12) to our Dashboard
  - You can export your .cer file to .p12 file using Keychain. Please refer to [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/82614238) to generate .p12 and upload to [Dashboard](https://admin.adfresca.com)

2. Check your provisioning
  - Nudge only supports APNS production environment. So, you should build your app with App Store or Ad Hoc Provisioning file to enable production mode.

3. Add the following codes to AppDelegate 
  ```objective-c
  #import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ....
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
    /// Check a push notification is form Nudge. Also, ignore a notification received when app is already running 
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }  
  } 
  ```

### Test Device Registration

Nudge supports a test campaign feature. With the test campaign feature, you can deliver test messages to only registered test devices. 

To register your test device to our dashboard, you need to get your test device ID from our SDK. Our SDK provides two ways to show test device IDs.
 
1. Using testDeviceId Property
  - After connecting your device with Xcode, you can simply print out test device ID with a logger.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  NSLog(@"Nudge Test Device ID = %@", fresca.testDeviceId); 
  [fresca load];
  [fresca show];
```

2. Displaying test device ID on your app screen using printTestDeviceId property
  - When you are not able to connect tester's device in your office, you have to set printTestDeviceId to true, and then let them install this app build. They can see their own test device ID on the app screen. 
  - It is useful when testers are working remotely. 
  - printTestDeviceId property must be set to false when you distribute your app on the store. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca load];
  [fresca show];
  ```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). You can register your device in the 'Test Device' menu.

### Test Mode

Nudge SDK supports a test mode feature. With the test mode feature, you can verify your SDK codes. When you add **setTestMode(YES)** code, SDK will print a log message with a result for each your SDK code. 

  ```objective-c
  [AdFrescaView setTestMode:YES];
  ```

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/ios_sdk_test_mode.png" width="900" />

The test mode currently provides 'Start Session', 'Push Messaging', 'In-App Purchase Tracking', 'Custom Parameter', and 'Stickiness Custom Parameter' logs. Other feature support will be available soon.


* * *

## IAP, Reward and Sales Promotion

### In-App Purchase Tracking

With In-App-Purchase Tracking , you can analyze all the purchases of your users, and use it for targeting specific user segments to display your campaigns.

There are two types of purchases you can track with our SDK.

1. **Hard Currency Item Purchase Tracking:**  Purchases made with real money. For example, a user purchased ‘$1.99' to get 'Gold 100' cash item
2. **Soft Currency Item Purchase Tracking:** Purchases made with virtual money. For example, a user purchased 'Gold 10' to get 'Rocket Launcher' item

You don't need to manually write down an item list. All the items are tracked by our SDK and automatically added to our dashboard. To see the list of items, go to 'Overview > Settings > In-App Items' page in our dashboard.

Let's get started and implement SDK codes with the examples below. 

#### Hard Currency Item Tracking

In iOS, the purchase of 'Hard Currency Item' is made with Apple's Storekit framework. When a user purchased the item successfully, simply create AFPurchase object and use logPurchase method. Also, call cancelPromotionPurchase method when a user cancelled or failed to purchase.

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

  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeHardItem
                                                    itemId:itemId
                                              currencyCode:currencyCode
                                                     price:[price doubleValue]
                                              purchaseDate:transactionDate
                                    transactionReceiptData:transactionReceiptData];

  [[AdFrescaView shardAdView] logPurchase:purchase];
}

- (void)failedTransaction:(SKPaymentTransaction *)transaction 
{
  [[AdFrescaView shardAdView] cancelPromotionPurchase];
  ....
}
```

For more details of AFPurchase object with the hard currency item, check the table below.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service can distinguish each item by this value.
currencyCode(string) | Set the current code of IOS 4217 standard. You may use SKProduct's's value or manually set the value from your server.
price(double) | Set the item price. You may use SKProduct's value or manually set the value from your server.
purchaseDate(date) | Set the date of purchase. You may use SKPaymentTransaction.transactionDate value. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of the user's device.
transactionReceiptData(nsdata) | Set the receipt property of SKPaymentTransaction object. We will use it to verify the receipt in the future.

#### Soft Currency Item Tracking

When a user purchased a soft currency item in the app, you can also create AFPurchase object and call logPurchase() method. Also, call cancelPromotionPurchase method when a user cancelled or failed to purchase.

```objective-c
- (void)didPurchaseSoftItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeSoftItem
                                                    itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil]; 

  [[AdFrescaView shardAdView] logPurchase:purchase];
}

- (void)didFailToPurchaseSoftItem {
  [[AdFrescaView shardAdView] cancelPromotionPurchase];
}
```

For more details of AFPurchase object with soft currency items, check the table below. You don't need to set transactionReceiptData property in soft currency item tracking.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service can distinguish each item by this value.
currencyCode(string) | Set the item's soft currency code. (ex: 'gold', 'gas')
price(double) | Set the item price. You may get this value from your server. (ex: 100 of gold)
purchaseDate(date) | Set the date of purchase. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of the user's device.
transactionReceiptData(nsdata) | Set nil for AFPurchaseTypeSoftItem

#### IAP Troubleshooting

After you call logPurchase() method, the purchase data is updated to our dashboard in real-time. You can see the list of updated item in 'Overview > Settings > In-App Items' menu.

If you don't see any data in our dashboard, your AFPurchase object may be invalid. To check it, you can implement  AFPurchaseDelegate and call log logPurchase(purchase, delegate) method. 

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPurchaseDelegate> {
  ...
}

// AppDelegate.m
- (void)didPurchaseSoftItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeSoftItem
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

When you set 'Reward Item' section of the reward campaign or 'Incentive item' section of the incentivized CPI & CPA campaigns, you should implement this 'reward item' code to give a reward item to users.

To implement codes, we use the two codes noted below:

- checkRewardItems method: This method is to check if any item is available to receive. We recommend to put this code when the app becomes active. 
- AFRewardItemDelegate implementation: When there is a reward available for the user, itemRewarded event is automatically called with AFRewardItem object from our SDK. You can give an item to the user with AFRewardItem object.

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
  
  // Ggive an item to users.  
  [self sendItemToUser:currentUserId itemId:item.uniqueValue quantity:item.quantity securityToken:item.securityToken];
}
```

itemRewarded event is called when there is a reward available for a user.

- Reward Campaign: The event is called when your user sees the campaign contents.
- Incentivized CPI Campaign: The event is called when SDK confirms Advertising App's install.
- Incentivized CPA Campaign: The event is called after SDK confirms Advertising App's install and the user called the targeted marketing event in Advertising App.
 
If a user has any network connectivity issues, our SDK stores the reward data in the app's local storage, and then re-checks during the next app session. So, we guarantee the user will always get a reward with our SDK.

#### implementing sendItemToUser()

You need to give a reward item to the user using your own client code or back-end server API. Your client may send an API request with an unique value of the item, quantity and security token value to your server. Then the server application will add the reward item to the user's item inventory.

#### Hack Proof Code

Our SDK never calls itemRewarded event more than once per campaign by checking it with device identifiers to avoid abuse. However it is still possible for hackers to hijack your API request between your client and your back-end server. 

If you want more security, you can use a security token, which is a unique value generated per a reward campaign. We generate a new token for you when you create a reward campaign but you can use your own. Here are steps to implement hack proof codes:

1. You will store a security token to your own database before starting a reward campaign. You should reject any reward requests with an invalid token.
2. If some users request rewards with the same token more than once, you should reject those requests.
3. If you think your security token is exposed to hackers, you can always change it in our dashboard.

### Sales Promotion

You can promote in-game items to specific user segements. When a user taps on an action button of an in-app message, a purchase UI will appear. Our SDK will automatically detect if the user made a purchase or not, and then will update the campaign performance in our dashboard in real time. Please note that [In-App Purchase Tracking](#in-app-purchase-tracking) feature is a prerequisite to this promotion feature.

Start implementing AFPromotionDelegate. onPromotion event is automatically called when a user taps on an action button of an image message in a sales promotion campaign. You just need to show the purchase UI of the promotion item using 'promotionPurchase' object. 

For Hard Currency Items, you should use StoreKit library codes to show purchase UI. You can get the product identifier value of SKProduct from ItemId property of promotionPurchase object.

For Soft Currency Items, you should use your own purchase UI which might be already implemented in your store page. Also there are discount options for soft currency item sales promotion campaigns. You can check the discount type using discountType property of promotionPurchase object

1. **Discount Price**: Users can buy a promotion item at a specific discounted price. You can get the price from price property.

2. **Discount Rate**: Users can buy a promotion item at a discount rate. You will calculate the discounted price applying the discount rate which can be earned from discountRate property.

```objective-c
// AppDelegate.h
@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPromotionDelegate> {

}
....

// AppDelegate.m
- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setPromotionDelegate:self];
}

- (void)onPromotion:(AFPurchase *)promotionPurchase {
  NSString *itemId = promotionPurchase.itemId;
  NSString *logMessage = @"onPromotion: no logMessage";
  
  if (promotionPurchase.type == AFPurchaseTypeHardItem) {
    // Use SKPaymentQueue to show the purchase ui of this item.
    SKProduct *product = [self paymentWithProductIdentifier:itemId];
    SKPayment *payment = [SKPayment paymentWithProduct:product];
    [[SKPaymentQueue defaultQueue] addPayment:payment];
    
    logMessage = [NSString stringWithFormat:@"on HARD_ITEM Promotion (%@)", itemId];
    
  } else if (promotionPurchase.type == AFPurchaseTypeSoftItem) {
    NSString *currencyCode = promotionPurchase.currencyCode;
    
    if (promotionPurchase.discountType == AFDiscountTypePrice) {
      // Use a discounted price
      double discountedPrice = promotionPurchase.price;
      [self showPurchaseUIWithItemId:itemId withDiscountedPrice:discountedPrice];
      logMessage = [NSString stringWithFormat:@"on SOFT_ITEM Promotion (%@) with %.2f %@", itemId, discountedPrice, currencyCode];

    } else if (promotionPurchase.discountType == AFDiscountTypeRate) {
      // Use this rate to calculate a discounted price of item. discountedPrice = originalPrice - (originalPrice * discountRate)
      double discountRate = promotionPurchase.discountRate;
      [self showPurchaseUIWithItemId:itemId withDiscountRate:discountRate];
      logMessage = [NSString stringWithFormat:@"on SOFT_ITEM Promotion (%@) with %.2f %%", itemId, discountRate * 100.0];
    }
    
    NSLog(@"%@", logMessage);
  }
}

```

Please make sure that you implement 'cancelPromotionPurchase' method when a user cancelled or failed to purchase items.


* * *

## Dynamic Targeting

### Custom Parameter

Our SDK can collect user specific data such as level, stage, maximum score, etc. We use them to deliver a personalized and targeted message in real time to specific user segments that you can define.

To implement codes, simply call setCustomParameterWithValue method with passing parameter's unique key and its value.

You will call the method after your app is launched and when the values have changed. (If you can't set the values without user sign in, you may set them right after a user signs in.)

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{
  ...
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forKey:@"level"];                    
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.stage] forKey:@"stage"];
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forKey:"facebook_flag"];   
}

- (void)levelDidChange:(int)level 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forKey:"level"];
}   

- (void)stageDidChange:(int)stage 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:stage] forKey:"stage"];
}
....
```

After you write the codes and set values, you will be able to see a list of custom parameters you added on [Dashboard](https://admin.adfresca.com). 1) Select an App 2) In 'Overview' menu, click 'Settings - Custom Parameters' button.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/custom_parameter_index.png">

You need to set 'Name' value of each custom parameter to activate. You can activate custom parameters up to 20. Nudge only collects data of activated custom parameters and use them for targeting.

* * *

### Stickiness Custom Parameter

(Stickiness Custom Parameter is currently in beta. To use this feature, contact our [support team](mailto:support@nudge.do))

If your app has a value to measure a user's stickiness (e.g. ‘play count’ in a stage-based game), you can use it to create a  'Stickiness Custom Parameter' with Nudge. You can define user segments such as 'users who played 30 stages in a week' and 'Users who played 5 stages today'.

To begin, you first need to set a new custom parameter (e.g. play count), and then set it as stickiness custom parameter (stickiness mode can only be set by Nudge team currently).

To implement codes, simply pass the value to **incrCustomParameterWithAmount** method whenever the stickiness value is increased. Our SDK will automatically calculate the accumulated value and daily increased value and update user profiles.

After you write the code, you will see new filters available in Segment UI of our dashboard. (e.g. 'Today's play count', 'Average play count in a week', and 'Total play count in a week').

```objective-c
- (void)didFinishGame
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca incrCustomParameterWithAmount:[NSNumber numberWithInt:1] forKey:"play_count"];
}
```

If your app is already live in app stores and have live data, you need to set the most up-to-date value before you call incrCustomParameterWithAmount method. You can check if the custom parameter value is already set by using **hasCustomParameterWithKey(key)** method. If the value is not set yet, set the accumulated value from your app server.

```objective-c
- (void)didUserSignIn 
{
  ....

  AdFrescaView *fresca = [AdFrescaView sharedAdView];       
  if (![fresca hasCustomParameterWithKey:"play_count"]) {
    [fresca setCustomParameterWithValue:[NSNumber numberWithInt:user.totalPlaycount] forKey:"play_count"];
  }
}
```

* * *

### Marketing Moment

A Marketing Moment is where and when you want to engage your users. For example, you may need to deliver the message when the user completes a quest or enters into the in-app store. You can deliver the personalized and targeted message at a specific moment in real time.

To implement codes, simply call load method with passing marketing moment's index. You can get the marketing moment's index in our [Dashboard](https://admin.adfresca.com): 1) Select an App 2) In 'Overview' menu, click 'Settings - Marketing Moment' button. 

You will call the method after the moment has happened in the app.

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

With implementing AdFrescaViewDelegate in your code, you can check all the events in the SDK. 

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

//This event occurs when SDK will start to receive contents
- (void)adViewWillReceiveAd:(AdFrescaView *)theAdView {}

//This event occurs when SDK receive contents successfully
- (void)adViewDidReceiveAd:(AdFrescaView *)theAdView {}

// This event occurs when content was not received properly with error information.
- (void)adView:(AdFrescaView *)view didFailToReceiveAdWithException:(AdException *)error {}

// This event occurs when the user closes view. it's also called when content is not loaded with an error after calling show(). So this is endpoint of our process
- (void)adViewClosed:(AdFrescaView *)adView {}
```

There are many good practices when implementing AdFrescaViewDelegate.

1. Scenarios 1: Display overlay contents in the bootup screen
  - Display Boot-up screen (Logo, etc)
  - Contents will be displayed over the boot-up screen
  - If the user closes the view, the main page will be loaded.
2. Scenarios 2: Insert Contents between each stage
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

If user clicked contents that opens App Store or other applications, user will leave your app screen.

In this case, if you implemented adViewClosed() event like above example, user may see an unnatural paging animation since the app was temporarily paused by another application.

To fix this issue, follow the steps below:
  1. In dashboard, you should change 'Close mode' to 'Override' in your marketing moment settings. (it will prevent to close view when user clicked)
  2. In app codes, Implement  applicationWillEnterForeground() event of AppDelegate like below:

```objective-c
#pragma mark – AdFrescaViewDelegate

- (void)applicationWillEnterForeground:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView shardAdView];
  if (!fresca.hidden && fresca.userClicked) {
    [fresca close];
  }
}
```

### Timeout Interval

You can set a timeout interval for a marketing moment request. If the message is not loaded within this time interval, the message won't be displayed to users and our SDK will return control to your app.

Default is 5 seconds and you can set from 1 seconds to 5 seconds.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];  
fresca.timeoutInterval = 3 // # secs  
[fresca load];
[fresca show];
```

* * *

## Reference

### Deep Link

You can set your own URL Schema as 'Deep Link' of the campaigns. So, you can navigate your users to a specific page or do some custom actions when a user clicks the image message. 

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

In this example, ItemViewController will be displayed when you set the campaign Deep Link as 'myapp://item'

* * *

### Cross Promotion Configuration

When using Incentivized CPI & CPA Campaigns, your users in 'Media App' can get an incentive item when they install 'Adverting App' from the campaigns.

- Medial App: The media app which displays the promotion image and gives an incentive item to users.
- AdvertisingApp: The promotion app which is displayed with an image in the media app's screen.

For more details on incentivized campaigns and configuration guide in our dashboard, please refer to ['Understanding Cross-promotion (Korean)'](https://adfresca.zendesk.com/entries/22033960) guide.

To integrate our SDK with this feature, you should have URL Schema value for the adverting app and implement codes to give an incentive item to users in the media app.

#### Configuration for Advertising App.:

  Check your url schemes to check app install in Info.plst as follows

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  In this case, you should set CPI Identifier value of advertising app to "myapp://" in our dashboard (Overview > App Store Edit).

  For iOS, url schema value may be duplicated with other apps, so be careful to choose an unique value.

  For Incentivized CPI Campaigns, our SDK Installation of the advertising app is not required. You can only set URL Schema to use app's install.

  However, if you use Incentivized CPA Campaigns, our SDK installation is required and you should also implement 'Marketing Moment' feature to check a reward condition. For example, when you set the reward condition to check 'Tutorial Complete' moment, you should call the marketing moment method to inform your user achieved the goal.
    
  ```objective-c
  - (void)didTutorialComplete {
    AdFrescaView *fresca = [AdFrescaView sharedAdView];   
    [fresca load:MOMENT_INDEX_TUTORIAL_COMPLETE];  
    [fresca show];
  }  
  ```

#### SDK implementation for Media App:

  To give an incentive item to the media app's users, please refer to the [Give Reward](#give-reward) section.

* * *

### IFV Only Option

As [SDK Installation](#installation) describes, SDK uses [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distinguish the user's device if you add AdSupport.framework. In other hands, SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distinguish user's device if you do not add AdSupport.framework.

If you are adding the framework or remove it while you're updating your app which already exists in the app store, the following issues may occur.

1. When you are going to remove the framework that already used:
  - Nudge server will automatically migrate your users' identifier to IFV since we already know both IFA and IFV values. There won't be any issue. (The migration is only available with SDK version higher than 1.3.3)
2. When you are going to add the framework:
  - Since our SDK does not have users' previous IFA values, we can't do the migration process. To solve this issue, we provide **setUseIFVOnly** method. If you set 'YES' value to the method, our SDK will try to match users with IFV value even though the framework is added. If you don't use this method while adding the frameowkr, your exstitng users will be recognized as new users. Please be careful to read this section.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [AdFrescaView startSession:API_KEY];
  [[AdFrescaView shardAdView] setUseIFVOnly:YES];
}
```

Please contact us if you have any concerns or issues on this section.

* * *

## Troubleshooting

Duplicated Symbol Error of SBJson may occur if you already have SBJson in your project.

<img src="https://adfresca.zendesk.com/attachments/token/ikafbcqjnj9jbak/?name=6666.png">

In this case, compiling will fail with the errors above.

You need to remove 'SBJson' folder in our SDK folder to solve this issue. The latest Nudge SDK uses [3.1 release](https://github.com/stig/json-framework/tree/v3.1) version of SBJson. You may have a problem when you use older versions in your project.

In other case, if you cannot see any message or get other errors, you can debug by implementing didFailToReceiveAdWithException event method of  AdFrescaViewDelegate 

```objective-c
- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- **v1.5.4 _(2015/03/27 Updated)_**
  - [Test Mode](#test-mode) is added.
- v1.5.3
  - [Custom Parameter](#custom-parameter) provides 'string' unique key. (Integer key is still available)
- v1.5.2
  - Fix minor bugs with 'Stickiness Custom Parameter'
- 1.5.1
  - Add hasCustomParameterWithIndex method. 
- 1.5.0
  - Add [Stickiness Custom Parameter](#stickiness-custom-parameter) feature.
- v1.4.9
  - AFPurchaseTypeHardItem and AFPurchaseTypeSoftItem enums are added to AFPurchase class to replace AFPurchaseTypeActualItem and AFPurchaseTypeVirtualItem which will be deprecated. Please refer to [In-App Purchase Tracking](#in-app-purchase-tracking) section.
- v1.4.8
  - Support In-App Purchase Tracking feature for Unity Plugin
- v1.4.7
  - Fix the landscape image view bugs for iPhone 6 models
- v1.4.6
    - Support A/B test feature. (No coding is required)
- v1.4.5
    - Fully tested with official ios 8. Also, there are no compatibility issues with older Nudge SDKs.
    - Please check [Push Messaging](#push-messaging) section to update your push registration codes with iOS 8.
    - Fix mior bugs of deep links.
- v1.4.4	
    - Fix mior bugs of promotion.
- v1.4.3
    - Support sales promotion campaign. Please refer to [Sales Promotion](#sales-promotion) section.
    - Support security token of reward campaign's hack proof. Please refer to [Give Reward](#give-reward) section.
    - Add cancelPromotionPurchase() method to [In-App Purchase Tracking](#in-app-purchase-tracking)
    - Support tap area feature.
- v1.4.2
    - SDK will match multiple campaigns and show multiple messages in one marketing moment request.
- v1.4.1
  - Support 64-bit architecture configuration of Xcode.
  - Include IAP Beta features to 1.4.1.
  - Rename some methods (loadAd -> load, showAd -> show, closeAd -> close). Old method will work fine as we guarantee the backward compatibility. 
- v1.4.01
  - 'In-App-Purchase Tracking' feature is now added to iOS SDK. Please refer to In-App-Purchase Tracking section.
- v1.3.5
  - SDK supports 'Reward Item' feature of the In-App Messaging campaign.
  - SDK supports 'Incentivized CPA Campaign'. Please refer to 'CPI Identifier' section for detail. 
  - AFRewardItemDelegate is added for easier implementation of reward item. Please refer to 'Reward Item' section for detail 
- v1.3.4 
  - Fix testDeviceId property to support all iOS versions
- v1.3.3 
  - When APNS device token is registered or changed, SDK now updates  a token value to our service in real-time. (previous SDKs only updated the value when app session started). 
- v1.3.2
  - Update custom parameters to set long long type.
  - customParameterWithIndex method will return nil value if no data was set
- v1.3.1 
  - Added 'Close mode' feature. You can control the closing action of an interstitial view on our dashboard.
  - SDK starts to cache n-App-Purchase Count, Custom Parameter information.
  - Fix a minor bug that landscape view was not normally shown on iOS 4.3
  - Update custom parameters to set long type.
- v1.3.0
  - Added Incentivized Campaign feature. Check '10. Reward Item' section.
- v.1.2.1
  - numberOfInAppPurchases property is added. You can set how many times user purchased in-app items, and then use it for campaign targeting options. (See 'In-App Purchased User Management' for detail)
isInAppPurchasedUser property is deprecated. Please use numberOfInAppPurchases instead.
- v.1.2.0
  - Event feature is available. (See 'Event Setting' for detail)
  - AD Slot feature is deprecated and replaced into Event. Default slot will be set to '1' event index and AD Only Slot will be set to '2' event index.
  - When load() is called again while previous request is not completed, the old request will be canceled and the latest one will show the contents. (In previous versions, you could not call load() while requesting)
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
  - AD Slot feature added as an In-App Messaging feature added (See 'AD Slot Setting')
  - adViewClosed bug fixed. adViewClosed event must be called after show() requested.
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

