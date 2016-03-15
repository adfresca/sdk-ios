## Contents
- [Basic Integration](#basic-integration)
  - [Installation](#installation)
  - [Start Session](#start-session)
  - [Sign In](#sign-in)
  - [In-App Messaging](#in-app-messaging)
  - [Push Messaging](#push-messaging)
  - [Test Device Registration](#test-device-registration)
  - [Test Mode](#test-mode)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
  - [Limited Time Offer](#limited-time-offer)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Profile Attributes](#custom-profile-attributes)
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

[iOS SDK Download](http://file.nudge.do/distribution/sdk/nudge-sdk-for-iOS.zip) 

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

4) In Info.plist, set 'aps-environment' value as 'production'. It is necessary to use a push notification feature.

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  If your orientation of the app is landscape only, set 'Initial interface orientation' to 'Landscape (right home button)'

  Also, set your own URL Scheme value. the example below shows how to set URL Scheme with "myapp" value. It will be used in the cross promotion feature.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>
  
  If you use [Limited Time Offers](#limited-time-offer), add a row and name the new key 'Fonts provided by application'. After this, you can fill the key (ex. 'Item 0') with the value 'nudge-icon.ttf'.

<img src="http://file.nudge.do/guide/sdk/nudge-icon-font.png">


5) In iOS 9 and Xcode 7+, [App Transport Security](https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/) is enabled by default. Thus, you need to configure domain exceptions to enable our SDK send data to Nudge servers. Check [Info.plist example](https://gist.github.com/sunku/2dba02239f168dfec5d9#file-nsapptransportsecurity-plist) and update your file in Xcode accordingly.

If you have a 'Duplicate Symbol' error, please refer to the [Troubleshooting](#troubleshooting) section.

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

### Sign In

Sign In feature allows you to track a user’s sign in or sign out actions. Nudge identifies a user with user_id, a string passed by signIn method so Nudge can recognize a user with multiple devices as a single user, not as multiple users, which is a more accurate way and also prevents a user from claiming multiple rewards by using different devices. You can also launch a campaign targeting users based on their signed-in status. (ie. signed-in users or guest users)

A user should be signed in at all times, either as a member or as a guest. If another user signs in, the signed-in user will be signed out automatically. You need to pass a user identifier (string) to **signIn(string)** method when a user signs in to your server (including auto sign-in). You also need to put **signInAsGuest(string)** for a guest user who has not signed up or signed in with existing accounts yet.


```objective-c
- (void)onAppStart {
  if(isSignedIn) {
    // It should be called on both auto sign-in and manual sign-in
    [[AdFrescaView shared] signIn:@"user_id"];
  } else {
	 // If you use a separate guest_id to track a guest user, you can pass it as an argument 
  	 // If you don’t use a separate identifier to track a guest user, you don’t need to set guest_id
    [[AdFrescaView shared] signInAsGuest:@"guest_id"];
  }
}
```

**signedUserId()** method will return a user_id for a signed-in user, a guest_id for a signed-in guest user, or a device identifier for a guest user without guest_id. You can use this method to test your codes.

### In-App Messaging

With in-app messaging, you can deliver a message to targeted users. Simply put 'Load' and 'Show' methods where and when you want to display a message. The type of message can be an interstitial image, text, or iFrame webpage. You can also reward an item to a user with in-app messaging. (Please refer to the [Give Reward](#give-reward) section.) The message is only displayed when a user's profile matches the in-app messaging campaign's target logics. We will discuss more details of the dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView shared]; 
  [fresca load]; 
  [fresca show]; 
} 
```

When you call in-app messaging methods, you will see the test message below. If you tap on the image, it will redirect to the product page of the app on the app store. You can deactivate this test message by changing the test campaign mode configuration later.

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
    // if your app has push on/off configuration, you should set nil value when 'off' is set by your user.
    [AdFrescaView registerDeviceToken:deviceToken];
  }

  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    if ([AdFrescaView isFrescaNotification:userInfo]) {
      [AdFrescaView handlePushNotification:userInfo];
    }  
  } 
  ```

4. if your app has push on/off configuration, you should update push device token to our SDK when your user changes on/off configuration.

```objective-c
-(void)didPushConfigChange:(BOOL)pushEnabled {
  if (pushEnabled) {
    [AdFrescaView registerDeviceTokenString:@"YOUR_APNS_DEVICE_TOKEN"];
  } else {
    [AdFrescaView registerDeviceTokenString:nil];
  }
} 
```


### Test Device Registration

Nudge supports a test campaign feature. With the test campaign feature, you can deliver test messages to only registered test devices. 

To register your test device to our dashboard, you need to get your test device ID from our SDK. Our SDK provides two ways to show test device IDs.
 
1. Using testDeviceId Property
  - After connecting your device with Xcode, you can simply print out test device ID with a logger.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView shared];
  NSLog(@"Nudge Test Device ID = %@", fresca.testDeviceId); 
  [fresca load];
  [fresca show];
```

2. Displaying test device ID on your app screen using printTestDeviceId property
  - When you are not able to connect tester's device in your office, you have to set printTestDeviceId to true, and then let them install this app build. They can see their own test device ID on the app screen. 
  - It is useful when testers are working remotely. 
  - printTestDeviceId property must be set to false when you distribute your app on the store. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView shared];
  fresca.printTestDeviceId = YES;
  [fresca load];
  [fresca show];
  ```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). You can register your device in the 'Test Device' menu.

### Test Mode

Nudge SDK supports a test mode feature. With the test mode feature, you can verify your SDK codes. When you add **setTestMode(YES)** code, SDK will print a log message with a result for each SDK code. 

  ```objective-c
  [AdFrescaView setTestMode:YES];
  ```

<img src="http://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/ios_sdk_test_mode.png" width="900" />

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

  [[AdFrescaView sharedAdView] logPurchase:purchase];
}

- (void)failedTransaction:(SKPaymentTransaction *)transaction 
{
  [[AdFrescaView sharedAdView] cancelPromotionPurchase];
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

  [[AdFrescaView sharedAdView] logPurchase:purchase];
}

- (void)didFailToPurchaseSoftItem {
  [[AdFrescaView sharedAdView] cancelPromotionPurchase];
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
  [[AdFrescaView sharedAdView] logPurchase:purchase, self];
}

- (void)purchase:(AFPurchase *)purchase didFailToLogWithException:(AdFrescaException *)exception {
  NSLog(@"AFPurchase didFailToLogWithException :: purchase = %@, exception = %@", [purchase JSONRepresentation], [exception description]);
}
```

* * *

### Give Reward

You can reward a free item to a user with a reward campaign or an incentivised CPI/CPA campaign.

There are two steps of reward claim process:

1. Reward Claim Request: Nudge SDK triggers a reward claim event when the current user is matched for reward campaign. On this event, you need to claim a reward for the user.
2. Finish Reward Claim: When you finish to claim a reward,  you should inform it to Nudge SDK. 

If there is a reward item for a user, onRewardClaim event is triggered and the item information is passed along, which you will use to give the item to a user.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFRewardClaimDelegate> {
  ...
}


// AppDelegate.m

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [[AdFrescaView sharedAdView] setRewardClaimDelegate:self];
}

- (void)onRewardClaim:(AFRewardItem *)item {
  NSString *logMessage = [NSString stringWithFormat:@"You got the reward item! (%@)", item.name];
  NSLog(@"%@", logMessage);
  
  // Give an item to a user  
  [self sendItemToUser:currentUserId itemId:item.uniqueValue quantity:item.quantity securityToken:item.securityToken rewardClaimToken:item.rewardClaimToken];
}
```

You need to inform Nudge SDK that you have given a reward to a user successfully by calling finishRewardClaim() method. Unless Nudge SDK receives the confirmation of the reward claim, Nudge SDK will assume the claim has failed due to some error on the client-side or the server-side then re-trigger onRewardClaim event. It won't happen until the next marketing moment is called and 3 minutes have passed after the previous event was triggered, which prevents giving a reward multiple times by triggering onRewardClaim event again while the previous event is being handled.


```objective-c
- (void)onRewardClaimSuccess:(AFRewardItem *)item {
  [[AdFrescaView shared] finishRewardClaim:item.rewardClaimToken];
}
```

#### Implementing SendItemToUser()

You need to give a reward item to the user using your own client code or back-end server API. Your client may send an API request with a unique value of a reward item, a quantity and a security token to your server. Then the server application will add a reward item to the user's item inventory or inbox.

#### Hack Proof Code

You can prevent client-side hacking using a security token and a reward claim limit count for a campaign. A security token is automatically created when a campaign is created using our dashboard and you can set it manually if needed. You can set a reward claim limit count for a campaign via our dashboard.

1. You can store a security token on your own database and compare it with a security token passed from a client.
2. If you think a security token is exposed, you can create a new one or change it on our dashboard.
3. If a user requests a reward item more than the reward claim limit count, the server should reject the request.

If you want Nudge to send a security token and a reward claim limit count for a campaign to your server via RESTful API automatically, please email us at support@nudge.do

* * *

### Sales Promotion

You can promote in-game items to specific user segements. When a user taps on an action button of an in-app message, a purchase UI will appear. Our SDK will automatically detect if the user makes a purchase or not, then will update campaign performance KPIs in our dashboard in real time. Please note that [In-App Purchase Tracking](#in-app-purchase-tracking) feature is a prerequisite to this promotion feature.

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
  AdFrescaView *fresca = [AdFrescaView shared];
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

Nudge SDK detects if a user makes a purchase using our [In-App Purchase Tracking](#in-app-purchase-tracking) feature. For better measurement, you need to implement **cancelPromotionPurchase** method when the user cancelled during the purchase process or the transaction has failed. 

* * *

### Limited Time Offer

You can draw more attention from customers and create a sense of urgency with a limited time offer, which is a special sales promotion of **a hard currency item for a limited time period only**. Nudge SDK will display an interstitial with the remaining time on the top bar and will hide the intersitial when the time is over.

<img src="http://file.nudge.do/guide/sdk/LTO_interstitial_landscape_sample.jpg">

**Notice:** Please don't forget to add a nudge-icon font definition to Info.plist. (Please refer to [Installation](#installation) for more detail.)

Once a limited time offer is displayed in a marketing moment, it will be no longer available in any marketing moment. You need to use the folllowing code to retreive information on acitve limited time offers and display their interstitials again.

You can retreieve information of active limited time offers with **checkActiveLimitedTimeOffersWithCompletionHandler**, which will return an array of JSON strings with a remaining time and a unique value of the promotion item, sorted by remaining time in ascending order. With these information, you can display the shortest remaining time of an offer (and a number of active limited time offers if neccessary) in the game UI.
 
```objective-c

[[AdFrescaView shared] checkActiveLimitedTimeOffersWithCompletionHandler:^(NSString *jsonStr) {
  if (jsonStr) {
    // Parse JSON strings in the returned array and use them to display the remaining time and the number of active limited time offers if neccessary.
    // JSON example: [{"remaining_time_in_seconds":1184, "item_unique_value":"item_03"}, ...] 
  } else {
    // Nudge SDK will return nil when it fails to retrieve information of active limited time offers. You can re-try or display an error message to a user.
  }
}];

```

You can display interstitials of active limited time offers using **displayActiveLimitedTimeOffers** method with count parameter. Nudge SDK will display interstitials of the offers unless their remaining time is over.

```objective-c

[[AdFrescaView shared] displayActiveLimitedTimeOffers:1];

```

* * *

## Dynamic Targeting

### Custom Profile Attributes

Nudge SDK provides two tracking methods for custom profile attributes: Custom Parameter and Event Counter. Custom Parameter is used to track the current value of specific user attributes. (ex: level, current stage, facebook sign-in flag) while Event Counter is used to count a user's specific event in the app. (ex: play count, a number of gacha count)

You can create segements using custom paramters and/or event counters then target them for campaigns and/or monitor their activities in real time. You can achieve better campaign performance when targeting specific users with more filters. (Nudge SDK collect values of default filters such as device id, language, country, app version, run_count, purchase_count, etc so you don’t need to define those values as custom parameters or event counters.)

**NOTICE**: Please make sure that you set/increase custom parameters or increase event counters after a user signs in.

#### Custom Parameters

Set a custom parameter with a ‘Unique Key’ string value (e.g. "level", "facebook_flag") and a current value (integer or boolean) using **setCustomParameterWithValue** method. When your app supports signing in to multiple devices, Please make sure to set Custom Parameters with the values stored in your server after a user signs in, which can prevent data discrepancy in the situation that a game client was killed or paused on one device before finishing the sync between Nudge SDK and Nudge servers then she runs the app on other device.


```objective-c
- (void)onSignIn {
  AdFrescaView *fresca = [AdFrescaView shared];
  [fresca signIn:@"user_id"]; // or signInAsGuest:@"guest_id"
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forKey:@"level"];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forKey:@"facebook_flag"];
}
```

Please use the same method to update the value whenever its value changes.

```objective-c
- (void)onUserLevelChanged:(int)level {
  AdFrescaView *fresca = [AdFrescaView shared];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forKey:@"level"];
}   
```

Or you can increase the value of a custom parameter using **incrCustomParameterWithAmount** method with a ‘Unique Key’ string value and an increment if necessary.

```objective-c
- (void)onWinningStreak
{
  AdFrescaView *fresca = [AdFrescaView shared];   
  [fresca incrCustomParameterWithAmount:[NSNumber numberWithInt:1] forKey:@"winning_streak"];
}

- (void)onResetWinningStreak
{
  AdFrescaView *fresca = [AdFrescaView shared];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:0] forKey:@"winning_streak"];
}
```

#### Event Counters

Event Counters stores a total count of specific events. Use **incrEventCounterWithAmount** method with a ‘Unique Key’ string value and an increment if neccessary. 

```objective-c
- (void)onFinishStage {
  AdFrescaView *fresca = [AdFrescaView shared];   
  [fresca incrEventCounterWithAmount:[NSNumber numberWithLong:1] forKey:@"play_count"];  
}   
```

#### Manage Custom Profile Attributes

Nudge SDK transfers custom profile attributes to Nudge servers whenever necessary. But Nudge server will only store activated custom profile attributes so you need to activate them using [Dashboard](https://admin.adfresca.com). (You can activate up to 20 custom parameters and event counters in total.)

<img src="http://file.nudge.do/guide/sdk/manage_custom_profile_attributes.jpg">

Under 'Overview' tab, click 'Settings - Custom Profile Attrs' menu. Locate the unique key of a custom parameter or an event counter and set its 'Name' then you can activate it by clicking "Activate" button.

#### Stickiness Event Counters

A stickiness event counter is a special event counter to measure a user’s stickiness to your app. For example, if you set ‘play count’ as a stickiness custom parameter in a stage-based game, You can define user segments with 3 additional filters: ‘Today’s play count, ‘Total Play count in a week’, and ‘Average play count in a week’. Stickiness event counters will help you to classify user groups by their loyalty and to monitor their activities in real time. 

If you want to use stickiness event counters, please send an email to support@nudge.do after you activate your event counter in your dashboard.

* * *

### Marketing Moment

A Marketing Moment is where and when you want to engage your users. For example, you may need to deliver the message when the user completes a quest or enters into the in-app store. You can deliver the personalized and targeted message at a specific moment in real time.

To implement codes, simply call load method with passing marketing moment's index. You can get the marketing moment's index in our [Dashboard](https://admin.adfresca.com): 1) Select an App 2) In 'Overview' menu, click 'Settings - Marketing Moment' button. 

You will call the method after the moment has happened in the app.

```objective-c
- (void)userDidEnterItemStore {
  AdFrescaView *fresca = [AdFrescaView shared];   
  [fresca load:EVENT_INDEX_STORE_PAGE];    
  [fresca show];
} 

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView shared];   
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
  AdFrescaView *fresca = [AdFrescaView shared];
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
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  if (!fresca.hidden && fresca.userClicked) {
    [fresca close];
  }
}
```

### Timeout Interval

You can set a timeout interval for a marketing moment request. If the message is not loaded within this time interval, the message won't be displayed to users and our SDK will return control to your app.

Default is 5 seconds and you can set from 1 seconds to 5 seconds.

```objective-c
AdFrescaView *fresca = [AdFrescaView shared];  
fresca.timeoutInterval = 3 // # secs  
[fresca load];
[fresca show];
```

* * *

## Reference

### Deep Link

You can set your own URL Schema as 'Deep Link' of the campaigns. So, you can navigate your users to a specific page or do some custom actions when a user clicks the image message. 

1. Set your custom url schemes in Info.plist as follows

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

  Check your url schemes to check app install in Info.plist as follows

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  In this case, you should set CPI Identifier value of advertising app to "myapp://" in our dashboard (Overview > App Store Edit).

  For iOS, url schema value may be duplicated with other apps, so be careful to choose an unique value.

  For Incentivized CPI Campaigns, our SDK Installation of the advertising app is not required. You can only set URL Schema to use app's install.

  However, if you use Incentivized CPA Campaigns, our SDK installation is required and you should also implement 'Marketing Moment' feature to check a reward condition. For example, when you set the reward condition to check 'Tutorial Complete' moment, you should call the marketing moment method to inform your user achieved the goal.
    
  ```objective-c
  - (void)didTutorialComplete {
    AdFrescaView *fresca = [AdFrescaView shared];   
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
  [[AdFrescaView sharedAdView] setUseIFVOnly:YES];
}
```

Please contact us if you have any concerns or issues on this section.

* * *

## Troubleshooting

Duplicated Symbol Error of SBJson may occur if you already have SBJson in your project.

<img src="https://adfresca.zendesk.com/attachments/token/ikafbcqjnj9jbak/?name=6666.epng">

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
- **v1.6.6 _(2016/03/11 Updated)_**
  - Added [Limited Time Offer](#limited-time-offer) feature.
- v1.6.5 (2016/03/10 Updated)
  - Revived the deprecated **incrCustomParameterWithAmount** method.
- v1.6.4 (2016/03/09 Updated)
  - Fixed a bug in In-App Purchase Tracking
- v1.6.3 (2016/02/27 Updated)
  - Added incrEventCounterWithAmount method and deprecated incrCustomParameterWithAmount. Please refer to [Custom Profile Attributes](#custom-profile-attributes) section.
- v1.6.2 (2016/01/23 Updated)
  - Added OnRewardClaim and finishRewardClaim methods and checkRewardItems has been deprecated. Please refer to [Give Reward](#give-reward) section.
- v1.5.6 (2015/06/02 Updated)
  - Support 'Push Reward Campaign'. Please refer to [Push Messaging](#push-messaging) section to revise your 'if statement' code in 'didReceiveRemoteNotification' event
- v1.5.5
  - For [In-App Purchase Tracking](#in-app-purchase-tracking), fixed an item name issue with '%' character. 
- v1.5.4
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

