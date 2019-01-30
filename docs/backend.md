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

### Payments
#### Token generation
Use this API to generate a payment token used to show the FKPG payment options screen via Ultra's client SDK. It opens FKPG outside Ultra’s container.

!!!note
    Remember that Flipkart provides the `category` value.

```
Path: 2/payment/token

Method: POST

Body: PaymentTokenRequest
PaymentTokenRequest {
merchantCredential(MerchantCredential),
amountPaise (long),
category(string),
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

**Sample Request:**
```
{
  "merchantCredential": {
    "name": "Enter name here",
    "password": "Enter hash here"
  },
  "merchantTransactionId": "transaction1",
  "merchantReferenceId": "order1",
  "amountPaise": 200,
  "paymentExpiryMilliSeconds": 100000,
  "category": "test",
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
**Sample Response:**
```
// To be included
```

#### Callback after PG Response
The system makes a POST call to `successfulCallBackUrl` as specified in the `paymentToken` request after the payment process is complete. Then, issue an HTTP redirect to the `fapp://` URL for redirection to the Ultra’s container. Also, specify the order confirmation page URL in the same format. This opens the order confirmation page within Ultra’s container while the payment happens externally. 

!!!note
    Please note that it is essential to redirect to Ultra’s container using `fapp://`. Do not open any UI outside this container.

**FORM data Payload:**
```
transaction_status=""&account_type=""&pg_trackid=""&merchant_adjustments=[ {'offer_id':'', 'offer_unit_price':0, 'amount_applied':0, 'amount_requested':0, 'actual_subvention_amount':0, 'effective_subvention_amount':0, ent_response_code':'', 'metadata':{'paymentSystem':'', 'offerId':'', 'discountType':''}} ]transaction_amount=""&emi_months=""&merchant_id=""&transaction_response_code=""&payzippy_transaction_id=""&having_multiple_transactions=""&bank_name=""&card_brand=""&hash_method=""&transaction_time=""&transaction_currency=""&payment_method=""&timestamp=""&merchant_key_id=""&primary_record={'transaction_id': '', 'primary_amount': ''}merchant_transaction_id=""&bank_transaction_id=""&payment_instrument=""&transaction_response_message=""&pg_mid=""&pg_name=""&pg_authcode=""&pg_id=""&is_international=""&fraud_action=""&is_risky_instrument=""&transaction_auth_state=""&hash=""&masked_card_number=""&transaction_status=""&account_type=""&pg_trackid=""&merchant_adjustments=""&transaction_amount=""&emi_months=""&merchant_id=""&transaction_response_code=""&payzippy_transaction_id=""&having_multiple_transactions=""&bank_name=""&card_brand=""&hash_method=""&transaction_time=""&transaction_currency=""&payment_method=""&timestamp=""&merchant_key_id=""&primary_record=""&merchant_transaction_id=""&bank_transaction_id=""&payment_instrument=""&transaction_response_message=""&pg_mid=""&pg_name=""&pg_authcode=""&pg_id=""&is_international=""&fraud_action=""&is_risky_instrument=""&transaction_auth_state=""&hash=""&masked_card_number=""
```

**Sample Request:**
```
Request sent as FORM parameters

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
**Sample Response:**
```
// To be included
```

#### Query
```
Path: 2/payment/query

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

**Sample Request:**
```
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
**Sample Response:**
```
// To be included
```

### Refund
#### Create
```
Path: /3/payment/refund/create

Method: POST
Body: RefundRequest

RefundRequest {
    merchantCredential (MerchantCredential),
    paymentMerchantTransactionId (string),
    refundAmountInPaise (long),
    refundMerchantTransactionId (string),
    refundReason (string),
    identityToken (string),
    orderDate(long, optional)
}

MerchantCredential {
    name (string),
    password (string)
}

Response : CreateRefundResponse

CreateRefundResponse {
    success (boolean),
    code (string),
    message (string),
    data (RefundQueryResponse)
}

RefundQueryResponse {
    parentRefund(ParentRefund),
    childRefundsDataList (Array[ChildRefundsData])
}

ParentRefund {
    id (string),
    status (string) = ["requested" or "init" or "processing" or "completed"
    or "failed" or "cancelled"],
    requestedAmountInPaise (long),
    orderId (string),
    refundType (string) = ['REFUND' or 'PAYOUT' or 'FREEBIE_REFUND' or
    'GOODWILL' or 'PRICE_DROP_REFUND' or 'CASHBACK' or 'SHIPPING_REFUND'
    'REDIRECTED_REFUND' or 'DEBIT' or 'EXCHANGE_REFUND' or 'REVERSE_PAYMENT'
    'REV_ADJUSTMENT' or 'OFFLINE'],
    clientRefId (string),
    refundReason (string),
    destinationId (string),
    createdAt (long),
    updatedAt (long)
}

ChildRefundsData {
    childRefund (ChildRefund),
    fundTransactions (Array[FundTransaction])
}

ChildRefund {
    private String id (string),

    status (string) = ["new" or "requested" or "triggered" or "init" or "cancelled" or "picked" or "processing" or "failed" or "completed"
    or "redirected" or "debited" or "splitted" or "clone_requested" or "cloned"],

    refundMode (string) = ['QC_EGV' or 'NEFT' or 'CREDIT_CARD' or 'IMPS' or 'E_GIFT_VOUCHER' or 'FLIPKART_CREDIT' or 'SCLP-WALLET' or 'COD' or
    'NETBANKING' or 'CREDIT_CARD_EMI' or 'DEBIT_CARD' or 'BACK_TO_SOURCE' or 'PART_NEFT' or 'QC-SCLP' or 'PHONEPE' or 'WALLET_DEFERRED' or
    'WALLET_INSTANT' or 'EGV-SCLP' or 'VISA_DIRECT' or 'EGV_WALLET' or 'EGV' or 'FLIPKART_FINANCE' or 'INVALID_REFUND_MODE' ],

    paymentGateway (string) = ['QC' or 'FLIPKART' or 'GCMS' or 'WALLET' or
    'APL' or 'FTS' or 'COD' or 'EGV' or 'DEFAULT'],

    refundAmountInPaise (long),
    forwardTransactionId (string),
    parentRefundId (string),
    promiseDate (long),
    bankReferenceNumber (string),
    createdAt (long),
    updatedAt (long)
}

FundTransaction {
    id (string),
    entity (string),
    entityId (string),
    status (string) = ["requested" or "init" or "processing" or "failed" or "completed" or "cancelled" or "debited"],
    transferStatus (string),
    transferResponseCode (string),
    externalRefId (string),
    pgMid (string),
    transactionAmountInPaise (long),
    parentRefundId (string),
    createdAt (long),
    updatedAt (long),
    dataBag (Map<String, String>)
}
```

**Sample Request:**
```
{
  "merchantCredential": {
    "name": "Enter name here",
    "password": "Enter hash here"
  },
  "paymentMerchantTransactionId": "transaction1",
  "refundAmountInPaise": 1000,
  "refundMerchantTransactionId": "transaction1-refund1"
  "refundReason": "testing",
  "identityToken": "Actual ID Token here"
}
```

**Sample Response:**
```
{
  "success": true,
  "code": "SUCCESS",
  "message": null,
  "data": {
    "parentRefund": {
          "id": "PR19010313341416072947206",
          "status": "init",
          "requestedAmountInPaise": 500,
          "orderId": "1527666755_68",
          "refundType": "REFUND",
          "clientRefId": "1527666755_67_refund_07",
          "refundReason": "CentaurAPITesting",
          "destinationId": null,
          "createdAt": 1546502654079,
          "updatedAt": 1546502654176
    },
    "childRefundsDataList": [
          {
            "childRefund": {
              "id": "CR1901031334148265203106",
              "status": "init",
              "refundMode": "NETBANKING",
              "paymentGateway": "FLIPKART",
              "refundAmountInPaise": 500,
              "forwardTransactionId": "PZT1901031332IJEMH01",
              "parentRefundId": "PR19010313341416072947206",
              "promiseDate": 1547193854000,
              "bankReferenceNumber": null,
              "createdAt": 1546502654179,
              "updatedAt": 1546502654182
            },
            "fundTransactions": []
          }
        ]
      }
    }
```

#### Query
***By Parent Refund ID:***
```
Path: /3/payment/refund/query/parentRefundId/{parentRefundId}

Method: GET

Response : RefundQueryResponse (Same as CreateRefundResponse)
```

**Sample Request:**
```
// To be included
```
**Sample Response:**
```
{
    "parentRefund": {
      "id": "PR19010313341416072947206",
      "status": "init",
      "requestedAmountInPaise": 500,
      "orderId": "1527666755_68",
      "refundType": "REFUND",
      "clientRefId": "1527666755_67_refund_07",
      "refundReason": "CentaurAPITesting",
      "destinationId": null,
      "createdAt": 1546502654079,
      "updatedAt": 1546502654176
    },
    "childRefundsDataList": [
      {
        "childRefund": {
          "id": "CR1901031334148265203106",
          "status": "processing",
          "refundMode": "NETBANKING",
          "paymentGateway": "FLIPKART",
          "refundAmountInPaise": 500,
          "forwardTransactionId": "PZT1901031332IJEMH01",
          "parentRefundId": "PR19010313341416072947206",
          "promiseDate": 1547193854000,
          "bankReferenceNumber": null,
          "createdAt": 1546502654179,
          "updatedAt": 1546502654359
        },
        "fundTransactions": [
          {
            "id": "RT19010313341426309566006",
            "entity": "child_refund",
            "entityId": "CR1901031334148265203106",
            "status": "processing",
            "transferStatus": "DEPENDENCY_SERVICE_FAILURE",
            "transferResponseCode": "FKPG_EXCEPTION",
            "externalRefId": null,
            "pgMid": null,
            "transactionAmountInPaise": 500,
            "parentRefundId": "PR19010313341416072947206",
            "createdAt": 1546502654360,
            "updatedAt": 1546502654376,
            "dataBag": {
              "key": "FKPG",
              "bank_reference_number": null,
              "reference_id": null,
              "pg_track_id": null
            }
          }
        ]
      }
    ]
  }
}
```

***By Order ID:***
```
Path: /3/payment/refund/query/orderId/{orderId}

Method: GET

Response : Array[RefundQueryResponse] (Same as CreateRefundResponse)
```

**Sample Request:**
```
// To be included
```
**Sample Response:**
```
[
    {
      "parentRefund": {
        "id": "PR19010313341416072947206",
        "status": "init",
        "requestedAmountInPaise": 500,
        "orderId": "1527666755_68",
        "refundType": "REFUND",
        "clientRefId": "1527666755_67_refund_07",
        "refundReason": "CentaurAPITesting",
        "destinationId": null,
        "createdAt": 1546502654079,
        "updatedAt": 1546502654176
      },
      "childRefundsDataList": [
        {
          "childRefund": {
            "id": "CR1901031334148265203106",
            "status": "processing",
            "refundMode": "NETBANKING",
            "paymentGateway": "FLIPKART",
            "refundAmountInPaise": 500,
            "forwardTransactionId": "PZT1901031332IJEMH01",
            "parentRefundId": "PR19010313341416072947206",
            "promiseDate": 1547193854000,
            "bankReferenceNumber": null,
            "createdAt": 1546502654179,
            "updatedAt": 1546502654359
          },
          "fundTransactions": [
            {
              "id": "RT19010313341426309566006",
              "entity": "child_refund",
              "entityId": "CR1901031334148265203106",
              "status": "processing",
              "transferStatus": "DEPENDENCY_SERVICE_FAILURE",
              "transferResponseCode": "FKPG_EXCEPTION",
              "externalRefId": null,
              "pgMid": null,
              "transactionAmountInPaise": 500,
              "parentRefundId": "PR19010313341416072947206",
              "createdAt": 1546502654360,
              "updatedAt": 1546502654376,
              "dataBag": {
                "key": "FKPG",
                "bank_reference_number": null,
                "reference_id": null,
                "pg_track_id": null
              }
            }
          ]
        }
      ]
    }
  ]
```

## OMS
### Schema
***Order:***
|name|type|mandatory|notes|
|---|---|---|---|
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
|---|---|---|---|
|itemId|string|yes|It is an item identifier. It draws the customer to a product from order page|
|title|string|yes|It is the description of an item. It is visible to the customer|
|image|string|yes|Item's image|
|basePrice|double|yes|Price of an item before applying any adjustments|
|finalPrice|double|yes|Price of an item after applying both `merchantAdjustments` and `flipkartAdjustments`. This is `null` if the state is `INIT`|
|category|string|yes|Category to which the item belongs. Flipkart provides this value. Please put no other values here|
|fulfillmentDate|long|yes|Date of fulfillment of the item (For e.g., delivery date, travel date, etc.)|
|itemState|enum(`INIT` or `SUCCESSFUL` or `CANCELLED` or `PENDING`)|yes|`INIT`: Used when the customer has not paid. <br/>
`PENDING`: Payment is successful, however, the merchant is yet to confirm the order. <br/>
`SUCCESSFUL`: Confirmed the item but not completely. Partial cancellation is also a part of this state. <br/> 
`CANCELLED`: Cancelled the item. Here, the complete final price should either reflect in `Reverse Transaction` or `cancellationCharges`|
|brand|string|yes|Provider/Manufacturer of product|
|product|string|yes|Description of the product. Sometimes, it is the Title of any e-commerce product. Other times, it provides specific details like Flight number, Recharge type, etc.|
|customerName|string|yes|Name of the person who has ordered the item|
|quantity|integer|yes|Number of products clubbed in the same order. In case of a category as `travel`, it is the number of passengers. If you have no such business requirement, send `1`|

***Forward Transaction:***
|name|type|mandatory|notes|
|---|---|---|---|
|transactionId|string|yes|PG shares the `transactionId`|
|amount|double|yes|Final amount deducted from customer|
|description|string|yes||
|timestamp|long|yes|Time of transaction|

***Reverse Transaction:***
|name|type|mandatory|notes|
|---|---|---|---|
|forwardTransactionId|string|yes|The forward transaction used to take money from customer|
|reverseTransactionId|string|yes||
|amount|double|yes|Refund amount|
|description|string|yes||
|timestamp|long|yes|Time of transaction|

***Merchant Adjustment:***
|name|type|mandatory|notes|
|---|---|---|---|
|adjustmentId|string|yes|Identifier for the adjustment on the merchant’s side|
|amount|double|yes|Adjustment amount|
|title|string|yes|Adjustment description. This could be visible to the customer|

***Flipkart Adjustment:***
|name|type|mandatory|notes|
|---|---|---|---|
|adjustmentId|string|yes|Identifier for the adjustment provided by Flipkart in PG response|
|amount|double|yes|Adjustment amount|

***Cancellation Charges:***
|name|type|mandatory|notes|
|---|---|---|---|
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
