# Ultra Client Side SDK documentation
Here are the details of the Client side APIs you need to incorporate while integration of your app with Ultra.

> Use this latest version of JS SDK: [1.0.0-beta](https://www.npmjs.com/package/fk-platform-sdk)

## Integrate JS SDK
#### Add the dependency
If you are using a node, add the following repository as an NPM package for both WebView and React Native platforms. Visit [here](https://www.npmjs.com/package/fk-platform-sdk) to get the latest SDK for this repository.

```
npm install --save fk-platform-sdk
```

Or if you are using only a web browser without node, include the following script directly inside the `<script>` tag:

```
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fkext-browser-min@1.0.0-beta.1.js
```

#### Initialize the SDK
Import the SDK as shown below. After importing, create a new platform instance using the `clientId` already shared with you by Flipkart. If you are not aware of the `clientId`, [check here]() for details.

##### WebView (with Node):

***WebView:***
``` js
import FKPlatform from "fk-platform-sdk/web"
let fkPlatform = new FKPlatform(clientId);
```
***React Native:***
``` js
import FKPlatform from "fk-platform-sdk"
let fkPlatform = new FKPlatform(clientId);
```

##### Browser (without Node):

```js
var fkPlatform = FKExtension.newPlatformInstance(clientID);
```

!!!note
    Call `FKPlatform.isPlatformAvailable()` or `window.FKExtension && FKExtension.isPlatformAvailable()` to check if you are inside Flipkart platform. We recommend you not to include any checks in your own (partner’s) code. To know more, [check this](#detecting-flipkart-environment).

## Modules
### Permissions
In this module, we have listed the functions you use for generating `grantToken`, getting the permission scopes, making a call to `getToken()`, resolving promise and handling denials from users.

**Get the list of permissions and permission scopes:**
```js
let permissionsModule = fkPlatform.getModuleHelper().getPermissionsModule()
const SCOPES = permissionsModule.getScopes();  //Get scopes
```

**List of Scopes:**
```
SCOPES.USER_EMAIL,
SCOPES.USER_MOBILE,
SCOPES.USER_NAME
```

**getToken Method Call:**
```
getToken: (permissions: ScopeAccessRequest[]) => Promise<NativeModuleResponse<PermissionsManagerResponse>>
```

**Relevant Interfaces:**
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

**Sample request for generating Grant Token:**
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

Here, in the sample request, `isMandatory` is a boolean type variable you set to `true` if you want the scope to be mandatory and a user must fill to proceed. `shouldVerify` is another boolean type variable you set if you want that the user must verify the same scope. Suppose you made a `getToken()` call: `getToken(['scope':'user.email', 'isMandatory':true, 'shouldVerify':true])`. This means that user can not grant permissions without entering and verifying a valid email address.

!!!note
    Set both `isMandatory` and `shouldVerify` to true only when there is an absolute need of the user’s verified email address at your side. This is because of the observation that if we impose such constraints on users, it results in a significant drop in the number of users who proceed beyond the grant permissions screen.

**Promise Resolution and Handling Denials**
If the user denies the permissions, the [`promise`]() gets resolved and you receive a `grantToken`. This case might happen with partial denials from the user i.e. when user denies the permission to allow the phone number but accepts to permit the email address. Hence, regarding the token generation point of view, system treats each acceptance and denial as a success. Check `NativeModuleResponse.result` attribute to view the actual list of accepted and rejected permissions.

!!!note
    An ideal way of handling denials from the user is to show a page that explains why these permissions are mandatory. To know more, refer [Android's user guide](https://developer.android.com/topic/performance/vitals/permissions).

After the promise has failed, it invokes the `catch` block function with an object that has a `message` and `code`. Click [here](#handling-errors) for the list of possible errors.

### Navigation
In this module, we have listed the methods through which you control your application’s exit behaviour for different scenarios and navigation of deep-links to the other parts of the app.

**Get the Navigation Module:**
```js
let navigationModule = fkPlatform.getModuleHelper().getNavigationModule()
```

**Exit Session:**
```js
exitSession(): void
// It closes your application and redirects user back to the page where you launched your app.
```

**Exit to Homepage:**
```js
exitToHomePage(): void
// It closes your app and redirects a user to Flipkart’s home page.
```

**Start Payment:**
```js
startPayment(paymentToken: string): void
// It launches Flipkart’s payment screen for the specified payment token.
```

After your server hits the `Flipkart’s Payment Gateway (FKPG)` server [API](), it generates the payment token. Ultra’s container switches to the Payment Gateway screen after you call `startPayment()` on the client side. Once the payment cycle completes, irrespective of a success or a failure, you receive a callback and the container switches back to Ultra. This callback happens through a `PageLoad()` event of either `successfulCallBackUrl` or `failureCallBackUrl` page at the time of payment token generation. This URL is accountable for validating the response (POST parameters) from FKPG within the Payment’s container. Then, it goes back to the Ultra’s container. The following URL starts the container switching:
```
fapp://action?value={"screenType":"ultra","params":{"url":"<URLENCODED_URL_TO_OPEN_WITHIN_ULTRA>","clientId":"<YOUR_CLIENT_ID>","postPaymentFlow":true,"success":<TRUE_OR_FALSE>}}
```

After the container’s switch:
| On WebView | On React Native |
| --- | --- |
| The URL specified in the `url` parameter of the above `fapp://` redirect loads normally through the regular `PageLoad()` event and renders the Order confirmation page for your service. | The React Native Event Emitter emits a custom event `loadUri` which contains the URL specified in the `url` parameter of the above `fapp://` redirect. |

**Clear History:**
```js
clearHistory()?: void

// Used for WebView only.
// It clears the history of all the forward and backward pages and maintains the existing page details only.
```

**Navigate to Flipkart app:**
```js
navigateToFlipkart(url : string): void
// It navigates to any Flipkart page based on the URL passed in the parameters. The URL should be a ‘fapp://’ redirect.
```

Following is the list of URLs supported by this method:
| Page Description              | URL                                                                                                                                                                          |
|------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Flipkart plus coins page | `fapp://action?value={"params": {"screenName": "LOCKED_COINS","valid":true},"screenType": "multiWidgetPage","type":"NAVIGATION","url": "/locked-coins"}` |

!!!note
    This method is available only for JS SDK 1.0.0-beta.1, Flipkart app v6.7 and Ultra SDK v1.9 onwards. Ensure that you have done a [user-agent version check](clients/#detecting-presence-of-a-certain-feature) before accessing this API.

**Notify page location change:** 
```js
notifyPageLocationChange(currentUri: string, isBackNavigation: boolean): void
```
```
where ‘currentUri’ is the representation of the current page in URI format. For example: http://example.com/search
‘isBackNavigation’ is ‘true’ if the page opens because of back navigation.
// It is specific to React Native platform only. 
```

Flipkart keeps trying to optimize the user funnel for basic analytics. Whenever a page change occurs, it is important for you to understand about your funnel using events. In case of WebViews, a change in URL infers that there is a change in the existing page. But, for React Native, there is no way to find it out. Thus, for React Native platform, you need to call this method every time your screen changes so that our analytics library gets notified when the user has navigated to some page that might have changed. We suggest using this method on your Home page, Search page, Details page and when the user presses back button if it results in a change of screen.

!!!note
    This method is available only for JS SDK 1.0.0-beta.1, Flipkart app v6.7 and Ultra SDK v1.9 onwards. Ensure that you have done a [user-agent version check](clients/#detecting-presence-of-a-certain-feature) before accessing this API.


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
