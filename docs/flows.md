#Flows
These flow diagrams will help you understand flow of data between systems well.


## High level diagram
![Architecture](img/image1.png)

## Scope Selection / Modification
Showing the permission prompt with Allow/Deny buttons to the user.

![Scope Selection](img/image2.png)

##Fetching Resource 
Fetching Oauth resources like `user.name`, `user.email`

![Fetching Resource](img/image4.png)

##Legend

- Client UI / Partner UI : The HTML / React-native UI which renders your experience within flipkart app.
- SDK : The Javascript client SDK which exposes all Ultra client side APIs.
- Client server : Your backend server.
- Ultra API : Public API exposed by Ultra (`platform.flipkart.net`)
