# Guidelines to follow while integration
This section captures the guidelines for broad use cases which relates to almost all of our Ultra partners’ applications.

### General
These general guidelines apply across all the pages within a partner’s application.

- No external redirection in your app.
- No obstruction of Flipkart's property within Ultra.
- Your app experience must have parity.

### Component-wise
Here are the instructions to follow for special components within Ultra:

#### Entry
- ***Login or Sign-up:*** Do not have your own login or sign-up flows. Disable all paths in your application that lead to any of these pages. Same pertains to the log out and account modification (password change, profile change, etc.) processes.
- ***Homepage:*** Do not include promotions that belong to non-Flipkart properties and the advertisements from external parties on your homepage.

#### Order Funnel
- ***Product Discovery:*** There are no explicit restrictions around flows such as the Search, the Flyout menu and other navigation panels. But you must take care of UI elements that might confuse the user. For example, the flyout menu must not be identical to Flipkart’s flyout menu, etc.
- ***Offers:*** At any point in the funnel, do not display the offers that customers cannot avail within Flipkart’s context. Even the instructional notes, like the messages describing how to avail an offer in another platform, are not allowed.
- ***Cart:*** Do not link shopping carts outside the Ultra platform. Because we don’t want to encourage user behaviors where they treat Ultra as one channel for partners to convert from. Facilitating completion of dropped off orders is not a key value proposition. Besides, linking external carts would create confusion for the users and it does not shape a right behavior with this freedom.
- ***Checkout:*** There are no specific restrictions on the checkout page. Try to optimize your checkout funnel to ensure a minimal amount of drop-offs from the users. However, there are no specific guidelines as it may vary from one partner application to another.

#### Payments
Use Flipkart’s Payment Gateway (FKPG) only for taking payments which applies to both forward and reverse flows.

#### Post-Order placement
- ***My Orders:*** Restrict the orders displayed on this page to the ones placed through Flipkart. This is because of the following reasons:

  - Orders placed within Flipkart or through redirection are actionable by Flipkart’s Customer Executives (CX) and they not responsible for any external orders.
  - FKPG does not support modification of the orders involving payment flows placed outside Flipkart.
  
- ***Contact Us or Help Center:*** You can use your own order tracking, issue resolution and other CX flows but for orders placed on the same platform only.
