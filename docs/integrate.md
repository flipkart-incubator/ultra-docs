# Integration with Ultra
---
# Steps

These steps act as a comprehensive guide for integrating your service using Ultra platform. If you wish to have a visual walk-through before following the detailed instructions, check [here](demo.md). For any issues or queries encountered anytime during this process, please reach out to [us](contact.md).

## 1: Choose your UI platform

Ultra supports [React Native](https://facebook.github.io/react-native/) as well as a simple HTML /PWA. You can build your application using either of these and we can help you in making the best choice by spotlighting a comparison between the two as follows:

| **React Native** | **HTML** |
|--------------|------|
| Supports JavaScript code | Supports JavaScript code |
| Extraordinary performance | Satisfactory performance |
| Highly optimised for mobile applications | Average optimisation |
| Bundle is delivered from Flipkart CDN, only differential components are downloaded and is cached within the Flipkart | Bundle and assets follow cache-control headers and might be slower to render even from cache |

### Ultra’s interaction with UI platform

#### Using React Native

Ultra pulls in the React Native bundles from your GitHub repository and then delivers it to Flipkart dynamically which we prefer to call as “Over-The-Air”. Flipkart receives these React Native JavaScript bundles via Dynamic Update Service (DUS). We recommend you to visit [this link](https://github.com/Flipkart/DUS) to understand how DUS works.

Once DUS fetches your bundle within the Flipkart, it loads the React Native within a React Fragment from where you can navigate to the other pages.

> This approach helps you in composing mobile applications quick, having a rich UI and by using only JavaScripts. You are free to pick your fundamental UI building blocks and merge them simultaneously using JavaScript and React to produce a real mobile application. But, limit the usage of the native bridges within Flipkart area only. Since one cannot deliver the local assets such as images/videos dynamically like JavaScript, you have to upload these assets to your CDN separately and reference them within the JavaScript code.

#### Using HTML

Ultra launches the webpage inside a mobile’s web-view. Here, you may pick your existing mobile website and mould it for Ultra after few tweaks. 

> This approach helps you in building a decent mobile application but it does not appear as realistic and responsive as the one developed using React Native.

!!!note One important point to keep in mind is that both the approaches do not allow you to navigate away from your main domain. Let us suppose your application is hosted on some domain for example, `ABC.com` and the site contains a hyperlink that navigates control to some other domain say, `ABC.org`. In such case, the site does not work and throws a `Security Error` message. If you have a similar situation in your application, please contact [us](contact.md) as we have got a way for bypassing this limitation by whitelisting the domains you own.

## 2: Integrate Ultra JavaScript SDK
After you have made up your mind on which UI platform to work with (either React Native or HTML/PWA), include Ultra’s JavaScript SDK into your application. This SDK helps you build applications that run within Flipkart app and give access to the Ultra’s JavaScript methods required for logins, authorizations (OAuth) and payments.

!!!note The logic presented here works in both React Native and HTML/PWA. These methods are asynchronous and always return a [promise](https://javascript.info/promise-basics) that gets resolved with some values except few other JavaScript methods that have no return value.

Next, [add the dependency](clients.md#step-1) for the SDK. [Contact us](contact.md) for generating the parameters specific to your application i.e. clientID and secret to access the Ultra APIs.

After adding the dependency, [initialize the SDK](clients.md#step-2) using the above parameters.

## 3: Ultra User Login Process
In this section, you learn how to fetch user details securely from Flipkart’s domain. A user can gain entry into your application only after allowing permissions to share their information (such as name, email, phone number etc.). This step guides you about the generation of grant token, fetching user’s information and login the user into your application.

### Generate grant token
You might have come across the terms which are very common in the technical world of authorization (OAuth) world such as “Granted Scopes” and “Grant tokens”. To know more, read [this](https://alexbilbie.com/guide-to-oauth-2-grants/).

Before fetching any data of the user, call the `getToken()` method and supply the list of parameters that includes the list of user resources you want to gather i.e. user’s name, email, phone number etc. After you call this method, it renders an `Allow/Deny permissions` prompt to the user as shown below:
![Permission prompt](img/permissions.png)

!!!note This permission popup has a special behavior when a single scope is requested and user has some unverified pre-filled value. For example, if you have asked permission for a user.email scope and the user has a unverified email address in Flipkart, it automatically initiates the email verification for the user. This helps user in avoiding any extra clicks.

Following use case diagram illustrates the entire scope selection process:


Legend:


We suggest to use flags like `isMandatory` and `shouldVerify` to control whether you want these permissions to be mandatory. Like here, `isMandatory` is a boolean type variable that you enable if you want the scope to be mandatory and must be filled by the user. `shouldVerify` is also a boolean type variable that you enable if you want that the same scope to be verified as well. To cite an example, if your `getToken()` call is `getToken(['scope':'user.email', 'isMandatory':true, 'shouldVerify':true])`, then it means that user is not allowed to grant permissions without entering the email address as well as verifying it.

!!!note You should set both `isMandatory` and `shouldVerify` to true only when you absolutely require the user’s verified email address because you may observe that if such constraints are imposed on users, there occurs a significant drop in the number of users who grant permissions.

!!!note Since most of the phone numbers of Flipkart users are OTP verified, there would be no requirement of such verification within this flow. But if there are any users who are not already OTP verified, a prompt for the OTP verification process appears to them before the control comes back to you.

!!!note Once the user has allowed the permissions, a [promise](https://javascript.info/promise-basics) is returned which contains the information about the token that is resolved.

The `getToken()` method returns a list of both allowed as well as rejected permissions that depend upon the user’s response to the `Allow/Deny permissions` prompt. It also returns an access token that has to be passed to your server so that it can hit the Flipkart’s API to read the user’s actual information. This information has to be read from one server by another server only using the provided `grantToken` so that it remains protected and secured.
