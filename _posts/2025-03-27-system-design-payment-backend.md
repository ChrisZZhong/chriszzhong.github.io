---
layout: post
title: "System design - Payment - Amazon"
date: 2025-03-27
description: "Decode payment backend architecture"
tag: System Design
---

# Payment System - payment backend of e-commenrce app

```
func: 
    pay-in flow
    pay-out flow
nonfunc:
    reliability:
        handle failed payments
    reconciliation

deepDive:
    PSP integration
    reconciliation
    handle payment processing delays
    handle failed payments
    exactly-once delivery
    consistency
    security
```

## Reference

[Design Payment System](https://newsletter.pragmaticengineer.com/p/designing-a-payment-system)

## Requirenemnt gathering & Estimation

note that payment processor like stripe only handle credit/debt card payments.

```
1. what kind of payment? payment backend service or something like PSP (payment service provider)
    say payment backend service handle money movement related to orders
2. payment options: Credit/debt cards, PayPal etc?
    focus on credit/debt card
3. Can I assume we use third party payment processors to process our payments, so we do not need to store sensitive credit card data ourselves (extremely high security and compliance requirements)?
    yes, you can use Stripe, Braintree, Square, etc.
4. support diff currencies? 
    No, only one currency
5. support pay-out flow as well?
    Yes
6. Non-function for consistency, need reconcile?
6. QPS, estimation
``` 

**Functional Requirements:**

Pay-in flow: payment system receives money from customers on behalf of sellers.

Pay-out flow: payment system sends money to sellers around the world.

**Non-functinal Requirements:**

Reliability & fault tolerance: could handle payment failure

A reconciliation process between internal services (payment systems, accounting systems) and external services (payment service providers) is required. The process asynchronously verifies that the payment information across these systems is consistent.

## Pay-in flow

Pay-in flow is the flow that customer transfer the money to payment system.

<img src="/images/System-Design/IMG_1453.png">

* `payment service`: accept payment events and coordinate the payment process.
    - 1.risk check: AML(Anti-Money Laundering)/CFT (Combating the Financing of Terrorism) only process payment `pass risk check`
    - risk check service is provided by third party

* `Payment executor service` (third party): The payment executor executes a single payment order via a Payment Service Provider (PSP). A payment event may contain several payment orders.

* `Payment Service Provider (PSP)` eg. Stripe, Paypal
    - A PSP moves money from account A to account B. In this simplified example, the PSP moves the money out of the buyer’s credit card account. 

* `Card schemes`
    - Card schemes are the organizations that process credit card operations. Well known card schemes are Visa, MasterCard, Discovery, etc. The card scheme ecosystem is very complex [3].

* `Ledger`
    - The ledger keeps a financial record of the payment transaction. For example, when a user pays the seller $1, we record it as debit $1 from a user and credit $1 to the seller. The ledger system is very important in post-payment analysis, such as calculating the total revenue of the e-commerce website or forecasting future revenue. 

* `Wallet`
    - The wallet keeps the account balance of the merchant. It may also record how much a given user has paid in total. 

* **workflow:**
    - When a user clicks the “place order” button, a payment event is generated and sent to the payment service.
    - The payment service stores the payment event in the database.
    - Sometimes, a single payment event may contain several payment orders. For example, you may select products from multiple sellers in a single checkout process. If the e-commerce website splits the checkout into multiple payment orders, the payment service calls the payment executor for each payment order.
    - The payment executor stores the payment order in the database.
    - The payment executor calls an external PSP to process the credit card payment.
    - After the payment executor has successfully processed the payment, the payment service updates the wallet to record how much money a given seller has.
    - The wallet server stores the updated balance information in the database.
    - After the wallet service has successfully updated the seller’s balance information, the payment service calls the ledger to update it.
    - The ledger service appends the new ledger information to the database

## API

* POST v1/payments
    - inputs:
        - buyer_info: json
        - checkout_id: string
        - credit_card_info: encrypted string, for PSP usage
        - payment_orders: list of orders

        每个order里都有一个payment_order_id，用于发送给PSP生成token(nonce)

    - order entity:
        - seller_id
        - amount: `string` “amount” field is “string,” rather than “double”. Double is not a good choice, It is recommended to keep numbers in string format during transmission and storage. They are only parsed to numbers when used for display or calculation
        - currency
        - payment_order_id: string

Note that the payment_order_id is globally unique. When the payment executor sends a payment request to a third-party PSP, the payment_order_id is used by the PSP as the deduplication ID, also called the `idempotency key`

```
通常情况下，PSP会根据你传入的 payment_order_id（或者其他唯一标识符）生成一个唯一的 token。这个过程可以总结如下：

支付注册请求：你的支付服务会发送包含付款信息（如金额、货币、支付请求过期时间等）的支付注册请求到 PSP，支付服务还会附带 payment_order_id（作为幂等键，用于确保请求的唯一性）。

PSP生成Token：PSP 接收到请求后，会处理并生成一个唯一的 token，这个 token 用于标识该注册的支付请求。这个 token 通常是一个 UUID，由 PSP 创建并返回给你的支付服务。

存储Token：你的支付服务在收到这个 token 后，会将其保存到数据库中，以供后续流程使用。

支付页面操作：当客户端展示由 PSP 托管的支付页面时，PSP 的 JavaScript 库会使用这个 token 来从 PSP 后端检索详细的支付信息，并完成支付。

因此，payment_order_id 是你传递给 PSP 的幂等键，而 token 是 PSP 基于此支付注册生成的唯一标识，用于后续支付操作。
```

* GET /v1/payments/{:id} (check payment status)
    - output: the execution status of a single payment order based on payment_order_id. 

## Modeling 

* events:
    - checkout_id: String PK
    - buyer_info: String
    - seller_info: String
    - credit_card_info: depends on the card provider
    - is_payment_done: boolean

* Payment Order:
    - payment_order_id: String PK
    - buyer_account: String
    - amount: String
    - currency: String
    - checkout_id: String FK (link to the payment events)
    - payment_order_status: String (eunm NOT_STARTED, EXECUTING, SUCCESS, FAILED)
    - ledger_updated: boolean
    - wallet_updated: boolean

## double-entry principle

There is a very important design principle in the ledger system: the double-entry principle (also called double-entry accounting/bookkeeping [6]). Double-entry system is fundamental to any payment system and is key to accurate bookkeeping. It records every payment transaction into two separate ledger accounts with the same amount. One account is debited and the other is credited with the same amount

| Account | Debt | Credit |
|---------|------|--------|
| Buyer   | $1   |    |
| Seller  |  | $1     |


The double-entry system states that the sum of all the transaction entries must be 0. One cent lost means someone else gains a cent. It provides end-to-end traceability and ensures consistency throughout the payment cycle. To find out more about implementing the double-entry system, see Square’s engineering blog about immutable double-entry accounting database service

## Pay-out flow

The components of the pay-out flow are very similar to the pay-in flow. One difference is that instead of using PSP to move money from the buyer’s credit card to the e-commerce website’s bank account, the pay-out flow uses a third-party pay-out provider to move money from the e-commerce website’s bank account to the seller’s bank account. 

Usually, the payment system uses third-party account payable providers like Tipalti to handle pay-outs. There are a lot of bookkeeping and regulatory requirements with pay-outs as well.

## Deep dive

### PSP integration

<img src="/images/System-Design/IMG_1453.png">

If the payment system can directly connect to banks or card schemes such as Visa or MasterCard, payment can be made without a PSP. These direct connections are uncommon and highly specialized. They are usually reserved for really large companies that can justify such an investment. For most companies, the payment system integrates with a PSP instead, in one of two ways:

- If a company can safely store sensitive payment information and chooses to do so, PSP can be integrated using API. The company is responsible for developing the payment web pages, collecting and storing sensitive payment information. PSP is responsible for connecting to banks or card schemes.

- If a company chooses not to store sensitive payment information due to complex regulations and security concerns, PSP provides a hosted payment page to collect card payment details and securely store them in PSP. This is the approach most companies take.

--------------------------

1.The user clicks the “checkout” button in the client browser. The client calls the payment service with the payment order information. 

2.After receiving the payment order information, the payment service sends a payment registration request to the PSP. This registration request contains payment information, such as the amount, currency, expiration date of the payment request, and the redirect URL. Because a payment order should be registered only once, there is a UUID field to ensure the exactly-once registration. This UUID is also called nonce [10]. Usually, this UUID is the ID of the payment order. 

3.The PSP returns a token back to the payment service. A token is a UUID on the PSP side that uniquely identifies the payment registration. We can examine the payment registration and the payment execution status later using this token.

4.The payment service stores the token in the database before calling the PSP-hosted payment page.

5.Once the token is persisted, the client displays a PSP-hosted payment page. Mobile applications usually use the PSP’s SDK integration for this functionality. Here we use Stripe’s web integration as an example (Figure 5). Stripe provides a JavaScript library that displays the payment UI, collects sensitive payment information, and calls the PSP directly to complete the payment. Sensitive payment information is collected by Stripe. It never reaches our payment system. The hosted payment page usually needs two pieces of information:

- The token we received in step 4. The PSP’s javascript code uses the token to retrieve detailed information about the payment request from the PSP’s backend. One important piece of information is how much money to collect.

- Another important piece of information is the redirect URL. This is the web page URL that is called when the payment is complete. When the PSP’s JavaScript finishes the payment, it redirects the browser to the redirect URL. Usually, the redirect URL is an e-commerce web page that shows the status of the checkout. Note that the redirect URL is different from the webhook URL in step 9.

6.The user fills in the payment details on the PSP’s web page, such as the credit card number, holder’s name, expiration date, etc, then clicks the pay button. The PSP starts the payment processing.

7.The PSP returns the payment status.

8.The web page is now redirected to the redirect URL. The payment status that is received in step 7 is typically appended to the URL. For example, the full redirect URL could be [12]

9.Asynchronously, the PSP calls the payment service with the payment status via a webhook. The webhook is an URL (callback url) on the payment system side that was registered with the PSP during the initial setup with the PSP. When the payment system receives payment events through the webhook, it extracts the payment status and updates the payment_order_status field in the Payment Order database table.

### Reconciliation

This is a practice that periodically compares the states among related services in order to verify that they are in agreement. It is usually the last line of defense in the payment system.

Every night the PSP or banks send a settlement file to their clients. The settlement file contains the balance of the bank account, together with all the transactions that took place on this bank account during the day. The reconciliation system parses the settlement file and compares the details with the ledger system.

### Handling payment processing delays

- webhook (callback) url when PSP finished executing the payment

- periodically polling the result, but will put the burdon on payment service

### Handling failed payments

Every payment system has to handle failed transactions. Reliability and fault tolerance are key requirements. We review some of the techniques for tackling those challenges.

- Tracking payment state
    - Whenever a failure happens, we can determine the current state of a payment transaction and decide whether a retry or refund is needed. The payment state can be persisted in an append-only database table. each time, we get the latest state by time

- Retry queue and dead letter queue
    - Retry queue: retryable errors such as transient errors are routed to a retry queue. 
    - Dead letter queue: if a message fails repeatedly, it eventually lands in the dead letter queue. A dead letter queue is useful for debugging and isolating problematic messages for inspection to determine why they were not processed successfully. 

process: 

- Check whether the failure is retryable.
    - Retryable failures are routed to a retry queue.
    - For non-retryable failures such as invalid input, errors are stored in a database.

- The payment system consumes events from the retry queue and retries failed payment transactions.

- If the payment transaction fails again:
    - If the retry count doesn’t exceed the threshold, the event is routed to the retry queue.
    - If the retry count exceeds the threshold, the event is put in the dead letter queue. Those failed events might need to be investigated.

Retry configurations

```
1. Immediate retry
2. Fixed intervals
3. Incremental intervals
4. Exponential backoff
5. Cancel: the client can cancel the request. This is a common practice when the failure is permanent or repeated requests are unlikely to be successful.
```

### Exactly-once delivery - Avoid double payment

- execute at least one time (retry already handled)
- At the same time, it is executed at-most-once. (idempotent key handled)
    - if executed more than one time, will return previous message beased on the nonce

### consistency

for payment system consistency, we usually rely on idempotency and reconciliation.

for replica consistency

- Raft for replica
- write/read on prime db, hard to scale

### Security issues

| Risk                        | Mitigation Strategy                        |
|-----------------------------|-------------------------------------------|
| Request/Response Eavesdropping | Use HTTPS                               |
| Data Tampering               | Enforce encryption and integrity monitoring |
| Man-in-the-middle Attack     | SSL with certificate pinning              |
| Data Loss                    | DB replication, snapshot                  |
| DDOS                         | Rate limiting and firewall               |
| Card Theft                   | Tokenization                             |
| PCI Compliance               | PCI DSS                                  |
| Fraud                        | Address verification, cvv, user behavior analysis|
