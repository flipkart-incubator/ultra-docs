# Ultra Client Side SDK documentation
Here are the details of the Client side APIs you need to incorporate while integration of your app with Ultra.

> Use the latest version of JS SDK: [![npm version](https://badge.fury.io/js/fk-platform-sdk.svg)](https://badge.fury.io/js/fk-platform-sdk)

## Integrate JS SDK
#### Add the dependency
If you are using `Node.js`, add the following repository as an NPM package for both WebView and React Native platforms. Visit [here](https://www.npmjs.com/package/fk-platform-sdk) to get the latest SDK for this repository.

```
npm install --save fk-platform-sdk
```

Or if you are using webview only, include the following script directly inside the `<script>` tag:

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

**Promise Resolution and Handling Denials:**

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


### Contacts
This module helps in launching a UI that allows user to select a contact from the user device’s address book and get its details.
> Contacts Module is available from Flipkart app v6.3 and Ultra v1.4.1 onwards.

**Get the contacts:**
```js
let contactModule = fkPlatform.getModuleHelper().getContactsModule()
```

**Picker:**
```js
pickPhoneNumber(): Promise<NativeModuleResponse<Contact>>
// It launches a UI for selecting one or more phone numbers and returns the same.
```

**Get Contact’s Name:**
```js
getContactInfo(phoneNumbers: string[]): Promise<NativeModuleResponse<{[key: string]: Contact;}>>
// It fetches names and other details against the list of phone numbers specified in the parameters. Here, “key(s)” of the returned map are the same phone numbers.
```

**Relevant Interfaces:**
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

### Analytics
Use this module for sending user generated actions to Ultra’s analytics library.

#### Add the dependency:
***With Node or for React Native:***

Add the [latest version of SDK](https://www.npmjs.com/package/fk-platform-sdk/v/1.0.6) to your `package.json` file:
```js
"fk-platform-sdk": "1.0.6"
```
Then, import the analytics module to the place where you want to push an event:
```js
import {<category_name>} from "fk-platform-sdk/analytics"; // for e.g <category_name> = Travel
```

***Without Node or Only WebView:***

Embed the following script inside the `<script>` tag in your code:
```js
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fk-analytics-travel-min@1.0.6.js
```

#### Create an Event object:
After importing the SDK, create an event object of any type (mentioned below) and for any specific category, for example, `Travel` or `Recharge`. Pass the arguments as described here for that event.

Let us suppose that your category is `Travel`.

***Search Event:***

Send this event from the screen where user searches for flights or buses or hotels.
```js
Search(fromLocation: string, toLocation: string[], date: Date[], roundTrip: boolean, travellerCount: number)
```
To create a Search event, pass the following arguments:

- **fromLocation:** It is of `string` type. For buses, `from Location` is the city name. In case of hotels, this represents location of the hotel. For flights, this is the international airport code.
- **toLocation:** A `string []` type. In case of buses or flights, `to Location` contains the destination city. For any intermediary locations, it stores different cities in the form of an array and in proper order. For hotels, this field will be empty.
- **date:** A `Date []` type. In case of a bus or a flight, this contains an array of dates for multi-city travel. For hotels, it means the start and end date of reservation.
- **roundTrip:** A `boolean` type. Set this to `True` for round trip travels.
- **travellerCount:** A `number` type that contains the data for the number of travellers. This must be greater than 0.

Here is the sample code:

***For WebView:***
```js
const search = FKExtension.analyticsHelper.travel.search("BLR", ["DEL"], [new  Date()], false, 1);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(search);
```

***For React Native:***
```js
const search = new Travel.Search("BLR", ["DEL"], [new Date()], false, 1);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(search);
```

***Select Event:***

Send this event when the user selects or clicks on any flight or bus or hotel from the list.
```js
Select(name: string, category: string, price: number)
```

To create a Select event, pass the following arguments:

- **name:** It is a `string`. This is the flight’s name in case of flights. A hotel’s name for hotels or name of a bus operator for buses.
- **category:** It is again a `string` type. This contains only one of the following values:
  - FLIGHT_INTERNATIONAL
  - FLIGHT_DOMESTIC
  - HOTEL_INTERNATIONAL
  - HOTEL_DOMESTIC
  - BUS
- **price:** It is a `number` type. The price of the selected product must be in Rupees and a non-negative value.

Here is the sample code:

***For WebView:***
```js
const select = FKExtension.analyticsHelper.travel.select("6E135","FLIGHT_DOMESTIC",3500);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(select);
```

***For React Native:***
```js
const select = new Travel.Select("6E135","FLIGHT_DOMESTIC",3500);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(select);
```

***ProceedToPay Event:***

Send this event after the user has completed the product selection and is proceeding towards the payment screen.
```js
ProceedToPay(amountToPay: number, offerAmount: number, bookingCharge: number, vasCharge: number)
```

To create a Proceed to Pay event, pass the below arguments. Note that all these values are in Rupees and accept decimal format:

- **amountToPay:** It is a `number`. This is a non-negative value and depicts the cost of the product (in Rupees) for `Travel` category.
- **offerAmount (Optional):** A `number` type. This represents the offer amount given to the customer. It is a non-negative value.
- **bookingCharge (Optional):** A `number` which contains the amount charged as convenience fees to the customer. This value is non-negative.
- **vasCharge (Optional):** It is a `number` type and a non-negative value. This shows the total amount of value added service charged to the customer.

Here is the sample code:

***For WebView:***
```js
const pay = FKExtension.analyticsHelper.travel.proceedToPayEvent(3500, 0, 350, 0);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(pay);
```

***For React Native:***
```js
const pay = new Travel.ProceedToPay(3500, 0, 350, 0);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(pay);
```

#### Sending the event:
Now, push any of the above-mentioned events as an argument.
```js
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(event);
```

!!!note
    If the event is invalid, the console displays an ‘error’ message describing the issue. It includes the list of parameters that are invalid.
    *Sample error log message :*
    ```
    Invalid event, ensure you resolve following issues: fromLocation is empty.
    ```

### Others
#### Handling Errors:
Each promise rejection carries an error code and a message.

**Relevant type:**
```js
interface NativeModuleError {
    message: string;
    errorCode: number;
}
```

Here are the common error codes in case of promise rejection:
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

#### Detecting Flipkart environment:
There are three ways by which you can detect whether your webpage is running within Flipkart’s Ultra container:
- Use JavaScript SDK's method: `FKPlatform.isPlatformAvailable()`. This is the most [PREFERRED] way.
- Use the `User Agent string` which contains `Ultra`. This method is useful only if you want to inject the JavaScript SDK while running within Flipkart's Ultra container. Here is an example of `user agent string`: `Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]`.
  !!!note
      If you have service workers making requests, this user agent will not be present. This is for [WEBVIEW-ONLY].
- Use X-Requested-With header that contains the value `com.flipkart.android`. This is also for [WEBVIEW-ONLY] but it works for all requests including service workers. This is the least preferred method.

#### Detect presence of a certain feature:
Few APIs are available only on a certain version of the Flipkart app. Each API that we mention here carries a note specifying the version of the Flipkart app it supports. To ensure backward compatibility, parse the User Agent string to figure out the correct app version support. The user agent string as shown below is available in both WebViews and React Native. 

For WebViews, access the User Agent via `navigator.userAgent` and for React Native, use `FKPlatform.getUserAgent()`.

User agent string in Ultra is of the following format:
```
Mozilla/5.0 (Linux; Android 8.0.0; Android SDK built for x86 Build/OSR1.170901.008; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/58.0.3029.125 Mobile Safari/537.36 [Flipkart/com.flipkart.android/850000/5.16/UltraSDK/4/1.4.2]
```

Here, in the last part of User Agent string, `5.16` is the `version name` and `850000` is the `version code` for `Flipkart app`. `1.4.2` is the `version name` and `4` is the `version code` for `Ultra SDK`.

If you want to check that any added modules are present or not, just look the Ultra’s version name i.e. 1.4.2.

#### Handling File uploads and downloads on WebView:
The file uploads and downloads feature is available from `Flipkart app v6.3` and `Ultra SDK v1.3.7` onwards. 

In WebView, make sure you do not use `download` attribute in the `href` for downloads. For example, `<a href='abcd.pdf' download='xyz.pdf' />` does not work. Even the APIs like `FileSaver.js` or Blob based APIs do not work because of the limitations in WebView. However, there are no restrictions for uploads.

#### Debugging:

**On Webview:**

We have enabled debugging of the APK (shared by us) within the Chrome browser. It means, you can connect to the Chrome instance via remote debugging. [Check here](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/) for details. If you encounter any API call failure or any images or page not loading issues, debug Ultra within the Chrome debugger.

**On React Native:**

The special APK (shared by us) can run your React Native code within Ultra. It points to the IP address of your React Native packager. This way, you need no special tools for development and debugging. This entire process runs the React native bundle without using the [Flipkart DUS](https://github.com/Flipkart/DUS).
