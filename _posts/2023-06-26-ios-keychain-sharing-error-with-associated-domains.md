---
title: "iOS Keychain Sharing Error with Associated Domains"
---

I’m working on an iOS app that includes a share extension, and I set up keychain sharing between the app and the extension so that the extension has access to the user’s access token. However, after I set up an associated domain to allow 1Password autofilling, the share extension could no longer access the keychain group. To fix this, I had to configure the same associated domain for the share extension.

The symptoms I saw while troubleshooting this were confusing, though, so I wanted to share some details.

[This StackOverflow answer](https://stackoverflow.com/a/41588311/477480) indicates that the way to share a keychain value between an iOS share extension and its containing app is to use keychain sharing.

For background on how keychain sharing works, Apple has a helpful document, [Sharing Access to Keychain Items Among a Collection of Apps](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps).

I followed the steps in these two documents: added the Keychain Sharing capability to both the containing app and the share extension, and configuring the same keychain group in both. As a result, my share extension was able to read the keychain value I set in the containing app.

Next, I wanted to implement 1Password autofill. [1Password’s iPhone apps](https://support.1password.com/ios-autofill/#get-help) document says:

> If you’re an iOS app developer, [set up your app’s associated domains](https://developer.apple.com/documentation/security/password_autofill/setting_up_an_app_s_associated_domains) to make it easier for your users to save and fill with 1Password.

The link goes to an Apple document about setting up associated domains. At first I did this only in the containing app. Once I did this, 1Password correctly suggested my account for the app.

Shortly after doing this I noticed errors in the share extension: it gave me an error while loading the access token. Specifically, it gave error -34018, which is `errSecMissingEntitlement`. This suggested that the share extension didn’t have access to the keychain group. Why would this be?

I put the same keychain group value in the Keychain Sharing capability settings for the app and extension. But as the “Sharing Access to Keychain Items” doc says, a prefix is automatically added at runtime, so when I wrote the code to save the value to the keychain group I had to add the prefix as well.

As I dug in to make sure I was using the right prefix, this is where things started to get confusing. What prefix is used?

- The [Sharing Access to Keychain Items](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps) doc says “Xcode automatically prefixes keychain groups with your team ID”. Both the containing app and the share extension have the same Apple Developer team ID, so this should be fine.
- However, the [Configuring keychain sharing](https://developer.apple.com/documentation/xcode/configuring-keychain-sharing/) doc says something different: “Xcode prepends the name of each keychain group with the $(AppIdentifierPrefix) build variable, which it automatically resolves at build time.” Sure enough, in the `.entitlements` files Xcode created when I set up keychain groups, `$(AppIdentifierPrefix)` shows at the start of the group name. What is `AppIdentifierPrefix` and how does it relate to the team ID?
- I found the answer mentioned in passing in [Troubleshooting -34018 Keychain Errors](https://developer.apple.com/forums/thread/114456): “your App ID prefix (in most cases this is your Team ID)”

So the app ID prefix is “in most cases” the team ID, but in some cases not. That sounded like it could be the cause: maybe in the app or the extension, the app ID prefix was *not* the team ID.

To find out if it was changed in this case, I followed the instructions in [Access App Identifier Prefix programmatically](https://stackoverflow.com/questions/11726672/access-app-identifier-prefix-programmatically) to check the app ID prefix at runtime. Sure enough, the app ID prefix in the containing app was the same as my team ID, but the app ID prefix in the share extension was a different value.

I wondered if adding the associated domain might have caused the app ID prefix to change. I didn’t have any concrete reason to think so; it was just a guess. This guess was informed by (1) associated domain and keychain sharing are both capabilities set in the same place in Xcode, and (2) they are both stored in the `.entitlements` files. I wasn’t able to find any information that said that the app ID prefix changes when the associated domains differ between the two, though.

To test out this guess, I set up the same associated domain in the share extension. After I did so, the share extension’s app ID prefix switched back to the team ID, and it was able to read the keychain entry.
