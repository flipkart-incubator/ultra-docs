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
  "exp": <unix timestamp in seconds> // Expiry of token. Should be in future. Also exp - iat should not be > 100.
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
Path: /2/oms
Method: POST
Body:
Order {
orderId (string),
description (string),
identityToken (string),
orderTimestamp (long),
orderUpdatedTimestamp(long),
orderUrl (string),
items (Array[Item]),
forwardTransactions (Array[ForwardTransaction], optional),
reverseTransactions (Array[ReverseTransaction], optional),
merchantAdjustments (Array[MerchantAdjustment], optional),
flipkartAdjustments (Array[FlipkartAdjustment], optional),
cancellationCharges (Array[CancellationCharges], optional)
}
Item {
itemId (string),
title (string),
image (string),
basePrice (double),
finalPrice (double, optional),
category (string),
fulfillmentDate (long),
itemState (string) = ['INIT' or 'SUCCESSFUL' or 'CANCELLED' or 'PENDING'],
brand (string),
product (string),
customerName (string),
quantity (string)
}
ForwardTransaction {
transactionId (string),
amount (double),
description (string),
timestamp (long)
}
ReverseTransaction {
forwardTransactionId (string),
reverseTransactionId (string),
amount (double),
description (string),
timestamp (long)
}
MerchantAdjustment {
adjustmentId (string),
amount (double),
title (string)
}
FlipkartAdjustment {
adjustmentId (string),
amount (double)
}
CancellationCharges {
itemId (string),
reason (string),
amount (double)
}
```
```
Sample Request
{
  "orderId": "DummyOrderId",
  "description": "This is a dummy description",
  "identityToken": "someIdentityToken",
  "orderTimestamp": 1530622713945,
  "orderUpdatedTimestamp": 1530622713945,
  "orderUrl": "someURLToOrderPage",
  "items": [
    {
      "itemId": "Product 1",
      "title": "This is a product",
      "image": "image.url",
      "basePrice": 120,
      "finalPrice": 100,
      "category": "test",
      "fulfillmentDate": 1530622713946,
      "itemState": "SUCCESSFUL",
      "brand": "Some brand",
      "product": "modelNumber",
      "customerName": "Lorem Ipsum",
      "quantity": 1
    },
    {
      "itemId": "Product 2",
      "title": "This is a product",
      "image": "image.url",
      "basePrice": 120,
      "finalPrice": 100,
      "category": "test",
      "fulfillmentDate": 1530622713946,
      "itemState": "SUCCESSFUL",
      "brand": "Some brand",
      "product": "modelNumber",
      "customerName": "Lorem Ipsum 2",
      "quantity": 1
    }
  ],
  "forwardTransactions": [
    {
      "transactionId": "transaction1",
      "amount": 100,
      "description": "Paid via FKPG",
      "timestamp": 1530622713956
    }
  ],
  "reverseTransactions": [
    {
      "forwardTransactionId": "transaction1",
      "reverseTransactionId": "rev_transaction1",
      "amount": 10,
      "description": "Refund for cancellation",
      "timestamp": 1530622714957
    }
  ],
  "merchantAdjustments": [
    {
      "adjustmentId": "Dummy merchant adjustment id",
      "title": "This is some title",
      "amount": 20
    }
  ],
  "flipkartAdjustments": [
    {
      "adjustmentId": "dummyAdjustmentId",
      "amount": 20
    }
  ],
  "cancellationCharges": [
    {
      "itemId": "Product 1",
      "reason": "Cancellation costs are sometimes deducted",
      "amount": 10
    }
  ]
}
```

