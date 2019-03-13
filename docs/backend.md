# Ultra Backend APIs

## Endpoint Contracts
There is no Staging endpoint for Ultra. Thus, you can work directly on the Production endpoint.
> The production endpoint is : `https://platform.flipkart.net`

## Security
All production endpoints are over `https://`. We validate an Ultra partner’s identity using `JWT`. Every call in header part of the secure token shares the `JWT` as a property called `secureToken`.

### Header
Header section comprises:
```
{
  "alg": "RS256",
  "typ": "JWT"
}
```

### Payload
Payload comprises:
```
{
  "iss": <clientId>, // Shared by Flipkart team
  "iat": <unix timestamp in seconds>, // Issued at. Should be in past.
  "exp": <unix timestamp in seconds> // Expiry of token. Should be in future. Also exp - iat should not be > 100.
}
```
***Sample:***
```
Path: /1/dummy

Method: GET

Header: secureToken

Response:  HTTP 200 (if secureToken is valid) otherwise this code will be self explanatory.
```

!!!note
    Please share the `public key` generated at your side with us in advance.

## Flows
### Access Token Generation
Fetch the `accessToken` before querying for any of the user’s resources (i.e. email, mobile number) using the `grantToken` from the SDK.

**GET Auth Token:**
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

**Sample Request:**
```
// To be included
```
**Sample Response:**
```
// To be included
```

### Resource Fetching
This API helps in fetching data of the user’s resources in bulk. Resources may be in the form: `user.mobile`, `user.email`.

**Fetch Bulk data:**
```
Path: /1/resource/bulk

Method: POST
Query parameters:
accessToken : String

Body: List<String> // where each string is scope for which data is to be fetched.

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

**Sample Request:**
```
// To be included
```
**Sample Response:**
```
// To be included
```

In the next two sections, we are going to share the details of the FKPG (Flipkart Payment Gateway) APIs used to start payments and receive refunds. [Contact Flipkart](contact.md) for the merchant credentials required to access these APIs.

!!!note
    Please note that merchant credentials differ from `clientId` and `secret`. These are specific to Ultra’s APIs.

## OMS
### Schema
***Order:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|orderId|string|yes|`orderId` is a reference to the merchant's OMS entry|
|description|string|yes||
|identityToken|string|yes|This identifies the user|
|orderTimestamp|long|yes|The time when the order starts|
|orderUpdatedTimestamp|long|yes|The time when order was last modified|
|orderUrl|string|yes|The URL that will take us to this specific order|
|items|List of Item|yes|List of items which are a part of this order|
|forwardTransactions|List of ForwardTransaction|no|These can be empty only if all items are in init state|
|reverseTransactions|List of ReverseTransaction|no|Populate only if money has been reversed to customer|
|merchantAdjustments|List of MerchantAdjustment|no|All the discounts offered by the merchant. `PG (Payment Gateway)` receives the Base price of items plus Merchant adjustments for payment from the customer|
|flipkartAdjustments|List of FlipkartAdjustment|no|After the response from PG, Merchants will know about additional discounts applied by PG. It shows such adjustments|
|cancellationCharges|List of CancellationCharges|no|Merchant can levy charges for the cancellation|

***Item:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|itemId|string|yes|It is an item identifier. It draws the customer to a product from order page|
|title|string|yes|It is the description of an item. It is visible to the customer|
|image|string|yes|Item's image|
|basePrice|double|yes|Price of an item before applying any adjustments|
|finalPrice|double|yes|Price of an item after applying both `merchantAdjustments` and `flipkartAdjustments`. This is `null` if the state is `INIT`|
|category|string|yes|Category to which the item belongs. Flipkart provides this value. Please put no other values here|
|fulfillmentDate|long|yes|Date of fulfillment of the item (For e.g., delivery date, travel date, etc.)|
|itemState|enum(`INIT` or `SUCCESSFUL` or `CANCELLED` or `PENDING`)|yes|`INIT`: Used when the customer has not paid. `PENDING`: Payment is successful, however, the merchant is yet to confirm the order. `SUCCESSFUL`: Confirmed the item but not completely. Partial cancellation is also a part of this state. `CANCELLED`: Cancelled the item. Here, the complete final price should either reflect in `Reverse Transaction` or `cancellationCharges`|
|brand|string|yes|Provider/Manufacturer of product|
|product|string|yes|Description of the product. Sometimes, it is the Title of any e-commerce product. Other times, it provides specific details like Flight number, Recharge type, etc.|
|customerName|string|yes|Name of the person who has ordered the item|
|quantity|integer|yes|Number of products clubbed in the same order. In case of a category as `travel`, it is the number of passengers. If you have no such business requirement, send `1`|

***Forward Transaction:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|transactionId|string|yes|PG shares the `transactionId`|
|amount|double|yes|Final amount deducted from customer|
|description|string|yes||
|timestamp|long|yes|Time of transaction|

***Reverse Transaction:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|forwardTransactionId|string|yes|The forward transaction used to take money from customer|
|reverseTransactionId|string|yes||
|amount|double|yes|Refund amount|
|description|string|yes||
|timestamp|long|yes|Time of transaction|

***Merchant Adjustment:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|adjustmentId|string|yes|Identifier for the adjustment on the merchant’s side|
|amount|double|yes|Adjustment amount|
|title|string|yes|Adjustment description. This could be visible to the customer|

***Flipkart Adjustment:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|adjustmentId|string|yes|Identifier for the adjustment provided by Flipkart in PG response|
|amount|double|yes|Adjustment amount|

***Cancellation Charges:***

|name|type|mandatory|notes|
| --- | :---: | :---: | --- |
|itemId|string|yes|Represents an item’s identity which must be available in the list of existing items|
|reason|string|yes|This is visible to the customer|
|amount|double|yes|Cancellation amount|

### OMS Upsert
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

**Sample Request:**
```
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
**Sample Response:**
```
// To be included
```

## Offers
This API lets you interact with the offers from Flipkart.

### Get the list of offers
```
Path: /2/offers/active
Method: GET
```
