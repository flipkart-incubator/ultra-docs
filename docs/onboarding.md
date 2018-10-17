# Onboarding a partner with Ultra

For on-boarding of a partner with Ultra platform, Flipkart has to keep a checklist of below mentioned steps to ensure a smooth transition:

* Put all legal contracts and business agreements with the partners to closure.

* Review information security and privacy policy agreement.

* Integrate API with both of the following services:

    * **Login:**

        * This service will integrate partner’s user service with oAuth2.0 built by Flipkart Authn service.

        * It will create and provide an identity token so that merchants have an account id equivalent and are not compromising Flipkart’s flow.

        * It will prompt a user for data sharing permissions when partner requests user’s data like email or phone number.

        * It will solicit missing data when a new use case arises at partner’s side and in return enriches Flipkart data.

        * It will also prompt email verification or phone verification flows within native application so that merchants don’t have to perform any verification on their end.

    * **Payment:**

        * This service will allow partners to configure their nodal account and MIDS to power payments completion via Flipkart payment gateway.

        * It will allow partners to decide the number and type of payment options for a user.

        * It will also allow partners to run bank offers on payments.

        * It will permit the partner to configure session or payment expiry timer, convenience or other fees.

        * The partners will take benefits of Fintech constructs like EMI, BNPL etc in the future.

        * It allows the partner to share the order and payment details like payment status to the user.

        * It will also help partners to issue refund and/or cancellation amount to the user.

* Ensure that there is a process agreement and semi-automated integration with the partner on:

    * **Customer experience:**

        * This will allow the visibility of orders in "My Orders" section.

        * It will enable the user to discover help in "Help Centre".

        * It will solve for communication on inbound and outbound calls.

    * **Accounting:**

        * It will handle reconciliation, Taxes, etc.

    * **Reporting:**

        * This will allow visibility of business metrics at both ends.

