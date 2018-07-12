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
