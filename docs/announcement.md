# Announcements / Latest News

### Primary identifier being migrated starting August 9, 2018

There is a change in the primary identifier (i.e. `clientId`) starting from 9th August 2018 used for identifying a client on Ultra application. Before that, `clientId` was comprising only one attribute `companyName` but now `clientId` will become a collective string of two attributes i.e. `companyName` and `appId` which will exist in long form as `<company Name>.<app Id>`.

Following is a comparison between the old and new values of `clientId` for different clients:

| Old ClientId | New ClientId     |
|--------------|------------------|
| mmt          | mmt.flight       |
| phonepe      | phonepe.recharge |
| playground   | playground.home  |


You must update the Deeplinks that support Ultra application within Flipkart to use the new `clientId` as a parameter.

**How to handle `clientId` inconsistency?**

When you open the Ultra application with a specific `clientId`, it should carry the same `clientId` parameter value for further navigations to avoid inconsistencies. If `clientId` value varies, it might cause problems in case of payment redirections. The post-payment redirect URL (`fapp://action?value={"screenType":"ultra","params":{"url":"<URLENCODED_URL_TO_OPEN_WITHIN_ULTRA>","clientId":"<YOUR_CLIENT_ID>","postPaymentFlow":true,"success":<TRUE_OR_FALSE>}}`) will show a new `clientId` value which will differ from its first value at the time of opening of the application. An example of this scenario is that if user opened the Ultra application with `clientId` value as `mmt`, after the update, the new value becomes as `mmt.flight`. This may cause issues when the user has a Deeplink of SMS or WhatsApp carrying an older `clientId` value.

This further impacts the navigation stack which will appear from `Flipkart Homepage ➡ Ultra (clientId = mmt)` to `Flipkart Homepage ➡ Ultra (clientId = mmt) ➡ Payments ➡ Ultra (clientId = mmt.flight)`. One can capture this when a user has pressed the back button and retraced the navigation stack but same goes unnoticed once the payment cycle is complete.

Thus, you need to change the `clientId` value from old to new one simultaneously across all the systems that depend on it. Some of the examples where this inconsistency may arise are SMS or WhatsApp, payment redirections or Ultra landing page. Everything should be in synchronization with respect to the `clientId` value in the application.

!!!note
* This change does not impact the behavior of the old application because you can either pass the old or the new value of the identifier and it can understand both to make sure backward compatibility.
* Applications that went live post this change will stay unaffected as they are using the new `clientId` format.
* While migrating to the new clientId format, you need to make sure you do not migrate the Deeplink and the redirect URL unless rest of the systems are ready to migrate as this will avoid any UI inconsistency.
