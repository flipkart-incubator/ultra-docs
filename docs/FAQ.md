FAQ’s


Q1. What is client id and its different usage across the integration  ?

A1.  	Client id = Merchantid.appid
where Merchantid = 'hotstar' and appid= 'test' or 'services' [Assumption]
For Testing the Ultra SDK and loading the hotstar experience , use Client id as 'hotstar.test' or 'hotstar.services'.
For generating secure token , use issuer name as 'hotstar.test' or 'hotstar.services'.
For Access token call , use Client id as per the naming convention (Merchant id in this case ) as 'hotstar'
 
 
 
Q2.  Receiving “Bad Grant token” while making an Access token Api call ?
 
A2.  This error comes due to following reasons : 
Client secret is not URL encoded .
You are passing client id ( Merchantid.appid) instead of Merchant_id as mentioned in the above question .
 
 
 
Q3. What is the Validity of Different Tokens ?
 
A3. Grant token : One time use .
Access token(JWT token) : Validity of 1 hour . 
Identity token : Same for Customer (Except for case where cx changes his/her mobile number with flipkart )



Q4. What are the different domains for Both SSO and OMS integration steps ?

A4. For SSO process ,
Both Prod and preprod : https://platform.flipkart.net
For OMS process ,
Prod : https://ultra.flipkart.net
Preprod : https://ultra-lite.flipkart.net


Q5. How to start with the integration process ?

A5 . Fetch the grant token which is client side call needs to be made through Flipkart SDK which can be imported in your web application .
Debug apk link 
https://drive.google.com/file/d/1A77aLrok52bPGvwm_rp-sFJq5ycTWHyg/view
Usage : install the apk(after uninstalling the existing Flipkart application ) and enter client id to open the preprod environment and Prod environment  


Q6. What is the entry point for initiating the SSo process?

A6. Ideally , when Cx is entering the partner experience after clicking on partner widget or when at the checkout page .


Q7.  Difference between all the order states in the OMS process ?

A7.   ORDER_CONFIRMED - when the order got created with successful payment status .
ORDER_FULFILLED -  Terminal state when order is delivered and return policy if any for   the order is completed .
ORDER_CANCELLED - Terminal state when order is cancelled or Cx asked for refund .

Note : after the terminal state is reached , the FK system won’t accept any order event for    an order . 2 order state events for an order .

