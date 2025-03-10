---
layout: post
title: "System design - Uber"
date: 2025-03-10
description: "Decode User backend architecture"
tag: System Design
---

# System Design - Uber

## [Reference](https://www.geeksforgeeks.org/system-design-of-uber-app-uber-system-architecture/)

## Functional Requirements

1.Users should be able to see all the cabs available with minimum price and ETA

2.Users should be able to book a cab for their destination

3.Users should be able to see the location of the driver

4.Users should be able to cancel their ride whenever they want

## Non-Functional Requirement

1.High Availability

2.High Reliability

3.Highly Scalable

4.Low Latency

## Traffic & Storage & bandwidth Estimation

## Challenges Break Down

<img src="https://media.geeksforgeeks.org/wp-content/cdn-uploads/20201120210648/Uber-System-Design-High-Level-Architecture.png">

### Match rider & cab

`Uber service:`

A user can request a ride through the application and within a few minutes, a driver arrives nearby his/her location to take them to their destination.

One of the main tasks of Uber service is to match the rider with cabs.

So we have two services:

`1.Demand Service (Rider)`

```
Demand service receives the request of the cab through a web socket and it tracks the GPS location of the user. It also receives different kinds of requirements such as the number of seats, type of car, or pool car.

Demand gives the location (cell ID) and user requirement to supply and make requests for the cabs.
```

-------------------

`2.Supply Sercice (Driver/Cabs)`

```
In our case, cabs are the supply services and they will be tracked by geolocation (latitude and longitude).

All the active cabs keep on sending the location to the server once every 4 seconds through a web application firewall and load balancer.

The accurate GPS location is sent to the data center through Kafka’s Rest APIs once it passes through the load balancer. Here we use Apache Kafka as the data hub.

Once the latest location is updated by Kafka it slowly passes through the respective worker nodes’ main memory.

Also, a copy of the location (state machine/latest location of cabs) will be sent to the database and to the dispatch optimization to keep the latest location updated.

We also need to track a few more things such as the number of seats, the presence of a car seat for children, the type of vehicle, can a wheelchair be fit, and allocation ( for example, a cab may have four seats but two of those are occupied.) 
```

-------------------

`3.Dispatch Service`

Now we have demand and supply service, we need a service to match them together.

Uber has a **Dispatch system** (Dispatch optimization/DISCO) in its architecture to match supply with demand. This dispatch system uses mobile phones and it takes the responsibility to match the drivers with riders (supply to demand). 

The goals of dispatch service are:

```
1.Reduce extra driving.

2.Minimum waiting time

3.Minimum overall ETA
```

-------------------

`To implement the dispatch service (google S2 or GeoHash)`

Earth has a spherical shape so it’s difficult to do summarization and approximation by using latitude and longitude. To solve this problem Uber uses the **Google S2** library. This library **divides the map data into tiny cells (for example 3km) and gives a unique ID to each cell**. This is an easy way to spread data in the distributed system and store it easily.

S2 library gives coverage for any given shape easily. Suppose you want to figure out all the supplies available within a 3km radius of a city.

```
    1   |   2  |   3
----------------------------
    4   |   5  |   6
----------------------------
    7   |   8  |   9
```

Since each cell has an ID the ID is used as a sharding key. When a location comes in from supply the cell ID for the location is determined. Using the cell ID as a shard key the location of the supply is updated. It is then sent out to a few replicas.
To match riders to drivers or just display cars on a map, DISCO sends a request to geo by supply

**Note that uber also use cell Id for supply cabs partition.**

----------------

`How Dispatch System Match the Riders to Drivers`

We have discussed that DISCO divides the map into tiny cells with a unique ID. This ID is used as a sharding key in DISCO. When supply receives the request from demand the location gets updated using the cell ID as a shard key.

These tiny cells’ responsibilities will be divided into different servers lies in multiple regions (consistent hashing).

For example, we can allocate the responsibility of 12 tiny cells to 6 different servers (2 cells for each server) lying in 6 different regions. 

<img src="https://media.geeksforgeeks.org/wp-content/cdn-uploads/20201120210754/supply-demand-node-distribution-system-design-uber.png">

Supply sends the request to the specific server based on the GPS location data. After that, the system draws the circle and filters out all the nearby cabs which meet the rider’s requirements.

After that, the list of the cab is sent to the ETA to calculate the distance between the rider and the cab, not geographically but by the road system.

The sorted ETA is then sent back to the supply system to offer to a driver.

------------------------

`How To Scale Dispatch System?`

The dispatch system (including supply, demand, and web socket) is built on NodeJS. NodeJS is the asynchronous and event-based framework that allows you to send and receive messages through WebSockets whenever you want.

Uber uses an open-source ringpop to make the application cooperative and scalable for heavy traffic. Ring pop has mainly three parts and it performs the below operation to scale the dispatch system. 

It maintains consistent hashing to assign the work across the workers. It helps in sharding the application in a way that’s scalable and fault-tolerant.

Ringpop uses RPC (Remote Procedure Call) protocol to make calls from one server to another server.

Ringpop also uses a SWIM membership protocol/gossip protocol that allows independent workers to discover each other’s responsibilities. This way each server/node knows the responsibility and the work of other nodes.

Ringpop detects the newly added nodes to the cluster and the node which is removed from the cluster. It distributes the loads evenly when a node is added or removed. 

### Build the Map [references](https://www.uber.com/blog/maps-metrics-computation/)

`Trace coverage: `

Trace coverage spot the missing road segments or incorrect road geometry.

Trace coverage calculation is based on two inputs: map data under testing and historic GPS traces of all Uber rides taken over a certain period of time.

It covers those GPS traces onto the map, comparing and matching them with road segments.

If we find missing road segments (no road is shown) on GPS traces then we take some steps to fix the deficiency. 

-----------------

`Preferred access (pick-up) point accuracy`

Pick-up points are an extremely important metric to the rider experience, especially at large venues such as airports and stadiums. For this metric, we compute the distance of an address or place’s location, as shown by the map pin in Figure 4, below, from all actual pick-up and drop-off points used by drivers. We then set the closest actual location to be the preferred access point for the said location pin. When a rider requests the location indicated by the map pin, the map guides the driver to the preferred access point. We continually compute this metric with the latest actual pick-up and drop-off locations to ensure freshness and accuracy of the suggested preferred access points.

<img src="https://blog.uber-cdn.com/cdn-cgi/image/width=300,quality=80,onerror=redirect,format=auto/wp-content/uploads/2018/07/Figure_4.png">

----------------

`How Does Uber Defines a Map Region?`

Before launching a new operation in a new area, Uber onboarded the new region to the map technology stack. In this map region, we define various subregions labeled with grades A, B, AB, and C. 

**Grade A**: This subregion is responsible to cover the urban centers and commute areas. Around 90% of Uber traffic gets covered in this subregion, so it’s important to build the highest quality map for subregion A. 

**Grade B**: This subregion covers the rural and suburban areas which are less populated and less traveled by Uber customers. 

**Grade AB**: A union of grade A and B subregions. 

**Grade C**: Covers the set of highway corridors connecting various Uber Territories.  

### Estimate Time Arrival

ETA is calculated based on the road system (not geographically) and there are a lot of factors involved in computing the ETA (like heavy traffic or road construction).

When a rider requests a cab from a location the app not only identifies the free/idle cabs but also includes the cabs which are about to finish a ride.

```
It may be possible that one of the cabs which are about to finish the ride is closer to the demand than the cab which is far away from the user. So many Uber cars on the road send GPS locations every 4 seconds, so to predict traffic we can use the driver’s app’s GPS location data. 
```

We can represent the entire road network on a graph to calculate the ETAs. We can use AI-simulated algorithms or simple Dijkstra’s algorithm to find out the best route in this graph.

In that graph, nodes represent intersections (available cabs), and edges represent road segments.

We represent the road segment distance or the traveling time through the edge weight. We also represent and model some additional factors in our graph such as one-way streets, turn costs, turn restrictions, and speed limits. 

Once the data structure is decided we can find the best route using **Dijkstra’s search** algorithm which is one of the best modern routing algorithms today. For faster performance, we also need to use OSRM (Open Source Routing Machine) which is based on contraction hierarchies.

## High level Design

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20231213140933/HLD-uber-app.jpg">

## Data Modeling

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20231215171020/Data-model-design-2.jpg">

### Database

Uber had to consider some of the requirements for the database for a better customer experience. These requirements are… 

    The database should be horizontally scalable. You can linearly add capacity by adding more servers.
    
    It should be able to handle a lot of reads and writes because once every 4-second cabs will be sending the GPS location and that location will be updated in the database.

    The system should never give downtime for any operation. It should be highly available no matter what operation you perform (expanding storage, backup, when new nodes are added, etc).

Earlier Uber was using the RDBMS PostgreSQL database but due to scalability issues uber switched to various databases. Uber uses a NoSQL database (schemaless) built on top of the MySQL database.  

    Redis for both caching and queuing. Some are behind Twemproxy (which provides scalability of the caching layer). Some are behind a custom clustering system.
    
    Uber uses Schemaless (built in-house on top of MySQL), Riak, and Cassandra. Schemaless is for long-term data storage. Riak and Cassandra meet high-availability, low-latency demands.

    MySQL database.

    Uber is building their own distributed column store that’s orchestrating a bunch of MySQL instances.

### Services

Customer Service: This service handles concerns related to customers such as customer information and authentication.

Driver Service: This service handles driver-related concerns such as authentication and driver information.

Payment Service: This service will be responsible for handling payments in our system.

Notification Service: This service will simply send push notifications to the users. It will be discussed in detail separately.

## Deep Dive

### Handle The Data center Failure

Datacenter failure doesn’t happen very often but Uber still maintains a backup data center to run the trip smoothly. This data center includes all the components but Uber never copies the existing data into the backup data center. 

Then how does Uber tackle the data center failure?? 

It actually uses driver phones as a source of trip data to tackle the problem of data center failure. 

When The driver’s phone app communicates with the dispatch system or the API call is happening between them, the dispatch system sends the encrypted state digest (to keep track of the latest information/data) to the driver’s phone app.

Every time this state digest will be received by the driver’s phone app. In case of a data center failure, the backup data center (backup DISCO) doesn’t know anything about the trip so it will ask for the state digest from the driver’s phone app and it will update itself with the state digest information received by the driver’s phone app. Untitled Diagram