##Purpose
OMS V2 is a view layer maintained by flipkart for orders placed on ultra platform. The OMS lite layer will act as a data source for flipkart’s my order page as well as CX agents.

OMS V2 has a check for idempotency meaning that any Order state that is accepted successfully will not be accepted again

More information on how to make the API calls to OMS is [here](backend.md#oms)

POST https://ultra-lite.flipkart.net/ultra/3/oms  (Preprod)

POST https://ultra.flipkart.net/ultra/3/oms [Prod]

The following diagram should explain the high level purpose of OMS lite.
![OMS HLD](img/oms_hld.png)

##Merchant Expectation
Merchant is expected to push for every state change in their OMS.

##Schema
###Order
|name|type|mandatory|notes|
|---|---|---|---|
|orderId|string|yes|orderId is expected to be reference to merchant's OMS entry|
|description|string|yes|Description of the order placed|
|identityToken|string|yes|This identifies the user,This will be sent to the partner during the SSO integration .Merchant
needs to pass it back|
|orderTimestamp|long|yes|Timestamp when the order was placed in the partner system|
|orderUpdatedTimestamp|long|yes|TImestamp for updation of order in
partner system|
|orderstate|string|yes|(ORDER_CONFIRMED , ORDER_FULFILLED, ORDER_CANCELLED).|
|forwardTransactions|double|yes|This indicates the order value paid by the customer to the merchant and based on which coins and commission are to be calculated by Ultra system. This is mandatory for all Order states and should always have value greater than 0|
|reverseTransactions|double|yes|Populate only if money has been reversed to customer.This field should not have value greater than forward transaction and is only expected in states ORDER_CANCELLED and ORDER_FULFILLED|
|orderMetadata|<Map<String, String>>|Optional|This field is only used to get additional information about the order or customer that might be needed for commission calculation or coin distribution like category, subcategory etc. It is an optional field|


###ORDER STATE
name|description|
|---|---|
|ORDER_CONFIRMED|his state indicates that order has been placed with the partner system and Pending Coins should be processed for customers. This is a mandatory state for OMS flow and should always be sent first to start OMS processing.|
|ORDER_CANCELLED|As evident from the name of the state, this state indicates that order has been cancelled with partner system and Coins should be redacted for customers. This state is mandatory to reflect order cancellation and commission redaction.|
|ORDER_FULFILLED|This state indicates that order has been Fulfilled (Delivered + Return/Replacement Period Finished) to customers, and there can be no change in order- modification / cancellation / return. Earned coins should be processed for customers and commission should be charged to merchants.This state is mandatory to reflect order completion and commission invoicing|

###Custom Error with Http Status Code
Http_Status_Code|Ultra_Status_Code|Error_Message|format|
|---|---|---|---|
|400|10|Invalid public key for merchant id|TBA|
400|11|JWT does not contain three sections|TBA|
400|12|JWT does not contain clientId|TBA|
400|13|JWT does not contain clientId|TBA|
400|14|JWT does not contain sub|TBA|
400|15|Account info not found for identity token|TBA|
429|31|Rate Limit reached for client|TBA|
400|21|Forward Transaction should be greater than 0|TBA|
400|22|Order State can be either ORDER_CONFIRMED ORDER_CANCELLED or ORDER_FULFILLED|{"timestamp":"2020-02-18T11:32:07.342+0000","message":"Order State can be either ORDER_CONFIRMED ORDER_CANCELLED or ORDER_FULFILLED","status":22,"error":"BadInputException","additionalInfo":{"orderState":"ORDER_INIT"}}|
400|23|Order State already exist|{"timestamp":"2020-02-18T11:32:07.342+0000","message":"Order state already exist","status":23,"error":"BadInputException","additionalInfo":{"orderState":"ORDER_FULFILLED"}}|
400|24|Order Event is invalid as unexpected order state is received|{"timestamp":"2020-02-18T11:32:07.342+0000","message":"Order Event is invalid as unexpected order state is received","status":24,"error":"BadInputException","additionalInfo":{"orderState":"ORDER_FULFILLED"}}|
400|25|Reverse Transaction should be 0|TBA|
400|26|Reverse Transaction is not supported for ORDER_CONFIRMED state and should be less than equal to forward transaction|TBA|

###Reconciliation Report UPLOAD

Reconciliation report is used to validate the oms events sent by the partner and is generated at (T+1)<sup>th</sup> 
day in the below-mentioned format.


- *Report At (T+1)<sup>th</sup>* : Will contain all the orders updated or created on T<sup>th</sup> day with their 
latest state plus the replayed orders.
- *File Format* : Allowed format is csv.
- *Max File Size* : 20MB

Reconciliation report example:
```
Order Id,Order States,Order Created Timestamp,Order Updated Timestamp,Total Forward Transaction,Total Reverse Transaction,Total Cancellation Charges
ORDER_ID_1,ORDER_CONFIRMED,1544424417569,1556802866753,760000,750000,10000
ORDER_ID_2,ORDER_FULFILLED,1544424417569,1556802866753,850000,0,0
```

Curl Request: 
```
curl --request POST \
--url https://platform.flipkart.net/2/oms/recon/report/upload \
--header 'content-type: multipart/form-data; boundary=---011000010111000001101001' \
--header 'securetoken: [secureToken]' \
--form file=[file]
```

###Reconciliation Report DOWNLOAD

Once we receive a recon report from a partner, we process and generate Reconciled Report and send it via email back to the partner. The Reconciled Report has the difference between the orders in Flipkart OMS and the partners OMS. Once the partner receives the document, they download it and replay the mismatched orders. All the effort here is manual and we would like to help to automate this entire process. Hence as part of it, we are creating an API where partners can run a cron job and pull the Reconciled Report through an API and all mismatches can be reconciled by replaying the events. 


Description of the API: 
```
GET https://platform.flipkart.net/2/oms/recon/report/download
HEADER 
secureToken : JWT_TOKEN

Response: 
Status : 200
Body: Success
Attachment: CSV-file

Failure for invalid secure token:
Status : 401 Unauthorized
Body : 	Unauthorized
```
