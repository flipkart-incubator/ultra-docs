# Ultra Client Side SDK

## Overview
This SDK enables developers to build applications that run inside Flipkart app.

All the methods mentioned here will work with both React Native and Webview. 
All the methods are asynchronous in nature and will always return a promise that gets resolved with the values. Fire and forget calls are an exception where you may not care about the response.

Note: Not open to all, access enabled only for partners.


## Getting Started
### Integration steps:
### Step 1)
If using node, add this repository as an npm package

```
npm install --save fk-platform-sdk
```

Alternatively, you can also include the following script directly:

`http://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fkext-browser-min@0.1.5.js (Version 0.1.5)`

### Step 2)
Import SDK and create a new platform instance. You will need to provide clientId given to you by Flipkart.

In Node Environment:

```js
import FKPlatform from "fk-platform-sdk"
let fkPlatform = new FKPlatform(clientId);
```

In Browser:

```js
var fkPlatform = FKExtension.newPlatformInstance(clientID);
```

Post this you can start using modules.

Note: You should call `FKPlatform.isPlatformAvailable()` or, `window.FKExtension && FKExtension.isPlatformAvailable()` to check if you're inside Flipkart platform. It is recommended not to do any checks in partner code.

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

`getToken: (permissions: ScopeAccessRequest[]) => Promise<NativeModuleResponse<PermissionsManagerResponse>>`

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

`isMandatory` is a boolean which says whether you want the scope to be mandatorily filled by the user

additionally `shouldVerify` is boolean which says whether you want the scope to be mandatorily verified as well

for e.g if you call `getToken(['scope':'user.email', 'isMandatory':true, 'shouldVerify':true])`, then that means user cannot grant permissions without filling his email address and also verifying it. You can use this on situations where you know that he user has an unverified email address and want to trigger email verification for the user.
Note that the flags `isMandatory` and `shouldVerify` should be set to `true` only when you ABSOLUTELY need an email address which has to be verified as well, because you could see a significant drop off of permission grants when such constraints are imposed on users. 

Also note that the permission popup has a special behaviour when a single scope is requested and also has a unverified value prefilled by the user. For e.g, when you ask permission for a user.email scope, and the user already has an unverified email address in our system, then the UI will automatically initiate the email verfication flow. This is done to avoid an extra click for the user.

The method `getToken` returns you both list of allowed and rejected permissions which depend on user response. It also returns an access token which you should be passing to your server using which it can hit Flipkart api to read relevant info. For security reasons secure info should only be read server to server using given grantToken.

Promise resolution : 
Promise fails and enters the `catch` block only if the user dismisses the permission popup (bottomsheet) or the SDK network calls failed.
If user denies a permission, the promise will be resolved successfully and you get a `grantToken`.
This is because there could be partial permission denial as well. For e.g user denied phone number, but granted email address
Hence all the denials as well as accepts will still be treated as success from a token generation point of view.
If you want to check if user denied a permission, you can check the `NativeModuleResponse` result to see if the permissions are granted or not.

If the promise fails, Catch block function is invoked with an object which has `message` and `code`.
Possible errors are : 
```
ERROR_CODE_UNKNOWN = 0;
ERROR_CODE_NETWORK_ERROR = 1001;
ERROR_CODE_JSON_PARSE_ERROR = 1002;
ERROR_CODE_INVALID_PERMISSIONS = 1003;
ERROR_CODE_USER_DISMISS = 1004;
ERROR_CODE_BAD_PERMISSION_REQUEST = 1005;
```

### Navigation Module
Common methods will enabled application to control their exit behaviour and deeplink into other parts of the app.

```js
let navigationModule = fkPlatform.getModuleHelper().getNavigationModule()
```
**Methods:**

`exitSession(): void`

Closes the application and takes user back to the page from where the app was launched.

`exitToHomePage(): void`

Closes the application and takes user to Flipkart homepage.

`startPayment(paymentToken: string): void`

Launches Flipkart payment screen for given token.

`clearHistory()?: void`

Clears both forward and backward history such that only the existing page remains.

### Contacts Module (from flipkart app v6.3, ultra v1.4.1)
Helps in launching a UI that allows user to select a contact from his address book.

```js
let contactModule = fkPlatform.getModuleHelper().getContactsModule()
```
**Methods:**

`pickPhoneNumber(): Promise<NativeModuleResponse<Contact>>`

Launches UI to pick a phone number and returns the same.

`getContactInfo(phoneNumbers: string[]): Promise<NativeModuleResponse<{[key: string]: Contact;}>>`

Fetches names and other info against given list of phone numbers. Keys of the returned map will be given phone numbers. 

Relevant types:

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

Codes:
0: Unknown error
```

### Detecting flipkart environment
If you want to detect if your webpage is running within Flipkart's ultra environment, you have three ways.
1. [PREFERRED] Use JS SDK's `FKPlatform.isPlatformAvailable()` method.
2. Use the user agent which contains "Ultra". This method is useful if you want to inject the JS SDK only when running within flipkart's ultra environment. An example user agent string 
` Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]`
Note that if you have service workers making requests, this user agent will not be present.
3. Use X-Requested-With header which will contain the value 'com.flipkart.android'. This will work for all requests including service workers. This is the least preferred method.

### Detecting presence of a certain feature
User agent string in ultra is of the following format:
` Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]`
In the above user agent string, 5.16 is the version name and 850000 is version code for flipkart app, 1.4.2 is version name and 4 is version code for ultra SDK. 

Based on just the ultra version name (1.4.2) you can check if certain newly added modules are present or not.

### Handling file uploads and downloads (from flipkart app v6.3, ultra v1.3.7)
Starting with Flipkart app version 6.3, ultra version 1.3.7, file uploads and downloads are present.
For downloads, ensure that 'download' attribute is not used. For e.g `<a href='abcd.pdf' download='xyz.pdf' />` will not work.
Also apis like FileSaver.js or blob based APIs dont work due to limitations in webview.
For uploads, there are no restrictions.

