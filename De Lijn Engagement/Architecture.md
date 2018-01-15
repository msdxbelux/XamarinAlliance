# Architecture

## TODO

Partitions

## Use case 1: when will my bus arrive?

![](img/uc1.jpg)

### Options

![](img/001.png)

### Enhancements to the existing solution

Here are the few enhancements that were done to the existing solution to make it scale at a reasonable cost.

**Reduce the size of the intermediate messages from ~30 KB to ~1KB.** The whole trip was retrieved from DocumentDb at the beginning of the data flow and passed along from function to function thru event hubs. Instead, a smaller amount of data will be retrieved and used from function to function. At the end, the whole trip will be retrieved, updated and sent back to the database.

**Send logs to a table API instead of a Document API**. Updating a document will require to download the whole document, modify it and sent the data back to Cosmos DB. Table API works at the row level so it's easy to insert new rows as new log messages come in. 

**reduce the number of functions**. Some couple of functions could be grouped as 1 function. This lower the number of event hubs needed. For instance, `provide realtime` and `handle status` can be merged into one function.

**make sure partitions are used for all data storages**. This is how scale out happens. IOT Hub, Event hubs, Cosmos DB, Azure storage table all have partition keys.



Here are details about some of those tasks

#### Table API

//TODO: explain Azure Table storage SDK. Vn+1 only can understand Cosmos DB connection string. Azure functions currently uses Vn.

#### Partitions and partition keys

//TODO: study how partitions and batching in Azure functions are related. Based on that, choose good partition keys for all data stores.

### Cost

Cost is important in a cloud architecture. A spreadsheet was put together to see what costs could be based on scale.

Here is a screen shot of this spread sheet: 

![](img/price-estimation.png)

When fully operational, the solution will receive 4 msg/sec from 3500 buses, active for 18 hours/day. The current version of the architecture includes 1 IoT Hub, 6 Azure Functions, 5 Event Hubs, Cosmos DB. In order to provide a good estimation, each component has been analyzed considering 15.120.000 msg per day.

### IoT Hub
Considering the number of messages per day, there are 2 possible solution for IoT Hub: S1 with 38 units or S2 with 3 units. However, since IoT Hub applies limitation throttles in order to avoid abuse, the best choice with 234 msg/sec is S2 with 3units, which allows 120 msg/sec/unit.

### Azure Functions
At the moment, all the Azure Functions are based on Consumption Plan, so the billing is based on number of executions, execution time and memory usage. Of course we actually have different values for each Function, but we estimated common values for execution time and memory usage, based on the first tests of the application. With the latest adjustment the solution is more performant, so the estimated costs we now have for Functions should be slightly higher than the final ones.

### Event Hubs
In this scenario, one of the main constraints for Event Hubs is the maximum load per throughput unit (1 MB/s ingress, 2 MB/s egress), since we have 3 Event Hubs running in parallel that need to handle a 30 KB message that contains the Trip. We have 234 msg/sec and 6 Function-Event Hub layers and we must keep the load under 1MB/s, so, in order to avoid latency, we should have 42 throughput units. 
Actually we can be more accurate in the estimation, since only 3 out of 5 EH actually need to manage the 30KB message. It means that the Event Hubs can be splitted into different namespaces with the optimal number of throuput units, to reduce costs. Another keypoint is the size of the message: reducing it from 30KB to 1KB the number of necessary throughput units goes down to 2, with a significant cost decrease. The customer is currently working on the message size.

### Service Bus 
This is an option that has been evaluated during the hackfest. Standard or Premium Tiers have been taken into account, in order to use topics, but the Premium Tier guarantees predictable performances, so it would be the best choice.
However, considering the message size ("big" for Event Hubs, but "small" for Service Bus) and the amount of messages per day, the monthly cost using Service Bus Topics is lower than using 5 Event Hubs. 
This seems to be a good choice for the solution, but if the number of messages increases significantly than a refactor will be necessary. 

### Cosmos DB
At the moment, Cosmos DB is used to store Trips, TripStops, Block, Audit Log. The cost we calculated is based on the current data, which is not supposed to increase (because it is periodically moved to a Blob Storage). 
During the latest test, the number of RUs has been increased up to 4000 and performance are good, but this is a key point in terms of performance so it will be taken into account when a higher number of buses will be tested.

## Use case 2: Buses capacity planning

A quick chat happened about a future use case: counting people in a bus. 

The main idea is to count people with 3D cameras on a few buses. On those buses and other buses, there are also some WIFI counting for devices entering, leaving and being in the bus. There is also swipes counting (mobib registrations).

[![](img/mobib.jpg)](https://www.delijn.be/nl/vervoerbewijzen/mobib-kaart/mobib.html)

A machine learning regression would learn from the 3D Cameras to predict the actual number of passengers from WIFI counting and mobib registrations counting. 

`nb_passengers = f(WIFI_in, WIFI_out, WIFI_on, mobib_swipes, ...)`

`...` additional features could include the same values for the last 3 stops, or the sum of all WIFI_in as well as WIFI_out since the bus was empty, demographics, weather, etc.

![](img/uc2-001.png)

During a quick discussion, we talked about IOT Hub / Event Hubs, Stream Analytics and Azure ML to implement such a solution, based on machine learning written in R.

![](img/uc2-002.png)

The discussion is to be continued. 