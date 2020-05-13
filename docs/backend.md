# Ultra Backend APIs

##Endpoint Contracts

Prod endpoint: `https://platform.flipkart.net`
> There is no staging endpoint for Ultra. You can do all development on Prod endpoint.

##Security
All production endpoints will be over https.
We will validate Client’s identity using JWT.
The JWT will be shared in every call in header as a ‘secureToken’
Properties of secure token
### Header
```
{
  "alg": "RS256",
  "typ": "JWT"
}
```
### Payload
'Note:' clientId should be fetched from flipkart.
```
{
  "iss": <clientId>,
  "iat": <unix timestamp in seconds>, // Issued at. Should be in past.
  "exp": <unix timestamp in seconds> // Expiry of token. Should be in future. Also exp - iat should not be > 1000.
}
```
The Public key should be shared with flipkart before hand.
```
Path: /1/dummy
Method: GET
Header: secureToken
Response:  Will be HTTP 200 if secureToken is valid otherwise response code will be self explanatory.
```

##Access token flow
Using the grantToken from the SDK, you have to fetch the accessToken before querying for any resources.
Get Auth token flow
```
Path: /1/authorization/auth

Method: GET

Query parameters:
grantToken : String
clientId : String
clientSecret : String

Response :
AuthTokenResponse {
identityToken (string),
accessToken (string)
}
```

##Resource fetching flow
Resources like user.mobile, user.email can be fetched with this API
fetch bulk data
```
Path: /1/resource/bulk

Method: POST
Query parameters:
accessToken : String
Body: List<String> //where each string is scope for which data is to be fetched.
Response : Map<String,Object> // String the scope and Object for each scope is defined below
user.mobile{
email(string)
isVerified(boolean)
}
user.email{
mobileNumber(string)
isVerified(boolean)
}
```

##OMS
OMS stands for [Order Management System](https://en.wikipedia.org/wiki/Order_management_system). Its the system which handles post order flows and its primary job is to power the `My Orders` screen on the flipkart apps.
When your system knows that an order has got placed or updated, you can call the following API to let Flipkart OMS know about the change.
This information will be rendered on the user's `My orders` screen with all the details you provide. We will also use this information for powering our CX agents for better experience.
For a better view of OMS, to understand its purpose and to understand the meaning of every field being uploaded refer [this page](oms.md) too
###OMS Upsert
```
Path: /3/oms
Method: POST
HEADER
secureToken : {secure_token} (JWT TOKEN)
Content-Type : application/json
Body:
Order {
  orderId : <String>
  orderState : <String> (ORDER_CONFIRMED , ORDER_FULFILLED, ORDER_CANCELLED)
  identityToken : <String>
  description : <String> (Optional)
  orderTimestamp(Order Creation Timestamp) : <Long>
  orderUpdatedTimestamp(Order Update time) : <Long>
  forwardTransaction(Amount paid by customer to Partner) : <Double>
  reverseTransaction(Amount Paid by Partner to customer) : <Double>
  lastFulfillmentDate: <Long>
  orderMetadata : <Map<String, String>> (Optional)
}
```
```
Sample Request
{
  "orderId": "ZO-B72BC17DD1",
  "orderState": "ORDER_CONFIRMED",
  "identityToken": "IDTKN848F9123941444B990045011A48D4920PEJ",
  "orderTimestamp": 1588686476,
  "orderUpdatedTimestamp": 1588686476,
  "forwardTransaction": 0.98,
  "reverseTransaction": 0,
  "lastFulfillmentDate": 1591286671
  "orderMetadata" : null // skip this field if not required
}
```

