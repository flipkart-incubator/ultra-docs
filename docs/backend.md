# Ultra Backend APIs

##Endpoint Contracts

Prod endpoint: `https://platform.flipkart.net`

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

##Payment flow.

This section describes FKPG (Flipkart payment gateway) APIs which can be used to initiate payments and for receiving refunds.
Contact Flipkart team to get the merchant credentials required to access this API. The merchant credentials are different from `clientId` and `secret` used for Ultra APIs

###Payment Token
This is the first step in initiating payments. Use this API to generate a payment token which can be used to show the FKPG payment options screen, this screen can be shown via Ultra's client SDK. This will open FKPG outside of Ultra container.

```
Path: 1/payment/token

Method: POST
Body: PaymentTokenRequest 
PaymentTokenRequest {
merchantCredential(MerchantCredential),
amountPaise (long),
paymentExpiryMilliSeconds (long, optional),
userInfo (UserInfo),
adjustmentWrapper(AdjustmentWrapper, optional),
priceSummary (PriceSummary),
successfulCallBackUrl (string),
failureCallBackUrl (string),
address (Address, optional),
description (string, optional),
metadata (string, optional),
merchantTransactionId (string),
merchantReferenceId (string)
}
MerchantCredential {
name (string),
password (string)
}
UserInfo {
name (string, optional),
phone (string, optional),
email (string, optional),
identityToken (string)
}
AdjustmentWrapper {
eligibleAdjustments(Array[EligibleAdjustment], optional)
}
EligibleAdjustment {
adjustment_id (string, optional),
offerUnitPrice (long, optional),
metadata (object, optional)
}
PriceSummary {
basePricePaise (long),
itemCount (integer),
breakup(Array[PriceSummaryBreakup])
}
PriceSummaryBreakup {
description (string),
displayText (string),
valueInPaise (long),
breakupType (string) = ['DEFAULT' or 'DISCOUNT']
}
Address {
addressLine (string, optional),
city (string, optional),
state (string, optional),
pincode (string, optional),
country (string, optional)
}
Response : PaymentToken
PaymentToken {
token (string)
}
```
```
Sample Request
{
  "merchantCredential": {
    "name": "Enter name here",
    "password": "Enter hash here"
  },
  "merchantTransactionId": "transaction1",
  "merchantReferenceId": "order1",
  "amountPaise": 200,
  "paymentExpiryMilliSeconds": 100000,
  "userInfo": {
    "identityToken": "Actual ID token here"
  },
  "successfulCallBackUrl": "http://www.partner.com/success",
  "failureCallBackUrl": "http://www.partner.com/failure",
  "description": "this is a test transaction",
  "metadata": "this is a test transaction",
  "priceSummary": {
    "basePricePaise": 100,
    "itemCount": 0,
    "breakup": [
      {
        "description": "convinience_fee",
        "displayText": "Convinence Fee",
        "valueInPaise": 100,
        "breakupType": "DEFAULT"
      }
    ]
  }
}
```

###Callback after PGResponse
Once a user pays, a `POST` call is made to the `successfulCallBackUrl` specified in payment token request. The payload of the FORM data is present below.
You can then issue a HTTP redirect to the `fapp:` URL to redirect back to ultra container and also specify the order confirmation page url to open in the same format.
This will open your order confirmation page within ultra container (payment happens outside of it). Its mandatory to redirect to ultra container via `fapp:`. Do not open any UI outside of ultra container.
```
Request (will be sent as form parameters)

{
      "transaction_status": "",	//	SUCCESS, FAILED
      "account_type": "NODAL",
      "pg_trackid": "",		
      "merchant_adjustments": 
      "[
      {"offer_id":"",
      "offer_unit_price":0,
      "amount_applied":0,
      "amount_requested":0,
      "actual_subvention_amount":0,
      "effective_subvention_amount":0,
      "Adjustment_response_code": AdjustmentResponseCode,	//	Appendix
      "metadata":{"paymentSystem":"", "offerId":"", "discountType":""}}
      ]",
      "transaction_amount": "",
      "emi_months": "",
      "merchant_id": "",
      "transaction_response_code": "",		//	Appendix
      "payzippy_transaction_id": "",
      "having_multiple_transactions": "",
      "bank_name": "",
      "card_brand": CardName,	//	Appendix
      "hash_method": "",
      "transaction_time": "",
      "transaction_currency": "",
      "payment_method": "",		//	Appendix
      "timestamp": "",
      "merchant_key_id": "",
      "primary_record": 
      "{"transaction_id": "", "primary_amount": ""}",
      "merchant_transaction_id": "",
      "bank_transaction_id": "",
      "payment_instrument": "",
      "transaction_response_message": "",
      "pg_mid": "",
      "pg_name": "",
      "pg_authcode": "",
      "pg_id": "",
      "is_international": "",
      "fraud_action": "",
      "is_risky_instrument": "",
      "transaction_auth_state": "",
      "hash": "",
      "masked_card_number": "",
"card_bin": "",
}

```
```
Form Data

transaction_status=""&account_type=""&pg_trackid=""&merchant_adjustments=[ {'offer_id':'', 'offer_unit_price':0, 'amount_applied':0, 'amount_requested':0, 'actual_subvention_amount':0, 'effective_subvention_amount':0, ent_response_code':'', 'metadata':{'paymentSystem':'', 'offerId':'', 'discountType':''}} ]transaction_amount=""&emi_months=""&merchant_id=""&transaction_response_code=""&payzippy_transaction_id=""&having_multiple_transactions=""&bank_name=""&card_brand=""&hash_method=""&transaction_time=""&transaction_currency=""&payment_method=""&timestamp=""&merchant_key_id=""&primary_record={'transaction_id': '', 'primary_amount': ''}merchant_transaction_id=""&bank_transaction_id=""&payment_instrument=""&transaction_response_message=""&pg_mid=""&pg_name=""&pg_authcode=""&pg_id=""&is_international=""&fraud_action=""&is_risky_instrument=""&transaction_auth_state=""&hash=""&masked_card_number=""&transaction_status=""&account_type=""&pg_trackid=""&merchant_adjustments=""&transaction_amount=""&emi_months=""&merchant_id=""&transaction_response_code=""&payzippy_transaction_id=""&having_multiple_transactions=""&bank_name=""&card_brand=""&hash_method=""&transaction_time=""&transaction_currency=""&payment_method=""&timestamp=""&merchant_key_id=""&primary_record=""&merchant_transaction_id=""&bank_transaction_id=""&payment_instrument=""&transaction_response_message=""&pg_mid=""&pg_name=""&pg_authcode=""&pg_id=""&is_international=""&fraud_action=""&is_risky_instrument=""&transaction_auth_state=""&hash=""&masked_card_number=""
```


###Query
```
Path: 1/payment/query

Method: POST
Body: QueryRequest 
QueryRequest {
merchantCredential(MerchantCredential),
forcePgQuery (boolean, optional),
merchantTransactionId (string),
paymentTransactionId (string),
transactionType (string, optional) = ['ADJUSTMENT' or 'SALE' or 'REFUND' or 'REVADJUSTMENT']
}
MerchantCredential {
name (string),
password (string)
}
Response : QueryResponse
 QueryResponse {
responseStatus (string, optional) = ['SUCCESS' or 'FAILED'],
responseType (string, optional),
responseMessage (string, optional),
messages (Array[QueryMessage], optional),
merchantId (string, optional),
merchantTransactionId (string, optional),
merchantReferenceId (string, optional),
transactionState (string, optional) = ['INITIATED' or 'PENDING' or 'SUCCESS' or 'FAILED'],
paymentTransactions (Array[PaymentTransaction], optional)
}

QueryMessage {
type (string, optional),
message (string, optional)
}
PaymentTransaction {
merchantTransactionId (string, optional),
paymentTransactionId (string, optional),
transactionAmount (long, optional),
transactionCurrency (string, optional),
transactionType (string, optional) = ['ADJUSTMENT' or 'SALE' or 'REFUND' or 'REVADJUSTMENT'],
transactionTime (string, optional),
realTransactionTime (string, optional),
transactionStatus (string, optional) = ['INITIATED' or 'PENDING' or 'SUCCESS' or 'FAILED'],
transactionAuthState (string, optional) = ['PRE_AUTH' or 'PARTIAL_CAPTURED' or 'FULL_CAPTURED' or 'CAPTURE_COMPLETED' or 'VOID' or 'FAILED' or 'SALE'],
transactionResponseCode (string, optional) = ['REFUND_REQUEST_ACCEPTED' or 'REFUND_REQUEST_SENT' or 'REFUNDED' or 'REFUND_VOID_SUCCESS' or 'TXNID_NOT_FOUND' or 'REFUND_NOT_SUPPORTED' or 'REFUND_WINDOW_EXPIRED' or 'SIMILAR_PREVIOUS_PARTIAL_REFUND_DETECTED' or 'SALE_TRANSACTION_UNSUCCESSFUL' or 'SALE_TRANSACTION_VOID' or 'PARTIAL_REFUNDS_UNSUPPORTED' or 'INSUFFICIENT_BALANCE' or 'DUPLICATE_REFUND_REQUEST' or 'EXCESS_REFUND_AMOUNT' or 'SALE_TRANSACTION_REFUNDED' or 'MULTIPLE_REFUNDS_UNSUPPORTED' or 'REFUND_TEMPORARILY_UNAVAILABLE' or 'VISA_DIRECT_TIMEOUT' or 'PENDING_ON_REFUND_REQUERY' or 'FAILED_ON_REFUND_REQUERY' or 'COULD_NOT_ACQUIRE_LOCK' or '_3DS_AUTH_FAILED' or '_3DS_AUTH_UNSUPPORTED' or 'ADDRESS_VERIFICATION_FAILED' or 'BANK_RESPONSE_DELAYED' or 'BIN_BLOCKED_BY_ACQUIRER' or 'CARD_EXPIRED' or 'CARD_NOT_ENROLLED' or 'CARD_NUMBER_INVALID' or 'COUNTRY_NOT_SUPPORTED' or 'CVV_INCORRECT' or 'CVV_MISSING' or 'SBI_DEBIT_CARD_BLOCKED' or 'PIN_INCORRECT' or 'CANCELLED_BY_ACQUIRER' or 'DECLINED_BY_ACQUIRER' or 'DECLINED_BY_ISSUER' or 'DECLINED_BY_RISK' or 'INSUFFICIENT_FUNDS' or 'ISSUER_TECHNICAL_ERROR' or 'EMI_NO_PG_FOR_CURRENT_AMOUNT' or 'NO_PG_FOR_CURRENT_AMOUNT' or 'UPI_EXTERNAL_VPA_UNSUPPORTED' or 'MID_NOT_ACTIVE' or 'MID_NOT_FOUND' or 'DUPLICATE_TXN_REQUEST' or 'INVALID_TRANSACTION_ID' or 'MERCHANT_AUTH_FAILED' or 'DECLINED_BY_PAYZIPPY' or 'BANK_UNAVAILABLE' or 'CARD_EXPIRY_DATE_INVALID' or 'CANCELLED_BY_USER' or 'USER_SESSION_TIMED_OUT' or 'USER_REFRESH_COUNT_EXCEEDED' or 'USER_RETRY_COUNT_EXCEEDED' or 'INVALID_PROCESSPAY_REQUEST' or 'MANDATORY_PARAM_MISSING' or 'INVALID_PARAM_FORMAT' or 'INVALID_PARAM_VALUE' or 'INVALID_PAYZIPPY_ACCOUNT' or 'NETBANKING_NOT_ENABLED' or 'NETBANKING_LIMIT_EXCEEDED' or 'ACCOUNT_ON_HOLD' or 'PAY_LATER_ELIGIBILITY_FAILURE' or 'PAYZIPPY_TECHNICAL_ERROR' or 'ACQUIRER_TECHNICAL_ERROR' or 'SUCCESS' or 'INITIATED' or 'PENDING' or 'REQUEST_FIELD_INVALID' or 'SYSTEM_UNDER_MAINTENANCE' or 'EMPTY_OTP_ERROR' or 'INTERNAL_SERVER_ERROR' or 'DATABASE_SERVER_ERROR' or 'PROTOCOL_ERROR' or 'NOT_PERMITTED' or 'CARD_NOT_IN_3DS_RANGE' or 'INVALID_AUTH_DATA' or 'INVALID_REQUEST_TYPE' or 'INVALID_REQUEST_XML_FORMAT' or 'HASH_CHECK_FAILED' or 'INVALID_MERCHANT_ID' or 'ITP_VALIDATION_FAILED' or 'INVALID_IVR_OTP_DATA' or 'TIMED_OUT' or 'UNABLE_TO_PROCESS_REQUEST' or 'INVALID_RESPONSE_XML_FORMAT' or 'CARD_BLOCKS' or 'OPEN_ECS_NACH_MANDATE' or 'DDF_DEALER' or 'CARD_OFFER_EXHAUSTED' or 'CHECK_SUM_MISMATCH'],
refundResponseCode (string, optional) = ['SUCCESS' or 'REFUND_REQUEST_ACCEPTED' or 'REFUND_REQUEST_SENT' or 'REFUNDED' or 'REFUND_VOID_SUCCESS' or 'TXNID_NOT_FOUND' or 'REFUND_NOT_SUPPORTED' or 'REFUND_WINDOW_EXPIRED' or 'SIMILAR_PREVIOUS_PARTIAL_REFUND_DETECTED' or 'SALE_TRANSACTION_UNSUCCESSFUL' or 'SALE_TRANSACTION_VOID' or 'PARTIAL_REFUNDS_UNSUPPORTED' or 'INSUFFICIENT_BALANCE' or 'DUPLICATE_REFUND_REQUEST' or 'EXCESS_REFUND_AMOUNT' or 'SALE_TRANSACTION_REFUNDED' or 'MULTIPLE_REFUNDS_UNSUPPORTED' or 'PAYZIPPY_TECHNICAL_ERROR' or 'ACQUIRER_TECHNICAL_ERROR' or 'REFUND_TEMPORARILY_UNAVAILABLE' or 'VISA_DIRECT_TIMEOUT' or 'PENDING_ON_REFUND_REQUERY' or 'FAILED_ON_REFUND_REQUERY' or 'COULD_NOT_ACQUIRE_LOCK' or 'UPI_EXTERNAL_VPA_UNSUPPORTED'],
transactionResponseMessage (string, optional),
bankArn (string, optional),
refundSla (RefundSla, optional),
paymentMethod (string, optional) = ['CREDIT' or 'DEBIT' or 'EMI' or 'NET' or 'PAYZIPPY' or 'NET_OPTIONS' or 'EMI_OPTIONS' or 'PHONEPE' or 'FLIPKART_CREDIT'],
paymentInstrument (string, optional) = ['CREDIT' or 'DEBIT' or 'EMI' or 'NET' or 'PAYZIPPY' or 'NET_OPTIONS' or 'EMI_OPTIONS' or 'PHONEPE' or 'FLIPKART_CREDIT'],
emiMonths (long, optional),
bankName (string, optional),
emiScheme (EmiScheme, optional),
fraudAction (string, optional),
fraudDecision (string, optional),
fraudDetails (string, optional),
fraudSource (string, optional),
pgMID (string, optional),
pgTrackId (string, optional),
pgId (string, optional),
pgName (string, optional),
bankTransactionId (string, optional),
terminalId (string, optional),
pgAuthCode (string, optional),
accountType (string, optional),
subventionPercentage (long, optional),
cardBrand (string, optional) = ['VISA' or 'MASTERCARD' or 'AMEX' or 'MAESTRO' or 'DINERS' or 'RUPAY' or 'DEFAULT' or 'DISCOVER' or 'BAJAJ'],
cardBin (string, optional),
orderId (string, optional),
relatedRecords (Array[RelatedRecord], optional),
primaryAmount (long, optional),
merchantAdjustments (Array[MerchantAdjustment], optional),
queryTransactionStatus (string, optional),
metadata (string, optional),
saleTransactionId (string, optional),
confirmedFraud (boolean, optional),
international (boolean, optional),
riskyInstrument (boolean, optional)
}
RefundSla {
minSla (long, optional),
maxSla (long, optional)
}
EmiScheme {
emiTenureInMonths (long, optional),
interestValue (long, optional),
interestType (string, optional)
}
RelatedRecord {
adjustmentTransactionId (string, optional),
relation (string, optional),
adjustmentId (string, optional),
adjustmentAmount (long, optional)
}
MerchantAdjustment {
transactionId (string, optional),
offerId (string, optional),
offerUnitPrice (long, optional),
amountApplied (long, optional),
amountRequested (long, optional),
promiseDate (string, optional),
actualSubventionAmount (long, optional),
effectiveSubventionAmount (long, optional),
adjustmentResponseCode (string, optional) = ['EXPIRED' or 'INVALID' or 'FAILED_BY_TIME' or 'REJECTED_BY_WHITELIST' or 'REJECTED_BY_BLACKLIST' or 'REJECTED_BY_RULE' or 'REJECTED_LOW_PRIORITY' or 'EXHAUSTED_BY_TXNID' or 'EXHAUSTED_BY_EMAIL' or 'EXHAUSTED_BY_ACCOUNT' or 'EXHAUSTED_BY_PHONE' or 'EXHAUSTED_BY_IP' or 'EXHAUSTED_BY_ACCID' or 'EXHAUSTED_BY_MID' or 'EXHAUSTED_GLOBAL' or 'EXHAUSTED_BY_CARD' or 'MISSING_PAYMENT_INFO' or 'ELIGIBLE' or 'NOT_ELIGIBLE' or 'OFFER_NOT_APPLIED' or 'REJECTED_DUE_TO_PAYMENT_WARNING'],
adjustmentType (string, optional) = ['INSTANT_DISCOUNT' or 'CASHBACK_ON_CARD' or 'CASHBACK_IN_BANK' or 'CASHBACK_IN_WALLET' or 'INSTANT_CASHBACK' or 'DOWN_PAYMENT'],
metadata (object, optional)
}
```
```
Sample Request
{
  "merchantCredential": {
    "name": "Enter name here",
    "password": "Enter hash here"
  },
  "forcePgQuery": true,
  "merchantTransactionId": "transaction1",
  "paymentTransactionId": "PZT1712211056FQM0400"
}
```

###Refund
```
Path: 1/payment/query

Method: POST
Body: RefundRequest 
RefundRequest {
merchantCredential(MerchantCredential),
merchantTransactionId (string),
payzippySaleTrasactionId (string),
refundAmount (long),
refundReason (string),
refundedBy (string),
merchantRefundTransactionId(string, optional),
idempotencyId (string, optional),
metadata (string, optional)
}
MerchantCredential {
name (string),
password (string)
}
Response : RefundResponse
RefundResponse {
bankArn (string, optional),
pgMID (string, optional),
pgTrackId (string, optional),
pgId (string, optional),
pgName (string, optional),
bankTransactionId (string, optional),
terminalId (string, optional),
cardBrand (string, optional) = ['VISA' or 'MASTERCARD' or 'AMEX' or 'MAESTRO' or 'DINERS' or 'RUPAY' or 'DEFAULT' or 'DISCOVER' or 'BAJAJ'],
cardBin (string, optional),
adjustmentTransactionIds (string, optional),
adjustmentRelations (string, optional),
adjustmentIds (string, optional),
adjustmentAmounts (string, optional),
refundAmount (long, optional),
merchantTransactionId (string, optional),
payzippyTransactionId (string, optional),
refundCurrency (string, optional),
transactionTime (long, optional),
refundSla (RefundSla, optional),
refundStatus (string, optional) = ['REFUND_REQUEST_RECEIVED' or 'REFUND_REQUEST_SUBMITTED' or 'SUCCESS' or 'FAILED' or 'REFUNDED' or 'PENDING'],
refundResponseCode (string, optional) = ['SUCCESS' or 'REFUND_REQUEST_ACCEPTED' or 'REFUND_REQUEST_SENT' or 'REFUNDED' or 'REFUND_VOID_SUCCESS' or 'TXNID_NOT_FOUND' or 'REFUND_NOT_SUPPORTED' or 'REFUND_WINDOW_EXPIRED' or 'SIMILAR_PREVIOUS_PARTIAL_REFUND_DETECTED' or 'SALE_TRANSACTION_UNSUCCESSFUL' or 'SALE_TRANSACTION_VOID' or 'PARTIAL_REFUNDS_UNSUPPORTED' or 'INSUFFICIENT_BALANCE' or 'DUPLICATE_REFUND_REQUEST' or 'EXCESS_REFUND_AMOUNT' or 'SALE_TRANSACTION_REFUNDED' or 'MULTIPLE_REFUNDS_UNSUPPORTED' or 'PAYZIPPY_TECHNICAL_ERROR' or 'ACQUIRER_TECHNICAL_ERROR' or 'REFUND_TEMPORARILY_UNAVAILABLE' or 'VISA_DIRECT_TIMEOUT' or 'PENDING_ON_REFUND_REQUERY' or 'FAILED_ON_REFUND_REQUERY' or 'COULD_NOT_ACQUIRE_LOCK' or 'UPI_EXTERNAL_VPA_UNSUPPORTED'],
refundResponseMessage (string, optional),
metaData (string, optional)
}
RefundSla {
minSla (long, optional),
maxSla (long, optional)
}
```
```
Sample Request
{
  "merchantCredential": {
    "name": "Enter name here",
    "password": "Enter hash here"
  },
  "merchantTransactionId": "transaction1",
  "payzippySaleTrasactionId": "PZT1712211056FQM0400",
  "refundAmount": 100,
  "refundReason": "testing",
  "refundedBy": "Anvay",
  "merchantRefundTransactionId": "transaction1-refund1"
}
```




##OMS
OMS stands for [Order Management System](https://en.wikipedia.org/wiki/Order_management_system). Its the system which handles post order flows and its primary job is to power the `My Orders` screen on the flipkart apps.
When your system knows that an order has got placed or updated, you can call the following APIs to let Flipkart OMS know about the change.
This information will be rendered on the user's `My orders` screen with all the details you provide.

###OMS Insert
```
Path: /1/oms/insert
Method: POST
Body: RefundRequest 
OMSPayload {
orderId (string),
imageUrl (string),
description (string),
amount (double),
disbursalText (string),
secondaryDisbursalText (string),
identityToken (string),
state (string) ['INITIATED' or 'SUCCESSFUL' or 'FAILED'],
items (Array[OMSItem]),
offers (Array[OMSOffers], optional),
timestamp (long)
}
OMSItem {
itemId (string),
title (string),
amount (long),
category (string)
}
OMSOffers {
offerId (string),
title (string),
source (string)
}
Response: String
```
```
Sample Request
{
	"orderId":"dummyOrder1",
	"imageUrl": "http://www.google.com/img/test",
	"description":"This is an order id",
	"amount": 220.14,
	"disbursalText":"You order has been recharged",
	"secondaryDisbursalText":"Recharge is already done, Go get a life",
	"identityToken":"IDTKNE4783BAFB3B54713A46EC4B9EB5D59DEKNG",
	"state":"DUMMY",
	"timestamp": 1523960011000,
	"items": [
		{
			"itemId" : "dummyItemId1",
			"title" : "This is the recharge product you just purchased",
			"amount" : 220.14,
			"category" : "recharge"
		}
	],
	"offers": [
		{
			"offerId" : "dummyOfferId",
			"title" : "Get everything for free",
			"source" : "merchant"
		}
	]
}
```
###OMS State update
```
Path: /1/oms/update/state
Method: PUT
Query: 
orderId : String
state : String  ['INITIATED' or 'SUCCESSFUL' or 'FAILED']
```

###OMS Refund
```
Path: /1/oms/create/refund
Method: PUT
Body: 
RefundPayload {
orderId (string),
transactionId (string),
amount (double),
source (string),
title (string),
refundId (string)
}
```
```
Sample Request
{
	"orderId" : "dummyOrder1",
	"transactionId" : "dummyRefundTransactionId",
	"amount" : 20.12,
	"source" : "merchant",
	"title" : "Refund because merchant says so",
	"refundId" : "dummyRefundId"
}
```

##Security
All production endpoints will be over https.
We will validate Client’s identity using JWT.
The JWT will be shared in every call in header as a ‘secureToken’

Properties of secure token
Algorithm RS256
Fields to be present in payload
iss -> Issuer -> which is equal to clientId
iat -> Issued At -> Timestamp with seconds resolution when the token was generate. This will fail if in future.
exp -> Expiry At -> Timestamp with seconds resolution when the token expires. exp-iat should not be more than 1000
The Public key should be shared with flipkart before hand.

##Test endpoint for secure token
Path: /1/dummy
Method: GET
Header: secureToken
Response:  Will be HTTP 200 if secureToken is valid otherwise response code will be self explanatory.