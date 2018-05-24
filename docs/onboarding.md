# Onboarding process
Today integration with a new partner requires us to take following steps.
* Closure on **Legal Contracts** and **Business Agreement** with partners.

* **Information security** review and privacy policy Agreement.

* **API Integration** with

    * **Login** as a service which

        * Integrates partners user service with oAuth2.0 built by Flipkart Authn Service.

        * Creates and Provides IdentityToken so that merchants have accountId equivalent, but yet user’s flipkart flow is not compromised.

        * Prompts user for data sharing permissions when partner requests user’s data like email/phone number.

        * Solicits missing data when a new use case arises at partners and in return enriches Flipkart data.

        * Prompts email verification/phone verification flows within native app so that merchants don’t have to do verification on their end.

    * **Payment** as a service which

        * Lets partner configure their nodal account and MIDS to power payments completion via flipkart payment gateway.

        * Lets partner decide the allowed payment options for a user.

        * Lets partner Run payment bank offers.

        * Lets partner configure expiry timer, convenience/ other fees.

        * Lets partner take the benefits of Fintech constructs like EMI, BNPL etc in future.

        * Lets partner share order and payment details like payment status.

        * Lets partner issue refunds and cancellations.

* Process Agreement and Semi automated integration with

    * **Customer experience**

        * which solves for Visibility of orders on My Orders

        * which solves for Discovery of help in Help Centre

        * which solves for Communication on inbound and outbound calls

    * **Accounting**

        * Which solves for reconciliation, Taxes, etc

    * **Reporting**

        * Which solves for visibility of business metrics periodically at both ends.
