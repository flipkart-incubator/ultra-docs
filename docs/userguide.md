##Integration steps on Client Side
###Step 1
Paste this within your HTML's head tag.

`<script src="https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fkext-browser-min@0.0.8.js" type="text/javascript"></script>`

###Step 2
Initialized the SDK with your clientId. Contact flipkart to generate a clientId and secret.

`<script type="text/javascript">
var clientId = "playground";
var fkPlatform = FKExtension.newPlatformInstance(clientId);
</script>`
Copy
###Step 3
Call getToken to get a grant token

`var scopeReq = [{"scope":"user.email","isMandatory":true,"shouldVerify":false},{"scope":"user.mobile","isMandatory":false,"shouldVerify":false},{"scope":"user.name","isMandatory":false,"shouldVerify":false}];
fkPlatform.getModuleHelper().getPermissionsModule().getToken(scopeReq).then(
function (e) {
    console.log("Your grant token is: " + e.grantToken);
}).catch(
function (e) {
    console.log(e.message);
}
`

###Step 4
Send the token to you server using a AJAX call or any other mechanism. This token can be used by the server to get an access token. This access token can be use to fetch the users identity token as well as other resources like name, email address and verified mobile number on the server side. Refer the server side guide for details. Note : Always get user data on server side and not on the client side to avoid security risks like MITM attacks.
