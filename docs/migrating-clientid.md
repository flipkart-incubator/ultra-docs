#Migrating ClientId

## Introduction
`clientId` is the primary identifier for identifying an ultra app.
Until now `clientId` was just the `companyName`.

Starting 9th August '18, `clientId` has become a combined string equal to `<companyName>.<appId>`

Here's a chart on the new values of `clientId` :

| Old ClientId | New ClientId     |
|--------------|------------------|
| mmt          | mmt.flight       |
| phonepe      | phonepe.recharge |
| playground   | playground.home  |


Deeplinks which open an ultra app within Flipkart consists of `clientId` as a parameter.
These deeplinks have to be updated to use the new clientId.

!!!note
	To ensure backward compatibility, the old app can understand the new `clientId` as well. This means you can either pass the old clientId or the new clientId and the app behaves exactly the same way.

## Handling `clientId` inconsistency

Once an ultra app is opened with a `clientId`, all the navigations should be based on the same `clientId`. 

This can potentially cause issues in case of payment redirects where post-payment `fapp://` [redirect url](clients#start-payment) specifies the newer `clientId` like `mmt.flight` but the app was opened with `clientId` set to `mmt`. This could potentially be caused if a user has a SMS/Whatsapp deeplink which contained the older `clientId`. 

Once this happens, due to the above issue, the navigation stack which usually looks like this :
 `Flipkart Homepage ➡ Ultra (clientId = mmt)`

  will now look like this 

  `Flipkart Homepage ➡ Ultra (clientId = mmt) ➡ Payments ➡ Ultra (clientId = mmt.flight)`


The manifestation of the above issue cannot be noticed once payment is complete, but can be easily seen when the user presses back button and retraces his navigation stack.

!!!warning
	To avoid clientId inconsistency, the changeover from old to new clientId has to be done simultaneously across systems which depend on clientId (for e.g SMS/Whatsapp, payment redirect, ultra landing page all of them should be in sync)

!!!note
	Newer apps which went live after this change and use the new `clientId` structure will be unaffected by this and can ignore this article.

!!!note 
	Whenever you migrate to the new `clientId` format, just do not migrate the deeplinks and the `fapp://` redirect to it until all systems are ready to migrate , and this will avoid the UI inconsistency issues.
