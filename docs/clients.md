# Ultra Client Side SDK documentation

> Latest JS SDK version : [1.0.6](https://www.npmjs.com/package/fk-platform-sdk)

## Overview


This SDK enables developers to build applications that run inside Flipkart app.

All the methods mentioned here will work with both React Native and Webview. 
All the methods are asynchronous in nature and will always return a promise that gets resolved with the values. Fire and forget calls are an exception where you may not care about the response.

## Getting Started
#### Add the dependency
If using node, add this repository as an npm package (Both webview and react-native)

```
npm install --save fk-platform-sdk
```

You can also visit [the NPM](https://www.npmjs.com/package/fk-platform-sdk) page for the SDK 

Alternatively, if you are using webview only you can also include the following script directly inside a `<script>` tag:

```
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fkext-browser-min@1.0.6.js
```

#### Initialize the SDK
Import SDK and create a new platform instance. You will need to provide clientId given to you by Flipkart.

In Node Environment:


##### For WEBVIEW
``` js
import FKPlatform from "fk-platform-sdk/web"
let fkPlatform = new FKPlatform(clientId);
```

##### For REACT NATIVE
``` js
import FKPlatform from "fk-platform-sdk"
let fkPlatform = new FKPlatform(clientId);

```

In Browser (Without using Node):

```js
var fkPlatform = FKExtension.newPlatformInstance(clientID);
```

Once this is done you can start using modules.

!!!note
    You should call `FKPlatform.isPlatformAvailable()` or, `window.FKExtension && FKExtension.isPlatformAvailable()` to check if you're inside Flipkart platform. It is recommended not to do any checks in partner code. More details [here](#detecting-flipkart-environment)

## Modules
### Permissions Module
```js
let permissionsModule = fkPlatform.getModuleHelper().getPermissionsModule()

//To get scopes:
const SCOPES = permissionsModule.getScopes();
```
**Available Scopes:**
```
SCOPES.USER_EMAIL,
SCOPES.USER_MOBILE,
SCOPES.USER_NAME
```
**Methods:**

```
getToken: (permissions: ScopeAccessRequest[]) => Promise<NativeModuleResponse<PermissionsManagerResponse>>
```

Relevant interfaces:

```js
interface ScopeAccessRequest {
    scope: Scopes;
    isMandatory?: boolean;
    shouldVerify?: boolean;
}

interface NativeModuleResponse<T> {
    result: T,
    grantToken?: string | null
}

interface PermissionsManagerResponse { [key: Scopes]: boolean; }
```

**Sample**

```javascript
var scopeReq = [{"scope":"user.email","isMandatory":true,"shouldVerify":false},{"scope":"user.mobile","isMandatory":false,"shouldVerify":false},{"scope":"user.name","isMandatory":false,"shouldVerify":false}];
fkPlatform.getModuleHelper().getPermissionsModule().getToken(scopeReq).then(
function (e) {
    console.log("Your grant token is: " + e.grantToken);
}).catch(
function (e) {
    console.log(e.message);
}
```

`isMandatory` is a boolean which says whether you want the scope to be mandatorily filled by the user

additionally `shouldVerify` is boolean which says whether you want the scope to be mandatorily verified as well

for e.g if you call `getToken(['scope':'user.email', 'isMandatory':true, 'shouldVerify':true])`, then that means user cannot grant permissions without filling his email address and also verifying it. You can use this on situations where you know that he user has an unverified email address and want to trigger email verification for the user.
Note that the flags `isMandatory` and `shouldVerify` should be set to `true` only when you ABSOLUTELY need an email address which has to be verified as well, because you could see a significant drop off of permission grants when such constraints are imposed on users. 

Also note that the permission popup has a special behaviour when a single scope is requested and also has a unverified value prefilled by the user. For e.g, when you ask permission for a user.email scope, and the user already has an unverified email address in our system, then the UI will automatically initiate the email verfication flow. This is done to avoid an extra click for the user.

The method `getToken` returns you both list of allowed and rejected permissions which depend on user response. It also returns an access token which you should be passing to your server using which it can hit Flipkart api to read relevant info. For security reasons secure info should only be read server to server using given grantToken.

#### Promise resolution
Promise fails and enters the `catch` block only if the user dismisses the permission popup (bottomsheet) or the SDK network calls failed.
If user denies a permission, the promise will be resolved successfully and you get a `grantToken`.
This is because there could be partial permission denial as well. For e.g user denied phone number, but granted email address
Hence all the denials as well as accepts will still be treated as success from a token generation point of view.
If you want to check if user denied a permission, you can check the `NativeModuleResponse` result to see if the permissions are granted or not.

If the promise fails, Catch block function is invoked with an object which has `message` and `code`.
Possible errors are [here](#handling-errors). 

#### Handling denials
You can analyze the `NativeModuleResponse.result` to know the exact list of scopes which got denied.
The ideal way of handling denials is to show a page to the user as to why these permissions are mandatory to continue. Refer [Android's user guide](https://developer.android.com/topic/performance/vitals/permissions) on this topic on knowing more about this.

### Navigation Module
Common methods which will enable your application to control their exit behaviour and deeplink into other parts of the app.

```js
let navigationModule = fkPlatform.getModuleHelper().getNavigationModule()
```

**Methods:**
The following methods are supported on the navigation module.

#### Exit Session
```js
exitSession(): void
```

Closes the application and takes user back to the page from where the app was launched.

#### Exit to Homepage
```js
exitToHomePage(): void
```

Closes the application and takes user to Flipkart homepage.

#### Start Payment
```js

startPayment(paymentToken: string): void
```

Launches Flipkart payment screen for given payment token. The payment token has to be generated by your server by hitting Flipkart PG server API. Once you call the startPayment() method on the client, the container will switch to Payments. Once payments is successful/failed the container switches back to ultra and you will get a callback.

This callback will happen through a page load to the `successfulCallBackUrl` or `failureCallBackUrl` specified when payment token was generated. This URL will be opened within the payment container and is supposed to validate the payment response from FKPG (POST params are to be validated) and then do a container switch back to ultra. A container switch can be initiated by a HTTP redirect to the following URL

```
fapp://action?value={"screenType":"ultra","params":{"url":"<URLENCODED_URL_TO_OPEN_WITHIN_ULTRA>","clientId":"<YOUR_CLIENT_ID>","postPaymentFlow":true,"success":<TRUE_OR_FALSE>}}
```

On Webview, once the container switches, the url specified in the params for `fapp://` will load as a regular page load. This loaded page should render the `Order confirmation` page for your service.

On ReactNative, once the container switches, you will get a custom event which will be emitted via RN Event Emitter.
The name of the event is `loadUri` which will contain the URL you specified as params in the above `fapp://` redirect.

#### Clear history 
> This is a Webview Only method

```js
clearHistory()?: void
```

Clears both forward and backward history such that only the existing page remains on webview.


#### Navigate to Flipkart

```js
navigateToFlipkart(url : string): void
```
> Available only on JS SDK 1.0.0-beta.1 onwards, FK app v6.7 onwards and ultra SDK v1.9 onwards. Please make sure you [do user-agent version check](clients/#detecting-presence-of-a-certain-feature) before accessing this API.

Navigates to a Flipkart page given the URL. The URL should be a `fapp://` URL.

The following URLs are supported.

| Description              | URL                                                                                                                                                                          |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Flipkart plus coins page | `fapp://action?value={"params": {"screenName": "LOCKED_COINS","valid":true},"screenType": "multiWidgetPage","type":"NAVIGATION","url": "/locked-coins"}` |

#### Notify page location change 
> This should be called only if you are using React Native.

> Available only on JS SDK 1.0.0-beta.1 onwards, FK app v6.7 onwards and ultra SDK v1.9 onwards
Please make sure you [do user-agent version check](clients/#detecting-presence-of-a-certain-feature) before accessing this API.
```js
notifyPageLocationChange(currentUri: string, isBackNavigation: boolean): void
```
```
currentUri : Some representation of the current page in the URI format. E.g : http://example.com/search
isBackNavigation: true if the page opened due to a back navigation. 
```

For basic analytics, Flipkart tries to optimize the funnel. For this purpose, its important to understand your funnel using events whenever a page change happens. On webviews, a page change can be inferred using the change in URL. But on react native, there is no way to infer this. Hence you are required to call this method to let flipkart's analytics to know that the user has navigated to a page.

Call this whenever the screen changes. For e.g once on homepage and then on search page and then on details page. Also call this if your screen changes when the back button is pressed.


### Contacts Module 
> Available from flipkart app v6.3, ultra v1.4.1

Helps in launching a UI that allows user to select a contact from his address book as well as get contact details.

```js
let contactModule = fkPlatform.getModuleHelper().getContactsModule()
```
**Methods:**
#### Picker
```js
pickPhoneNumber(): Promise<NativeModuleResponse<Contact>>
```

Launches UI to pick a phone number and returns the same.

#### Get Contact Name
```js
getContactInfo(phoneNumbers: string[]): Promise<NativeModuleResponse<{[key: string]: Contact;}>>
```

Fetches names and other info against given list of phone numbers. Keys of the returned map will be given phone numbers. 

### Analytics Module
Use this module if you wish to send user generated actions to ultra analytics library. For detailed description of this SDK refer [this page](analytics-overview.md).

### Relevant types

```js
interface NativeModuleResponse<T> {
    result: T,
    grantToken?: string | null
}

interface Contact {
    name: string
    phoneNumber: string
}
```

### Handling Errors
Every promise reject contains an error code and a message:
Type:
```js
interface NativeModuleError {
    message: string;
    errorCode: number;
}
```

Error Codes:
```js
ERROR_CODE_UNKNOWN = 0;
ERROR_CODE_NETWORK_ERROR = 1001;
ERROR_CODE_JSON_PARSE_ERROR = 1002;
ERROR_CODE_INVALID_PERMISSIONS = 1003;
ERROR_CODE_USER_DISMISS = 1004;
ERROR_CODE_BAD_PERMISSION_REQUEST = 1005;
ERROR_CODE_BAD_CONTACT_FETCH_REQUEST = 1006;
ERROR_CODE_NATIVE_PERMISSIONS_MISSING = 1007;

```

### Detecting flipkart environment
If you want to detect if your webpage is running within Flipkart's ultra environment, you have three ways.

1. [PREFERRED] Use JS SDK's `FKPlatform.isPlatformAvailable()` method.

2. [WEBVIEW-ONLY] Use the user agent which contains "Ultra". This method is useful if you want to inject the JS SDK only when running within flipkart's ultra environment. An example user agent string 
` Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]`
Note that if you have service workers making requests, this user agent will not be present.

3. [WEBVIEW-ONLY] Use X-Requested-With header which will contain the value 'com.flipkart.android'. This will work for all requests including service workers. This is the least preferred method.

### Detecting presence of a certain feature
> User Agent is available in both webview and react native.

Certain APIs are only available on certain versions of the FK app.
Every API mentioned above will have a note specifying which version of FK app contains it.
For backward compatibility, you should parse the user agent string and figure out the app version.

For webview, you can access user agent via `navigator.userAgent` and on react native you can use `FKPlatform.getUserAgent()`

User agent string in ultra is of the following format:

```
Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]
```

In the above user agent string, if you see the last part, 5.16 is the version name and 850000 is version code for flipkart app, 1.4.2 is version name and 4 is version code for ultra SDK. 

Based on just the ultra version name (1.4.2) you can check if certain newly added modules are present or not.

### Handling file uploads and downloads on webview 
> Available from flipkart app v6.3, ultra v1.3.7

Starting with Flipkart app version 6.3, ultra version 1.3.7, file uploads and downloads are present.
For downloads, ensure that 'download' attribute is not used. For e.g `<a href='abcd.pdf' download='xyz.pdf' />` will not work.
Also apis like FileSaver.js or blob based APIs dont work due to limitations in webview.
For uploads, there are no restrictions.

### Debugging Webview
If you see API calls failing or images/page not loading issues, you can also debug ultra within Chrome debugger. The APK which is shared with partners have debugging enabled for webviews. This means you will able to connect to the chrome instance via Remote debugging. [See here](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/)

### Debugging React Native
The special APK shared with partners has a way to run your react native code within ultra by pointing to your IP address of your react native packager. This way you can actively develop as well as debug the code without any special tools. This mechanism runs your react native bundle without using [Flipkart DUS](https://github.com/Flipkart/DUS).
