##New JS SDK Methods
_20/Aug/2018_

New JS SDK methods added in v1.0.0. Only available in v6.7 of FK app.

* [Navigate to Flipkart](clients.md#navigate-to-flipkart): To navigate to any flipkart native page
* [Notify page location change](clients.md#notify-page-location-change-react-native-only): To notify that your page location has changed (React native only)


##New v2 Server Side APIs
_09/Aug/2018_

We have updated APIs for following

* New way for generating [secureToken](backend.md#security). Earlier we used to just take company name as clientId. From now on `clientId = <companyName>.<appId>`
* V2 for get active offers. [Active offers](backend.md#get-offers-list)
* V2 for payment APIs
    * [PaymentToken](backend.md#payment-token).  Added category in request.
    * [Query](backend.md#query). Linked to new secureToken.
    * [Refund](backend.md#refund). Linked to new secureToken.
* Do not forget to update your clientId in post payment [fapp:// redirect](clients.md#start-payment) too. 

## OMS contracts updated
_12/July/2018_

After feedback from merchants we have decided to change the following in our v2 OMS contracts.

* Addition of orderUrl in [Order](oms.md#order).
* Addition of brand, product, customerName, quantity in [Item](oms.md#item).

----

## Root Cause Analysis of Order failures on MMT (07/07/2018)

### Summary
From approximately 2pm to 9pm on 7 July even if users made the payment for ticket, order was not getting confirmed. There was 100% failure and no successful order were placed in this time.

### Event Description
Ultra's offers were configured using a json config file. There was a planned activity going on to move offers from this file to a database fronted with UI, so that the offers can be edited by business and be exposed to merchants using Offer API. The changes were coded and merged tested on local and then deployed on master. To ensure that double offer is not applied, Offer DB was populated to be a copy of config that was live. Post this console was expected to go live and exposed to business. However due to various reasons, the console go live was delayed. Because of which the transition period where both Offer database and Offer Config we appending offers was extended over weekend. All of this was done on 4th July. There was no impact on production because both config and database were appending the same offers.

On 7th July at 2pm the config was changed to have new offers. Because of this FKPG got 2 offers: one from db (the old one) and another from config(the new one). Because of this MMT was getting 2 offers in response. As a result of which they were not confirming the orders on their side. This was corrected at 9 pm when Offers DB was disabled.

### Chronology of Events/Timeline
* 4 July :
    * Offer DB deployed to prod with offers in sync with Offers Config.
* 5 July :
    * Offer Console was not deployed due to tech issues. **Offers DB was not disabled.**
    * Offers on both sources are in sync, hence production is stable.
* 7 July :
    * 2:00 pm: Offers changed in config. Leading to double offers. MMT does not support multiple offers applied in a single transaction , it right fully stopped processing orders for these payment transaction.
    * 5:30 pm: MMT informs Flipkart that none of the orders are getting confirmed.
    * 8:37 pm: Tech team was notified about the issue.
    * 9:00 pm: After investigation Flipkart tech realises that the issue is with offer DB being out of sync. Offer DB was disabled. Production was stabilised.

### Findings
**Q** Why was transition period long.

**A** The console deployment had some issues. Because of which switchover was blocked.

**Q** How could the transition period have been shortened.

**A** Rather than keeping the config in sync, we should have disabled offer DB, till console was not deployed.

**Q** Why did it take 3 hours + for flipkart to debug.

**A** There is no on-call email alias published for Ultra. By the time it came through to the engineering team 3 hours had elapsed.

### Corrective Action
* We are setting up an on call alias for ultra, which should be used by all merchants for any production issue. Every stakeholder of ultra will be a part of this on call so that all issues are broadcasted immediately.
* Define a play book for configuring offers and publish it. This playbook will include standard operating procedures around monitoring the health after an offer has been deployed
* Have a weekly deployment cycles on Thursday of every week, allowing for one business day to notice and fix any issues. Hot fixes can still be done on Friday/Weekends with an on call support.
* Changes in ultra will be broadcasted beforehand internally for other's FYI.

### Open Tickets
* **Ultra-33** : Create an alias for ultra-oncall
* **Ultra-34** : Create a playbook for configuring offers

----

## Root Cause Analysis of Ultra Outage (24/06/2018-25/06/2018)

### Summary
Ultra api’s were throwing internal server error from 24th Jul 2018 (2:20 am) to 25th Jul 2018(10:40am). Preventing new user’s to come onboard ultra platform as well as transacting.

### Event Description
On 24th Jun 2018 at about 2:20am an ultra API deployment was done to patch a bug for feature being developed right now. The patch was small and only for on of internal APIs. However along with above mentioned patch, rate limiting which was merged to master was also deployed. Since the config for rate limiting was only set for our test app, after deployment MMT and Phonepe api’s were throttled to 0.
The issue was observed on 25th 10:19am after MMT notified that APIs are not working correctly. The issue was patched by adding proper config in config service and the issue was rectified by 10:40 am.

### Chronology of Events/Timeline
* 11th Jun 2018: Rate limiting implemented and tested for playground app. Merged to master.
* 22th Jun 2018
    * 5:00pm : Patched a bug for an internal API, merged to master after code review and testing.
    * 5:10pm : Deployment failed due to bug in deployment service. Raised a ticket and informed
internal team that deployment will be done asap.
* 24th Jun 2018
    * 1:00 am: Verified that deployment service bug is fixed and started deployment to preprod.
    * 2:00 am: Verified the internal API on preprod and propagated the changes to prod.
    * 2:20 am: Prod deployment completed.
* 25th Jun 2018
    * 12:25 am: MMT notified on slack that the API is not working.
    * 10:21 am: Ultra team noticed the bug and started debugging. * 10:40 am: Correct config updated for both Phonepe and MMT.

### Findings and Root Cause
**Q** Why did the APIs fail?

**A** The APIs failed because rate limiting was deployed without correct config.

**Q** Why was Config not present for rate limiting?

**A** We consider this as a miss during the code review and testing process. Config should have been propagated for all merchant’s before the code was merged to master.

**Q** Why was the issue not detected earlier?

**A** There was a dip in QPS numbers. And Alerts were not triggered.

**Q** Are the alerts working?

**A** While figuring out this issue we have figured out that our alerting are not working and will be fixed asap.

### Corrective Action
We will take the following corrective actions.
* Fix and test alerting.
* Ensure that deployments don’t happen over weekend.
* Post every deployment test all API’s thoroughly.
* Have regular deployments so that testing the delta is manageable.
* Solve open tickets.

### Open Tickets
**Ultra-5** : Figure out how to add alerts on QPS and HTTP response codes. (**Resolved**)

**Ultra-4** : Investigate why error log growth did not cause an alert. (**Resolved**)
