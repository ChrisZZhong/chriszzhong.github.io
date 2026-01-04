---
layout: post
title: "System Design - Robinhood"
date: 2025-12-17
description: ""
tag: System Design
prime: false
---

## Common Problems

Design Robinhood
================

Understanding the Problem
-------------------------

**📈 What is [Robinhood](https://robinhood.com/)?** Robinhood is a commission-free trading platform for stocks, ETFs, options, and cryptocurrencies. It features real-time market data and basic order management. Robinhood isn't an exchange in its own right, but rather a stock broker; it routes trades through market makers ("exchanges") and is compensated by those exchanges via [payment for order flow](https://en.wikipedia.org/wiki/Payment_for_order_flow).

### Background: Financial Markets

There's some basic financial terms to understand before jumping into this design:

*   **Symbol**: An abbreviation used to uniquely identify a stock (e.g. META, AAPL). Also known as a "ticker".
    
*   **Order**: An order to buy or sell a stock. Can be a _market order_ or a _limit order_.
    
*   **Market Order**: An order to trigger immediate purchase or sale of a stock at the current market price. Has no price target and just specifies a number of shares.
    
*   **Limit Order**: An order to purchase or sell a stock at a specified price. Specifies a number of shares and a target price, and can sit on an exchange waiting to be filled or cancelled by the original creator of the order.
    

Outside of the above financial details, it's worth understanding the responsibilities of Robinhood as a business / system. **Robinhood is a brokerage and interfaces with external entities that actually manage order filling / cancellation.** This means that we're building a brokerage system that facilitates customer orders and provides a customer stock data. _We are not building an exchange._

For the purposes of this problem, we can assume Robinhood is interfacing with an "exchange" that has the following capabilities:

*   **Order Processing**: Synchronously places orders and cancels orders via request/response API.
    
*   **Trade Feed**: Offers subscribing to a trade feed for symbols. "Pushes" data to the client every time a trade occurs, including the symbol, price per share, number of shares, and the orderId.
    

For this interview, the interviewer is offering up an external API (the exchange) to aid in building the system. As a candidate, it's in your best interest to briefly clarify the exchange interface (APIs, both synchronous and asynchronous) so you have an idea of the tools at your disposal. Typically, the assumptions you make about the interface have broad consequences in your design, so it's a good idea to align with the interviewer on the details.

### [Functional Requirements](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#1-functional-requirements)

**Core Requirements**

1.  Users can see live prices of stocks.
    
2.  Users can manage orders for stocks (market / limit orders, create / cancel orders).
    

**Below the line (out of scope)**

*   Users can trade outside of market hours.
    
*   Users can trade ETFs, options, crypto.
    
*   Users can see the [order book](https://www.investopedia.com/terms/o/order-book.asp) in real time.
    

This question focuses on stock viewing and ordering. It excludes advanced trading behaviors and doesn't primarily involve viewing historical stock or portfolio data. If you're unsure what features to focus on for a feature-rich app like Robinhood or similar, have some brief back and forth with the interviewer to figure out what part of the system they care the most about.

### [Non-Functional Requirements](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#2-non-functional-requirements)

**Core Requirements**

1.  The system prefers high consistency for order management; it's _essential_ for users to see up-to-date order information when making trades.
    
2.  The system should scale to a high number of trades per day (20M daily active users, 5 trades per day on average, 1000s of symbols).
    
3.  The system should have low latency when reflecting symbol price updates and when placing orders (under 200ms).
    
4.  The system should minimize the number of active clients connecting to an external exchange API. Exchange data feeds / client connections are typically expensive.
    

**Below the line (out of scope)**

*   The system connects to multiple exchanges for stock trading.
    
*   The system manages trading fees / calculations (we can assume fees are not in scope).
    
*   The system enforces daily limits on trading behavior.
    
*   The system protects against bot usage.
    

Here's how it might look on a whiteboard:

Non-Functional Requirements

For this question, given the small number of functional requirements, the non-functional requirements are even more important to pin down. They characterize the complexity of these deceptively simple live price / order placement capabilities. Enumerating these challenges is important, as it will deeply affect your design.

The Set Up
----------

### Planning the Approach

Before you move on to designing the system, it's important to start by taking a moment to plan your strategy. Generally, we recommend building your design up sequentially, going one by one through your functional requirements. This will help you stay focused and ensure you don't get lost in the weeds as you go. Once you've satisfied the functional requirements, you'll rely on your non-functional requirements to guide you through the deep dives.

### [Defining the Core Entities](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#core-entities-2-minutes)

Let's go through each high level entity. I like to do this upfront before diving into other aspects of the system so we have a list of concepts to refer back to when talking about the details of the system. At this stage, it isn't necessary to enumerate every column or detail. It's all about laying the foundation.

For Robinhood, the primary entities are pretty straightforward:

1.  **User**: A user of the system.
    
2.  **Symbol**: A stock being traded.
    
3.  **Order**: An order for a buy or sell, created by a user.
    

In the actual interview, this can be as simple as a short list like this. Just make sure you talk through the entities with your interviewer to ensure you are on the same page.

Core Entities

### [The API](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#api-design-5-minutes)

The API is the primary interface that users will interact with. It's important to define the API early on, as it will guide your high-level design. We just need to define an endpoint for each of our functional requirements.

Let's start with an endpoint to get a symbol, which will include details and price data. We might have an endpoint like this:

    GET /symbol/:name
    Response: Symbol

To create an order, an endpoint might look like this:

    POST /order
    Request: {
      position: "buy",
      symbol: "META",
      priceInCents: 52210,
      numShares: 10
    }
    Response: Order

Note we're using priceInCents instead of price to avoid floating point precision issues. Especially for financial application it's better to use integers to avoid errors and [financial scams](https://screenrant.com/justice-league-incarnate-superman-iii-scheme-easter-egg).

To cancel an order, the endpoint could be as simple as:

    DELETE /order/:id
    Response: {
      ok: true
    }

Finally, to list orders for a user, the request could be:

    GET /orders
    Response: Order[] (paginated)

With each of these requests, the user information will be passed in the headers (either via session token or JWT). This is a common pattern for APIs and is a good way to ensure that the user is authenticated and authorized to perform the action while preserving security. You should avoid passing user information in the request body, as this can be easily manipulated by the client.

[High-Level Design](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#high-level-design-10-15-minutes)
---------------------------------------------------------------------------------------------------------------------------

### 1) Users can see live prices of stocks

The first requirement of Robinhood is allowing users to see the live price of stocks. This might be one stock or many stocks, depending on what the user is viewing in the UI. To keep our design extensible, let's assume a user can see many live stock prices at once. To support this design, let's analyze a few options.

### 

Bad Solution: Polling Exchange Directly

##### Approach

This solution involves polling the exchange directly for price data per symbol. The client would poll every few seconds (per symbol), and update the price shown to the user based on the response.

##### Challenges

This is a simple approach that will not scale and will not minimize exchange client connections / calls. There's a few fundamental problems with this design:

*   **Redundant Exchange Calls**: This approach is a very inefficient way to get price information in terms of exchange call volume. It involves polling, which happens indiscriminately, even if the price has not changed. Additionally, it involves many clients requesting the same information from the exchange, which is wasteful. If 5000 clients are requesting a price at the same time, the price isn't different per client, yet we're expending 5000 calls (repeatedly) to disperse this information.
    
*   **Slow Updates**: If we're pursuing a polling solution, we'll see slower updates than we'd like to pricing information of symbols. In the non-functional requirements, we indicated we wanted a reasonably short SLA to update clients about symbol prices (under 200ms), and the only way we'd guarantee that with this solution is if we poll data every 200ms, which is unreasonable.
    

For a design like this, we can rule out polling the exchange directly as a viable option.

Polling

### 

Good Solution: Polling Internal Cache

##### Approach

This solution still involves polling for price information, but we're polling a symbol service that performs a key-value look-up on an internal cache that is kept up-to-date by a symbol price processor that is listening to prices on the exchange.

This approach prevents excess connections to the exchange by "proxying" it with a service that listens and records the most essential detail: the symbol price. This price is then made available to clients of Robinhood via polling.

##### Challenges

This approach is certainly an improvement, but is still subject to some issues. Clients still indiscriminately poll, even if the price has not changed, leading to some wasted HTTP traffic. Additionally, this approach is a slow way to get price updates; the polling interval dictates the worst-case SLA for a price update propagating to the client. Can we do better?

Polling Internal Cache

### 

Great Solution: Server Sent Event (SSE) Price Updates

##### Approach

A great approach here involves Server Sent Events (SSE). SSE is a persistent connection (similar to websockets), but it is unidirectional and goes over HTTP instead of a separate protocol. For this example, the client isn't sending us data, so SSE is a superior choice to websockets. For more details on reasoning through the websocket vs. SSE trade-off analysis, feel free to reference our [FB Live Comments write-up](https://www.hellointerview.com/learn/system-design/problem-breakdowns/fb-live-comments#2-viewers-can-see-all-comments-in-near-real-time-as-they-are-posted).

###### Pattern: Real-time Updates

Real-time stock price updates are a perfect example of the real-time updates pattern. Here we use Server Sent Events (SSE) to establish persistent connections that allow our servers to push live price changes to clients instantly. This approach handles the networking fundamentals of real-time communication while ensuring users see current market prices without the inefficiency of constant polling.

[Learn This Pattern](/learn/system-design/patterns/realtime-updates)

This approach involves re-working our API to instead have a /subscribe POST request, with a body containing a list of symbols we want to subscribe to. Our backend can then setup a SSE connection between the client and a symbol service, and send the client an initial list of symbol prices. This initial list of prices is serviced by a cache that is kept up-to-date by a processor that is listening to the exchange. Additionally, that processor is sending our symbol service those prices so that the symbol service can send that data to clients that have subscribed to price updates for different symbols.

##### Challenges

Adding SSE introduces challenges, mainly involving maintaining a connection between client and server. The load balancer will need to be configured to support "sticky sessions" so that a user and server can maintain a connection to promote data transfer. Additionally, a persistent connection means we have to consider how disconnects / reconnects work. Finally, we need to consider how we route user connections and symbol data. Several of these details are covered later in our deep dive sections, so stay tuned.

SSE

### 2) Users can manage orders for stocks

The second requirement of Robinhood is allowing users to manage orders for stocks. Users should be able to create orders (limit or market), cancel outstanding orders, and list user orders.

Let's consider our options for creating and cancelling orders via the exchange.

### 

Bad Solution: Send Orders Directly to Exchange

##### Approach

This solution involves directly interacting with the exchange to submit orders. Any orders issued by the client are directly submitted to the exchange.

##### Challenges

While this is a "mainline" way to submit orders that cuts out any incurred latency from a backend proxying the exchange, it can lead to large number of exchange clients and concurrent requests, which will be very expensive. Additionally, there's no clear path for the client to check status of an order, outside of polling the exchange or directly listening to trade feeds, both of which aren't viable solutions. Finally, the client is exclusively responsible for tracking orders. This isn't great, as we can't consider the client's storage to be reliable; the user might uninstall the app or the phone hardware might fail.

Let's consider other solutions.

Send Direct

### 

Good Solution: Send Orders to Dispatch Service via Queue

##### Approach

This solution involves sending orders to an order service, which enqueues them for an order dispatch service. This soluton avoids excess exchange clients by proxying the exchange order execution with the order dispatch service. This service can bear the responsibility of efficient exchange communication. This service sends orders to this dispatch service via a queue. The queue prevents the dispatch service from being overloaded, and the queue volume can serve as a metric that the dispatch service could elastically scale off of.

##### Challenges

This approach is on the right track as it proxies the exchange and allows a path for elastic scalability in the face of increased order load (e.g. bursts in trading traffic). However, this approach breaks down when we consider our tight order SLA (under 200ms as a goal).

If we consider moments of high queue load, perhaps during high trading traffic and before the order dispatch service has scaled up, orders might take a while to be dispatched to the exchange, which could violate our goal SLA. In particular sensitive moments of trading, this can be really bad for users. Imagine a user who wants to quickly order stocks or quickly cancel an outstanding order. It would be unacceptable for them to be left waiting for our dispatcher to eventually handle their order, or for our service to start more machines up to scale up given increased queue load.

What other options do we have?

Dispatch Via Queue

### 

Great Solution: Order Gateway

##### Approach

This approach involves sending our orders directly from the order service to an order dispatch gateway. The gateway would enable external internet communication with the exchange via the order service. The gateway would make the requests to the exchange appear as if they were originating from a small set of IPs.

For this approach, our gateway would be an [AWS NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) which would allow for our order services to make requests to the exchange but then appear under a single or small number of IPs ([elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html), in AWS terms).

Given that the gateway is managing outbound requests, this approach relies on the order service that is accepting requests from the client to play a role in actually managing orders. This service will run business logic to manage orders and will scale up / down as necessary given order volume. Given this search is being routed to from clients, we might make the auto-scaling criterion for this service quite sensitive (e.g. auto-scale when average 50% CPU usage is hit across the fleet) or we might over-provision this service to absorb trading spikes.

##### Challenges

This approach requires our order service to do more to manage orders, meaning it will need to be written in a way that is both efficient for client interaction and efficient for exchange interaction (e.g. potentially batching orders together).

Order Gateway

Now that we know how we'll dispatch orders to the exchange, we also must consider how we'll store orders on our side for the purposes of exposing them to the user. The user should be able to GET /orders to see all their outstanding orders, so how might we keep data on our side to service this request?

In order to track orders, we can stand up an order database that is updated when orders are created or cancelled. The order database will be a relational database to promote consistency via ACID guarantees, and will be partitioned on userId (the ID of the user submitting order changes). This will make querying orders of a user fast, as the query will go to a single node.

The order itself will contain information about the order submitted / cancelled. It will also contain data about the state of the order (pending prior to being submitted to the exchange, submitted when it's submitted to the exchange, etc.). Finally, the order will contain an externalOrderId field, which will be populated by the ID that the exchange responds with when the order service submits the order synchronously.

Additionally, to keep these orders up-to-date, we'll need some sort of trade processor tailing the exchange's trades to see if orders maintained on our side get updated.

Of note, this trade processor would be a fleet of machines that are connected to the exchange in some way to receive price updates. The communication interface here doesn't matter too much. For most systems like this, a client of the exchange like Robinhood would setup a webhook endpoint and register that with the exchange, and the exchange would call the webhook whenever it had updates. In the case of webhooks, the trade processor would have a load balancer and a fleet of machines serving that webhook endpoint. For the sake of simplicity, we visualize the trade processor as a single square on our flowchart.

High Level Design

You might be wondering how we concretely reflect updates based on the exchange's trade feed. Stay tuned, as we'll dive into this in one of our deep dive sections.

[Potential Deep Dives](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery#deep-dives-10-minutes)
--------------------------------------------------------------------------------------------------------------------

### 1) How can the system scale up live price updates?

It's worth considering how the system will scale live price updates. If many users are subscribing to price updates from several stocks, the system will need to ensure that price updates are successfully propagated to the user via SSE.

The main problem we need to solve is: **how do we route symbol price updates to the symbol service servers connected to users who care about those symbol updates?**

To enable this functionality in a scalable way, we can leverage [Redis pub/sub](https://redis.io/docs/latest/develop/interact/pubsub/) to publish / subscribe to price updates. Users can subscribe to price updates via the symbol service and the symbol service can ensure it is subscribed to symbol price updates via Redis for all the symbols the users care about. The new system diagram might look like this now:

Pub/Sub for Price Updates

Want to learn more about Redis? Check out our [Redis deep dive](https://www.hellointerview.com/learn/system-design/deep-dives/redis) for in-depth discussion of the different ways Redis can be used practically in system design interviews.

Let's walk through the full workflow for price updates:

1.  A user subscribes via a symbol service server. The server tracks Symbol -> Set<userId> mapping, so it adds an entry for each symbol the user is subscribed to.
    
2.  The symbol service server is managing subscriptions to Redis pub/sub for channels corresponding to each symbol. It ensures that it has an active subscription for each symbol the user is subscribed to. If it lacks a subscription, it subscribes to the channel for the symbol.
    
3.  When a symbol changes price, that price update is processed by the symbol price processor, which publishes that price update to Redis. Each symbol service server that is subscribed to that symbol's price updates get the price update via subscription. They then fan-out and send that price update to all users who care about that symbol's price updates.
    
4.  If a user unsubscribes from symbols or disconnects (detected via some heartbeat mechanism), the symbol service server will go through each symbol they were subscribed to and removes them from the Set<userId>. If the symbol service server no longer has any users subscribed to a symbol, it can unsubscribe from the Redis channel for that symbol.
    

The above would scale as it would enable our users to evenly distribute load across the symbol service. Additionally, it would be self-regulating in managing what price updates are being propagated to symbol service servers.

### 2) How does the system track order updates?

When digging into the order dispatch flow, it's worth clarifying how we'll ensure our order DB is updated as orders are updated on the exchange.

Firstly, it's not clear from our current design how the trade processor might reflect updates in the orders DB, based off just a trade. There's no efficient way for the trade processor to look-up a trade via the externalOrderId to update a row in the Order table, given that the table is partitioned by userId . This necessitates a separate key-value data store mapping externalOrderId to the (orderId, userId) that is corresponds to. For this key-value store, we could use something like [RocksDB](https://rocksdb.org/). This key-value store would be populated by the order service after an order is submitted to the exchange synchronously.

This new key-value store enables the trade processor to quickly determine whether the trade involved an order from our system, and subsequently look up the order (go to shard via userId -> look up Order via orderId) to update the Order's details. The new system diagram might look like this:

Order update deep dive

### 3) How does the system manage order consistency?

Order consistency is extremely important and worth deep-diving into. Order consistency is defined as orders being stored with consistency on our side and also consistently managed on the exchange side. As we'll get into, fault-tolerance is important for maintaining order consistency.

Before we dig in, we firstly can revisit our order storage mechansim. Our order database is going to be a horizontally partitioned relational database (e.g. Postgres). All order updates will happen on a single node (the partition that the order exists on). All order reads will also occur on a single node, as we'll be partitioning by userId.

When an order is created, the system goes through the following workflow:

1.  Store an order in the order database with pending as the status. _It's important that this is stored first because then we have a record of the client could. If we didn't store this first, then the client could create an order on the exchange, the system could fail, and then our system has no way of knowing there's an outstanding order._
    
2.  Submit the order to the exchange. Get back externalOrderId immediately (the order submission is synchronous).
    
3.  Write externalOrderId to our key-value database and update the order in the order database with status as submitted and externalOrderId as the ID received from the DB.
    
4.  Respond to the client that the order was successful.
    

The above workflow seems very reasonable, but it might break down if failures occur at different parts of the process. Let's consider several failures, and ways we can mitigate these failures if they pose a risk to our consistency:

*   **Failure storing order**: If there's a failure storing the order, we can respond with a failure to the client and stop the workflow.
    
*   **Failure submitting order to exchange**: If there's a failure submitting the order to the exchange, we can mark the order as failed and respond to the client.
    
*   **Failure processing order after exchange submission**: If there's an error updating the database after an exchange submission, we might consider having a "clean-up" job that deals with outstanding, pending orders in the database. Most exchange APIs offer a clientOrderId metadata field when submitting an order (see [E\*TRADE example](https://apisb.etrade.com/docs/api/order/api-order-v1.html#/definitions/PreviewOrderRequest)) so the "clean-up" job can asynchronously query the exchange to see if the order went through via this clientOrderId identifier, and do one of two things: 1) record the externalOrderId if the order did go through, or 2) mark the order as failed if the order didn't go through.
    

Now that we've considered the order flow, let's consider the cancel flow. When an order is cancelled, the system goes through the following workflow:

1.  Update order status to pending\_cancel. _We do this first to enable resolving failed cancels later._
    
2.  Submit the order cancellation to the exchange.
    
3.  Record the order cancellation in the database.
    
4.  Respond to the client that the cancellation was successful.
    

Let's walk through different failures to ensure we are safe from inconsistency:

*   **Failure updating status to pending\_cancel**: If there's a failure updating the order status upfront, we respond with a failure to the client and stop the workflow.
    
*   **Failure cancelling order**: If there's a failure cancelling the order via the exchange, we can respond with a failure to the client and rely on a "clean-up" process to scan pending\_cancel orders (ensure they are cancelled).
    
*   **Failure storing cancelled status in DB**: If there's a failure updating the order status in the DB, we can rely on a "clean-up" process to pending\_cancel orders (ensure they are cancelled, or no-op and just update status to cancelled if they have already been cancelled).
    

Based on the above analysis, we have 1) a clear understanding of the order create and cancel workflows, and 2) identified the need for a "clean-up" background process to ensure our order state becomes consistent in the face of failures at different points in our order / cancel workflows. Below is the updated system diagram reflecting our changes:

Order consistency deep dive

### Some additional deep dives you might consider

Robinhood, like most fintech systems, is a complex and interesting application, and it's hard to cover every possible consideration in this guide. Here are a few additional deep dives you might consider:

1.  **Excess price updates**: If a set of stocks has a lot of trades or price updates, how might the system handle the load and avoid overwhelming the client? This might be interesting to cover.
    
2.  **Limiting exchange correspondence**: While we certainly covered ways we'd "proxy" the exchange and avoid excess concurrent connections / clients, it might be worthwhile to dive into other ways the system might limit exchange correspondence, while still scaling and serving the userbase (e.g. perhaps considering batching orders into single requests).
    
3.  **Live order updates**: It might be worthwile to dive into how the system would propagate order updates to the user in real time (e.g. if the user is looking at orders in the app, they see their orders get filled in real time if they're waiting on the exchange).
    
4.  **Historical price / portfolio value data**: In this design, we didn't focus on historical price / portfolio data at all, but some interviewers might consider this a requirement. It's worthwhile to ponder how a system would enable showing historical price data (over different time windows) and historical user portfolio value.
    

[What is Expected at Each Level?](https://www.hellointerview.com/blog/the-system-design-interview-what-is-expected-at-each-level)
---------------------------------------------------------------------------------------------------------------------------------

Ok, that was a lot. You may be thinking, "how much of that is actually required from me in an interview?" Let’s break it down.

### Mid-level

**Breadth vs. Depth:** A mid-level candidate will be mostly focused on breadth (80% vs 20%). You should be able to craft a high-level design that meets the functional requirements you've defined, but many of the components will be abstractions with which you only have surface-level familiarity.

**Probing the Basics:** Your interviewer will spend some time probing the basics to confirm that you know what each component in your system does. For example, if you add an API Gateway, expect that they may ask you what it does and how it works (at a high level). In short, the interviewer is not taking anything for granted with respect to your knowledge.

**Mixture of Driving and Taking the Backseat:** You should drive the early stages of the interview in particular, but the interviewer doesn’t expect that you are able to proactively recognize problems in your design with high precision. Because of this, it’s reasonable that they will take over and drive the later stages of the interview while probing your design.

**The Bar for Robinhood:** For this question, an E4 candidate will have clearly defined the API endpoints and data model, landed on a high-level design that is functional for price updates and ordering. I don't expect candidates to know in-depth information about specific technologies, but the candidate should converge on ideas involving efficient price update propagation and consistent order management. I also expect the candidate to know effective ways of proxying the exchange to avoid excess connections / clients.

### Senior

**Depth of Expertise**: As a senior candidate, expectations shift towards more in-depth knowledge — about 60% breadth and 40% depth. This means you should be able to go into technical details in areas where you have hands-on experience. It's crucial that you demonstrate a deep understanding of key concepts and technologies relevant to the task at hand.

**Advanced System Design**: You should be familiar with advanced system design principles (different technologies, their use-cases, how they fit together). Your ability to navigate these advanced topics with confidence and clarity is key.

**Articulating Architectural Decisions**: You should be able to clearly articulate the pros and cons of different architectural choices, especially how they impact scalability, performance, and maintainability. You justify your decisions and explain the trade-offs involved in your design choices.

**Problem-Solving and Proactivity**: You should demonstrate strong problem-solving skills and a proactive approach. This includes anticipating potential challenges in your designs and suggesting improvements. You need to be adept at identifying and addressing bottlenecks, optimizing performance, and ensuring system reliability.

**The Bar for Robinhood:** For this question, E5 candidates are expected to quickly go through the initial high-level design so that they can spend time discussing, in detail, how to handle real-time price propagation and consistent orders. I expect the candidate to design a reasonable, scalable solution for live prices, and I expect the candidate to design a good order workflow with some mindfulness of consistency / fault tolerance.

### Staff+

**Emphasis on Depth**: As a staff+ candidate, the expectation is a deep dive into the nuances of system design — I'm looking for about 40% breadth and 60% depth in your understanding. This level is all about demonstrating that, while you may not have solved this particular problem before, you have solved enough problems in the real world to be able to confidently design a solution backed by your experience.

You should know which technologies to use, not just in theory but in practice, and be able to draw from your past experiences to explain how they’d be applied to solve specific problems effectively. The interviewer knows you know the small stuff (REST API, data normalization, etc.) so you can breeze through that at a high level so you have time to get into what is interesting.

**High Degree of Proactivity**: At this level, an exceptional degree of proactivity is expected. You should be able to identify and solve issues independently, demonstrating a strong ability to recognize and address the core challenges in system design. This involves not just responding to problems as they arise but anticipating them and implementing preemptive solutions. Your interviewer should intervene only to focus, not to steer.

**Practical Application of Technology**: You should be well-versed in the practical application of various technologies. Your experience should guide the conversation, showing a clear understanding of how different tools and systems can be configured in real-world scenarios to meet specific requirements.

**Complex Problem-Solving and Decision-Making**: Your problem-solving skills should be top-notch. This means not only being able to tackle complex technical challenges but also making informed decisions that consider various factors such as scalability, performance, reliability, and maintenance.

**Advanced System Design and Scalability**: Your approach to system design should be advanced, focusing on scalability and reliability, especially under high load conditions. This includes a thorough understanding of distributed systems, load balancing, caching strategies, and other advanced concepts necessary for building robust, scalable systems.

**The Bar for Robinhood:** For a staff-level candidate, expectations are high regarding the depth and quality of solutions, especially for the complex scenarios discussed earlier. Exceptional candidates delve deeply into each of the topics mentioned above and may even steer the conversation in a different direction, focusing extensively on a topic they find particularly interesting or relevant. They are also expected to possess a solid understanding of the trade-offs between various solutions and to be able to articulate them clearly, treating the interviewer as a peer.

---

## 📋 核心总结 / Core Summary

### 1. Functional Requirements / 功能需求

**Core Requirements / 核心需求:**
1. **Users can see live prices of stocks / 用户可以看到股票的实时价格**
   - 支持查看一个或多个股票的实时价格更新

2. **Users can manage orders for stocks / 用户管理股票订单**
   - 创建订单（market order / limit order）
   - 取消订单
   - 查看订单列表

### 2. Non-Functional Requirements / 非功能需求

**Core Requirements / 核心需求:**
1. **High Consistency / 高一致性**: 订单管理需要高一致性，用户在进行交易时必须看到最新的订单信息
2. **High Scalability / 高可扩展性**: 支持20M日活用户，平均每天5笔交易，数千个交易符号
3. **Low Latency / 低延迟**: 价格更新和订单提交的延迟需低于200ms
4. **Minimize Exchange Connections / 最小化交易所连接**: 减少与外部交易所API的活跃客户端连接数（连接成本高）

### 3. Core Entities / 核心实体

1. **User / 用户**: 系统用户
2. **Symbol / 交易符号**: 股票标识符（如META, AAPL）
3. **Order / 订单**: 用户创建的买入或卖出订单

### 4. API Design / API设计

- `GET /symbol/:name` - 获取股票信息和价格数据
- `POST /order` - 创建订单
  - Request: `{position: "buy", symbol: "META", priceInCents: 52210, numShares: 10}`
  - 使用 `priceInCents` 而非 `price` 避免浮点数精度问题
- `DELETE /order/:id` - 取消订单
- `GET /orders` - 获取用户订单列表（分页）
- `POST /subscribe` - 订阅价格更新（SSE连接）

**安全设计**: 用户信息通过header传递（session token或JWT），而非request body

### 5. High-Level Design / 高层设计

#### 5.1 满足功能需求1: Users can see live prices of stocks

**问题**: 如何高效地向用户推送实时股票价格更新？

**解决方案对比:**

| Solution | Approach | 解决的问题 | Challenges / 挑战 | Tradeoff |
|----------|----------|-----------|------------------|----------|
| **Bad: Polling Exchange Directly** | 客户端直接轮询交易所API | - | • **冗余的交易所调用**: 大量客户端请求相同数据，浪费资源（如5000个客户端同时请求相同价格）<br>• **更新延迟**: 无法满足200ms SLA，除非每200ms轮询一次（不现实） | 简单但不具备可扩展性 |
| **Good: Polling Internal Cache** | 客户端轮询内部缓存（由Symbol Price Processor从交易所更新） | • 减少交易所连接数（通过代理服务）<br>• 降低交易所调用频率 | • **仍有轮询开销**: 即使价格未变，客户端仍在轮询，浪费HTTP流量<br>• **更新延迟**: 轮询间隔决定最差情况下的更新延迟 | 比直接轮询交易所好，但仍不够实时 |
| **Great: Server Sent Events (SSE)** | 使用SSE建立持久连接，服务器推送价格更新到客户端 | • **实时推送**: 价格变化立即推送给客户端<br>• **减少轮询开销**: 仅在价格变化时发送数据<br>• **减少交易所连接**: 通过Symbol Service代理 | • **连接管理**: 需要sticky sessions，处理断线重连<br>• **路由问题**: 需要将价格更新路由到正确的服务器和客户端 | 最优方案，但需要处理连接管理复杂性 |

**最终架构**:
- **Symbol Price Processor**: 监听交易所的价格更新
- **Cache**: 存储最新价格
- **Symbol Service**: 通过SSE向客户端推送价格更新
- **SSE连接**: 建立持久连接，实时推送

**完整Workflow / 完整工作流程**:

**Phase 1: 用户订阅流程 / User Subscription Flow**
1. 用户客户端发起 `POST /subscribe` 请求，包含要订阅的symbol列表（如["META", "AAPL"]）
2. Symbol Service接收请求，建立SSE连接（持久HTTP连接）
3. Symbol Service维护内存映射：`Symbol -> Set<userId>`（跟踪哪些用户订阅了哪些symbol）
4. Symbol Service检查是否已订阅该symbol的Redis channel，如果没有则订阅
5. Symbol Service从Cache获取当前价格，通过SSE发送初始价格数据给用户
6. SSE连接保持打开状态，等待后续价格更新

**Phase 2: 价格更新流程 / Price Update Flow**
1. **交易所发布价格更新** → 交易所trade feed推送价格变化（如META从$500变为$501）
2. **Symbol Price Processor接收** → Processor监听trade feed，接收到价格更新事件
3. **更新Cache** → Processor更新内部Cache中的最新价格（Key: "META", Value: 50100 cents）
4. **发布到Redis** → Processor将价格更新发布到Redis Pub/Sub（Channel: "symbol:META", Message: {symbol: "META", price: 50100, timestamp: ...}）
5. **Symbol Service接收** → 所有订阅了"symbol:META" channel的Symbol Service服务器接收Redis消息
6. **查找订阅用户** → 每个Symbol Service服务器查看内存映射，找到所有订阅了"META"的用户ID
7. **推送给用户** → Symbol Service服务器通过对应的SSE连接，将价格更新推送给所有订阅用户
8. **客户端更新** → 用户客户端接收SSE消息，更新UI显示最新价格

**Phase 3: 连接管理 / Connection Management**
- **心跳机制**: 定期发送keepalive消息，检测连接是否存活
- **断线重连**: 客户端检测到断线时自动重连，重新订阅
- **清理订阅**: 用户取消订阅或断开连接时，Symbol Service移除映射，如果无用户订阅该symbol则取消Redis订阅

**面试叙述要点 / Key Points for Interview**:
- "用户订阅时，我们建立SSE持久连接，并维护symbol到用户的映射"
- "当价格更新时，Processor更新Cache并发布到Redis，订阅的服务器接收后推送给所有相关用户"
- "整个过程是异步的，价格更新在200ms内从交易所到达用户客户端"
- "我们通过Redis Pub/Sub实现了水平扩展，多个Symbol Service服务器可以共享价格更新"

#### 5.2 满足功能需求2: Users can manage orders for stocks

**问题**: 如何高效地管理订单提交和取消？

**解决方案对比:**

| Solution | Approach | 解决的问题 | Challenges / 挑战 | Tradeoff |
|----------|----------|-----------|------------------|----------|
| **Bad: Send Orders Directly to Exchange** | 客户端直接向交易所提交订单 | - | • **大量交易所客户端**: 成本高<br>• **订单状态追踪困难**: 客户端需要自己轮询或监听trade feed<br>• **不可靠**: 客户端存储不可靠（用户可能卸载app或设备故障） | 简单但不可扩展且不可靠 |
| **Good: Send Orders to Dispatch Service via Queue** | 订单服务 → 队列 → 订单分发服务 → 交易所 | • 减少交易所连接（通过代理）<br>• 支持弹性扩展（根据队列长度扩容） | • **延迟问题**: 高负载时队列可能很长，无法满足200ms SLA<br>• **用户体验差**: 用户需要等待队列处理，在敏感交易时刻不可接受 | 可扩展但无法满足低延迟要求 |
| **Great: Order Gateway** | 订单服务 → 订单网关（AWS NAT Gateway）→ 交易所 | • **低延迟**: 直接同步提交到交易所<br>• **统一IP**: 通过NAT Gateway统一出站IP<br>• **可扩展**: 订单服务可以弹性扩展 | • **服务复杂性**: 订单服务需要同时处理客户端交互和交易所交互（可能需要批量处理订单） | 最优方案，平衡了延迟和可扩展性 |

**订单存储**:
- **Order Database**: 关系型数据库（Postgres），按`userId`分区，保证一致性（ACID）
- **字段**: orderId, userId, symbol, status (pending/submitted/cancelled), externalOrderId, price, numShares等
- **Trade Processor**: 监听交易所的trade feed，更新订单状态

### 6. Deep Dives / 深入探讨

#### 6.1 How can the system scale up live price updates? / 如何扩展实时价格更新？

**问题**: 如何将价格更新路由到正确的Symbol Service服务器和订阅用户？

**解决方案对比: Redis Pub/Sub vs Kafka**

| 维度 / Dimension | Redis Pub/Sub | Kafka Pub/Sub |
|------|--------------|---------------|
| **吞吐量 / Throughput** | ⭐⭐ 中等（10k-100k msg/sec） | ⭐⭐⭐ 高（100k-1M+ msg/sec） |
| **延迟 / Latency** | ⭐⭐⭐ 低（< 1ms） | ⭐⭐ 中等（1-10ms） |
| **消息持久化 / Persistence** | ❌ 无（subscriber断开则丢失） | ✅ 有（可配置retention） |
| **消息重放 / Replay** | ❌ 不支持 | ✅ 支持（offset管理） |
| **集群支持 / Clustering** | ⚠️ 有限（Pub/Sub不支持集群） | ✅ 完整支持 |
| **实现复杂度 / Complexity** | ⭐⭐⭐ 简单 | ⭐⭐ 中等 |
| **适用场景 / Use Case** | 低延迟、临时数据、小规模 | 高吞吐、需要持久化、大规模 |

**方案选择 / Solution Selection**:

**Redis Pub/Sub 适用场景**:
- 价格更新频率不是特别高（< 100k updates/sec）
- 允许消息丢失（价格更新很快，丢失一次影响不大）
- 追求最低延迟
- 系统规模中等（数千个symbol，百万级用户）

**Kafka Pub/Sub 适用场景**:
- 高吞吐量需求（> 100k updates/sec，如高频交易）
- 需要消息持久化和重放（如订单状态更新）
- 需要更好的水平扩展
- 系统规模很大（数万个symbol，千万级用户）

**推荐方案 / Recommended Solution**: 
考虑到价格更新需要**fan-out模式**（每个Symbol Service都需要收到相同的更新），**Redis Pub/Sub更合适**：
1. ✅ 天然支持fan-out，实现简单
2. ✅ 低延迟（< 1ms vs Kafka的1-10ms）
3. ✅ 允许消息丢失（价格更新频率高，丢失一次影响不大）
4. ✅ 资源开销小（Kafka需要多个Consumer Group，每个都消费所有消息）
5. ✅ 运维简单

**何时使用Kafka**:
- 需要消息持久化和重放（如订单状态更新）
- 系统规模特别大（> 100k updates/sec），需要Kafka的高吞吐量
- 需要严格的可靠性保证（不能丢失任何消息）

**Approach / 方法（Redis Pub/Sub）**:
- Symbol Price Processor发布价格更新到Redis（Channel: `symbol:{symbol}`）
- Symbol Service服务器订阅相关symbol的Redis channel
- 当用户订阅时，Symbol Service服务器维护`Symbol -> Set<userId>`映射
- 价格更新时，Redis将消息广播给所有订阅的Symbol Service（fan-out），服务器推送给用户

**多对多关系分析 / Many-to-Many Relationship Analysis**:

**当前设计：多对多关系（Fan-out模式）**
- 多个Symbol Service可以订阅同一个symbol
- 一个Symbol Service可以订阅多个symbol
- 每个Symbol Service独立维护自己连接的用户的映射

**为什么不建议Sharding（一个Symbol Service负责特定symbol）？**

| 维度 / Dimension | 多对多（Fan-out） | Sharding（一对一） |
|------|----------------|-------------------|
| **负载均衡 / Load Balancing** | ✅ 多个服务器共享同一symbol的负载 | ❌ 热门symbol可能导致单服务器过载 |
| **容错性 / Fault Tolerance** | ✅ 某服务器挂掉，其他服务器仍可服务 | ❌ 某服务器挂掉，该symbol完全不可用 |
| **扩展性 / Scalability** | ✅ 水平扩展，添加服务器即可 | ⚠️ 需要重新sharding，复杂 |
| **热点问题 / Hotspot** | ✅ 自动分散到多服务器 | ❌ 热门symbol集中到特定服务器 |
| **实现复杂度 / Complexity** | ⭐⭐ 中等（需要管理订阅关系） | ⭐⭐⭐ 高（需要sharding策略和rebalancing） |

**多对多关系是正常的 / Many-to-Many is Normal**:

**重要概念：Fan-out vs Load Sharing**
- **Fan-out模式**: 每个Symbol Service都需要收到相同的价格更新（因为各自维护不同的用户订阅）
- **Load Sharing模式**: 多个Consumer共享处理，每个消息只被一个Consumer处理（不适用于此场景）

**Kafka实现Fan-out的两种方式**:

**方式1：每个Symbol Service使用独立的Consumer Group（推荐）**
- 每个Symbol Service使用不同的Consumer Group ID（如 `symbol-service-1`, `symbol-service-2`）
- 同一个topic的每条消息会被所有Consumer Group消费（fan-out效果）
- 每个Symbol Service独立维护offset，互不影响
- 优势：天然fan-out，简单直观
- 劣势：每个Consumer Group都会消费所有消息（内存和带宽开销）

**方式2：使用Redis Pub/Sub（更简单）**
- Redis Pub/Sub天然支持fan-out，所有订阅的服务器都会收到消息
- 实现更简单，延迟更低
- 不需要管理Consumer Group

**为什么不用同一个Consumer Group？**
- ❌ 同一个Consumer Group中，每条消息只会被一个Consumer处理
- ❌ 如果Symbol Service-1处理了AAPL的价格更新，Symbol Service-2就收不到
- ❌ 但Symbol Service-2也有订阅AAPL的用户，它也需要这个更新！
- ✅ 因此必须使用fan-out模式（每个服务器都收到所有更新）

**Workflow / 工作流程（使用Kafka + 独立Consumer Group）**:
1. 用户通过Symbol Service服务器订阅，服务器添加symbol到映射
2. 每个Symbol Service使用独立的Consumer Group订阅topic（如 `symbol-service-{server-id}`）
3. 价格更新时，Processor发布到Kafka（Key=symbol，自动partitioning）
4. **Kafka将消息分发给所有Consumer Group**（每个Symbol Service都会收到）
5. 每个Symbol Service接收消息后，查找自己维护的映射，推送给订阅该symbol的用户
6. 用户取消订阅或断开连接时，服务器清理映射

**Workflow / 工作流程（使用Redis Pub/Sub - 更推荐）**:
1. 用户通过Symbol Service服务器订阅，服务器添加symbol到映射
2. 如果Symbol Service还未订阅该symbol的Redis channel，则订阅
3. 价格更新时，Processor发布到Redis（Channel: `symbol:{symbol}`）
4. **Redis将消息广播给所有订阅的Symbol Service**（fan-out）
5. 每个Symbol Service接收消息后，查找自己维护的映射，推送给订阅该symbol的用户
6. 用户取消订阅或断开连接时，服务器清理映射，如果无用户订阅则取消Redis订阅

**Redis Pub/Sub vs Kafka（独立Consumer Group）对比**:

| 维度 / Dimension | Redis Pub/Sub | Kafka（多Consumer Group） |
|------|--------------|-------------------------|
| **Fan-out支持 / Fan-out** | ✅ 原生支持 | ✅ 支持（需多个Group） |
| **消息持久化 / Persistence** | ❌ 无 | ✅ 有 |
| **延迟 / Latency** | ⭐⭐⭐ 低（< 1ms） | ⭐⭐ 中等（1-10ms） |
| **资源开销 / Resource** | ⭐⭐⭐ 低 | ⭐ 高（每个Group都消费） |
| **实现复杂度 / Complexity** | ⭐⭐⭐ 简单 | ⭐⭐ 中等 |
| **消息丢失容忍 / Loss Tolerance** | ✅ 可接受（价格更新很快） | ✅ 持久化，更可靠 |

**推荐方案 / Recommended Solution**:
对于价格更新场景，**Redis Pub/Sub更合适**，因为：
1. ✅ 天然fan-out，实现简单
2. ✅ 低延迟（< 1ms vs 1-10ms）
3. ✅ 允许消息丢失（价格更新频率高，丢失一次影响不大）
4. ✅ 资源开销小（Kafka多Group模式会重复消费）
5. ✅ 运维简单

**何时使用Kafka**:
- 需要消息持久化和重放
- 需要更高可靠性（不能丢失消息）
- 系统规模特别大，需要Kafka的其他特性

#### 6.2 How does the system track order updates? / 系统如何追踪订单更新？

**问题**: 如何通过`externalOrderId`快速更新订单状态（Order DB按`userId`分区）？

**解决方案: Key-Value Store (RocksDB)**

**Approach / 方法**:
- 使用Key-Value Store存储`externalOrderId -> (orderId, userId)`映射
- 订单提交到交易所后，Order Service将映射写入K-V Store
- Trade Processor收到trade feed时，通过`externalOrderId`查找`(orderId, userId)`，然后更新Order DB

**优势**:
- 快速查找：O(1)时间复杂度
- 支持高并发写入
- 持久化存储

#### 6.3 How does the system manage order consistency? / 系统如何管理订单一致性？

**问题**: 如何保证订单状态在系统故障时的一致性？

**解决方案: 事务性工作流 + Clean-up Job**

**Order Creation Workflow / 订单创建流程**:
1. 在Order DB中存储订单（status: `pending`）- **必须先存储，确保有记录**
2. 提交订单到交易所（同步），获得`externalOrderId`
3. 写入K-V Store（externalOrderId -> orderId, userId）并更新Order DB（status: `submitted`）
4. 返回成功给客户端

**Failure Handling / 失败处理**:
- **存储失败**: 返回失败给客户端，停止流程
- **提交到交易所失败**: 标记订单为`failed`，返回给客户端
- **提交后处理失败**: Clean-up job处理pending订单，通过`clientOrderId`查询交易所确认状态

**Order Cancellation Workflow / 订单取消流程**:
1. 更新订单状态为`pending_cancel` - **先更新状态，便于后续处理失败**
2. 向交易所提交取消请求
3. 更新Order DB状态为`cancelled`
4. 返回成功给客户端

**Failure Handling / 失败处理**:
- **更新状态失败**: 返回失败，停止流程
- **取消失败**: 返回失败，Clean-up job处理`pending_cancel`订单
- **存储取消状态失败**: Clean-up job处理，确认取消状态

**Clean-up Job / 清理任务**:
- 后台任务扫描`pending`和`pending_cancel`订单
- 通过`clientOrderId`查询交易所确认实际状态
- 更新数据库状态，保证最终一致性

**Tradeoff**:
- **一致性保证**: 通过事务和Clean-up job保证最终一致性
- **复杂性**: 需要处理各种失败场景，增加系统复杂性
- **延迟**: Clean-up job是异步的，可能存在短暂不一致

### 7. Transaction Processing Essentials / 交易处理要点总结

金融系统中的交易处理（订单创建、取消、状态更新）需要严格的一致性保证。以下是关键要点：

#### 7.1 Consistency / 一致性保证

**问题**: 如何保证订单状态在系统故障时的一致性？

**核心原则 / Core Principles**:
1. **先写本地，再写外部 / Write Locally First, Then External**
   - 订单必须先写入本地数据库（status: `pending`），再提交到交易所
   - 如果先提交到交易所后失败，系统无法追踪订单状态

2. **状态机管理 / State Machine Management**
   - 定义清晰的状态转换：`pending` → `submitted` → `filled` / `cancelled`
   - 状态转换必须是原子的，使用数据库事务保证

3. **最终一致性 / Eventual Consistency**
   - 通过Clean-up job保证最终一致性
   - 接受短暂的不一致（如`pending`状态），通过异步任务修复

**实现方式 / Implementation**:
- **数据库事务 / Database Transactions**: 使用ACID事务保证单节点内的原子性
- **分布式事务 / Distributed Transactions**: 避免使用（复杂且性能差），使用补偿机制
- **补偿机制 / Compensation**: Clean-up job定期检查并修复不一致状态

#### 7.2 Clean-up Job / 清理任务

**目的 / Purpose**: 保证最终一致性，处理失败的交易状态

**工作机制 / How It Works**:
1. **扫描异常状态 / Scan Abnormal States**
   - 定期扫描`pending`订单（已创建但未确认提交）
   - 定期扫描`pending_cancel`订单（已请求取消但未确认）

2. **查询交易所 / Query Exchange**
   - 使用`clientOrderId`查询交易所确认订单实际状态
   - 交易所API通常支持通过`clientOrderId`查询订单状态

3. **修复状态 / Fix State**
   - 如果订单已提交：更新为`submitted`，记录`externalOrderId`
   - 如果订单未提交：标记为`failed`
   - 如果订单已取消：更新为`cancelled`
   - 如果订单未取消：重试取消或标记为已取消（如果已filled）

**配置 / Configuration**:
- **执行频率 / Frequency**: 每30秒-5分钟执行一次（根据SLA调整）
- **超时设置 / Timeout**: 超过一定时间（如5分钟）的`pending`订单需要处理
- **重试策略 / Retry Strategy**: 查询失败时使用指数退避重试

#### 7.3 Idempotency / 幂等性

**问题**: 如何防止重复提交导致的重复订单？

**核心概念 / Core Concept**:
- **幂等性 / Idempotency**: 相同的请求执行多次，结果应该相同
- **Idempotency Key**: 客户端提供的唯一标识符，用于去重

**实现方式 / Implementation**:

**1. 客户端提供Idempotency Key / Client-Provided Idempotency Key**
```
POST /order
Headers: {
  Idempotency-Key: "uuid-v4-generated-by-client"
}
Request: {
  position: "buy",
  symbol: "META",
  priceInCents: 52210,
  numShares: 10
}
```

**2. 服务端检查 / Server-Side Check**
- 使用`Idempotency-Key`作为唯一键检查是否已处理
- 如果已处理：返回之前的结果（相同的订单ID和状态）
- 如果未处理：创建新订单，记录`Idempotency-Key`与订单的映射

**3. 存储Idempotency Key / Store Idempotency Key**
- **Redis**: 存储`idempotency-key:{key}` → `{orderId, status, response}`
  - TTL: 24小时（足够长，覆盖客户端重试窗口）
  - 快速查找：O(1)
- **数据库**: 在Order表中添加`idempotency_key`字段（唯一索引）
  - 更持久，但查询稍慢

**最佳实践 / Best Practices**:
- **客户端生成**: 客户端生成UUID v4作为Idempotency Key
- **服务端验证**: 服务端必须验证Key的格式和唯一性
- **响应缓存**: 缓存之前的响应，直接返回（避免重复处理）
- **Key格式**: 使用UUID v4，确保全局唯一

#### 7.4 Double Payment / 重复支付防护

**问题**: 如何防止用户重复提交订单导致重复扣款？

**防护措施 / Protection Measures**:

**1. 幂等性检查 / Idempotency Check**
- 使用Idempotency Key防止重复请求
- 如果请求重复，返回已创建的订单

**2. 状态检查 / State Check**
- 检查用户账户状态（余额、持仓限制等）
- 检查订单状态（避免重复处理同一订单）

**3. 分布式锁 / Distributed Lock**
- 使用Redis分布式锁防止并发重复提交
- Lock Key: `order:user:{userId}:symbol:{symbol}`
- Lock Timeout: 5秒（足够处理一个订单）
- 如果获取锁失败，返回"处理中"错误

**4. 数据库唯一约束 / Database Unique Constraints**
- Order表添加唯一索引：`(userId, idempotency_key)`
- 数据库层面防止重复插入

**5. 余额检查 / Balance Check**
- 下单前检查账户余额
- 使用数据库事务锁定账户记录
- 防止余额不足或重复扣款

#### 7.5 其他关键要点 / Other Key Points

**1. Retry Strategy / 重试策略**

**何时重试 / When to Retry**:
- 网络错误（timeout、connection error）
- 临时性错误（5xx错误码）
- 不重试：4xx错误（客户端错误）、幂等性冲突

**重试策略 / Retry Strategies**:
- **Exponential Backoff / 指数退避**: 1s, 2s, 4s, 8s...
- **Jitter / 抖动**: 添加随机延迟，避免雷群效应
- **Max Retries / 最大重试次数**: 3-5次
- **Dead Letter Queue / 死信队列**: 超过重试次数后放入DLQ

**2. Timeout Handling / 超时处理**

**超时场景 / Timeout Scenarios**:
- 客户端请求超时（30秒）
- 交易所API调用超时（5-10秒）
- 数据库操作超时（5秒）

**处理方式 / Handling**:
- 设置合理的超时时间
- 超时后标记为`pending`，由Clean-up job处理
- 客户端可以查询订单状态（使用Idempotency Key）

**3. Transaction Isolation / 事务隔离**

**隔离级别 / Isolation Levels**:
- **Read Committed / 读已提交**: 默认级别，适合大多数场景
- **Repeatable Read / 可重复读**: 需要时可使用，防止幻读

**注意事项 / Considerations**:
- 避免长时间事务（锁定资源）
- 使用乐观锁（version字段）而非悲观锁（SELECT FOR UPDATE）

**4. Audit Log / 审计日志**

**记录内容 / Log Content**:
- 所有订单操作（创建、取消、状态变更）
- 操作时间戳、操作人、IP地址
- 订单详情（金额、symbol、数量）
- 操作结果（成功/失败、错误信息）

**用途 / Usage**:
- 故障排查
- 合规审计
- 用户争议处理
- 数据分析

**5. Reconciliation / 对账**

**目的 / Purpose**: 定期与交易所对账，确保数据一致性

**实现方式 / Implementation**:
- **每日对账 / Daily Reconciliation**: 每日凌晨对账前一天的所有订单
- **对账内容 / Content**: 订单数量、金额、状态
- **异常处理 / Exception Handling**: 发现不一致时告警并人工处理

**6. Rate Limiting / 限流**

**目的 / Purpose**: 防止恶意请求和系统过载

**实现方式 / Implementation**:
- **用户级别 / Per User**: 每个用户每秒最多10个订单
- **Symbol级别 / Per Symbol**: 热门symbol单独限流
- **全局限流 / Global**: 系统总TPS限制

**算法 / Algorithm**:
- Token Bucket / 令牌桶
- Sliding Window / 滑动窗口
- 分布式限流：使用Redis实现

#### 7.6 面试要点总结 / Interview Key Points

**必须掌握的概念 / Must-Know Concepts**:
1. ✅ **Idempotency Key**: 防止重复请求的核心机制
2. ✅ **Clean-up Job**: 保证最终一致性的关键组件
3. ✅ **状态机管理**: 清晰的状态转换规则
4. ✅ **分布式锁**: 防止并发重复提交
5. ✅ **重试策略**: 指数退避 + 死信队列
6. ✅ **超时处理**: 超时后标记pending，异步处理

**常见问题 / Common Questions**:
- "如何处理订单提交到交易所后服务崩溃？" → Clean-up job查询交易所状态
- "如何防止用户重复点击导致重复订单？" → Idempotency Key + 分布式锁
- "如何保证订单状态一致性？" → 状态机 + 数据库事务 + Clean-up job
- "订单超时怎么办？" → 标记pending，Clean-up job处理

### 8. Key Takeaways / 关键要点

1. **实时数据推送**: 使用SSE优于轮询，提供更好的实时性和资源效率
2. **代理模式**: 通过内部服务代理外部API（交易所），减少连接数和成本
3. **数据一致性**: 金融系统需要高一致性，需要仔细设计工作流和失败处理
4. **可扩展性**: 使用Pub/Sub模式（Redis）支持实时更新的水平扩展
5. **存储设计**: 根据查询模式选择存储（关系型DB按userId分区，K-V Store用于快速查找）
6. **失败恢复**: Clean-up job是保证最终一致性的关键组件
7. **幂等性保证**: Idempotency Key是防止重复请求的核心机制
8. **交易安全**: 分布式锁 + 状态检查 + 幂等性检查多重防护

