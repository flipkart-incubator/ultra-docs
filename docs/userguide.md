##Integration steps
###Step 1
Add the Ultra Javascript SDK to your HTML/React-native app. More details [here](clients.md#step-1).

Initialize the SDK with your clientId. More details [here](clients.md#step-2).

Contact flipkart to generate a `clientId` and `secret` needs to access Ultra APIs. 

###Step 2
Call `getToken` on the client SDK to get a grant token. This will render the Allow/Deny permission prompt to user. More details [here](clients.md#permissions-module).

###Step 3
Send the token to you server using a AJAX call or any other mechanism. This token can be used by the server to get an access token. 


###Step 4
Use this grant token and fetch the access token by making a call to `/1/authorization/auth`. More details [here](backend.md#access-token-flow).
Use the access token and fetch the resources like `user.email` and `user.mobile`. More details [here](backend.md#resource-fetching-flow).

Note : Although the APIs to fetch user data is available on client side also, make sure you always get user data on server side and not on the client side to avoid security risks like MITM attacks.

###Step 5
Use the value of `mobileNumber` in combination with `isVerified` of `user.mobile` to automatically log the user in. You could now set a cookie which prevent call to `getToken` from happening each time. This completes the Login flow.

###Step 6
Follow steps in [payment flow](backend.md#payment-flow) to initiate payments and once successful redirect to your own order confirmation page. Also integrate with [refund API](backend.md#refund) as per your business requirements.

###Step 7
Integrate with Flipkart OMS APIs. More info [here](backend.md#oms).

###Step 8
Checkout the [demo page](demo.md) to see this flow in action.