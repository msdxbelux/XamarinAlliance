# ADS / Hackfest report

## Introduction

On December 6th and 7th part of CSE team spent 2 days in a hackfest with De Lijn company, in order to provide technical support on their IoT solution. The goal of the project is to manage messages coming from buses, analyze, store and provide near real time information to customers about any delay of the buses. 

We jumped in late in the project, the customer had already implemented an architecture using Azure services, but they were facing performance issues. We planned 2 days in-person hackfest, followed by 2 more days in the next week working remotely. The main goal was to understand the reasons of latency and work on scalability, because the customer had an important milestone on December 15th to review the performance of the solution on significant load (250 buses). 

Even though it started as a hackfest, we actually had deep discussions around the architecture, providing some suggestions aimed to improve performances, making some tests and significant changes.


## De Lijn

De Lijn is a company run by the Flemish government in Belgium to provide public transportation with more than 3500 buses and 359 trams. It was founded in 1991 after the public transportation companies of Antwerp and Ghent fused with the Flemish part of the NMVB (Nationale Maatschappij van Buurtspoorwegen, or the "National Company of Neighborhood Railways").

The company has a partial decentralised organisation. In each Flemish province a provincial department or entity is responsible for the day-to-day service and contact with the stakeholders & passengers. The headquarters in Mechelen (Central Services) coordinates and supports the preparation of the entities. The headquarters also outlines the overal policy and directs the centrally organised activities.

![](img/abus.jpg)

## Venue

De Lijn headquarter is located in Mechelen, and that's where developers are working on this project. Mechelen is a small and nice city, De Lijn office is not far from the center, easy to reach by train from Brussells airport. In the hall you can sit on bus seats and the office is equipped with advanced technologies. 

![](img/mechelen.jpg) 

![](img/delijn-reception.jpg)

## Use case

The goal of this project is to allow people waiting for the bus to use an application and have detailed information about the actual delay. To do so, several messages are sent from the buses to the cloud, to be analyzed, stored and to provide in the end a near real time response to the user.  
This project was born to replace an application already available for users, which has a significant delay (up to 5 minutes), so performance is one of the most important aspect for De Lijn.

![](img/uc1.jpg)

As mentioned before, we jumped in when the project was very close to the stress test. They created an architecture using Azure Functions as microservices, which means that they had a long pipe of stages "Function - Evant Hub" and the message had to pass thru 8 of these stages to be completely analyzed. Besides that, at some point they retrieved a 30KB message from CosmosDB and passed this message thru the stages, having a high load for the Event Hubs.

![](img/architecture.png)


## What we achieved

Starting from the architecture above, we provided some guidance to improve performances. We also analzed different architectures, in terms of scalability, cost, complexity. So we actually did ADS and hackfest at the same time, but one of the main goal at that moment was to find a solution good enough in terms of performance, but also easy enough to be implemented in very short time, before the stress test they planned for December 15th (1 week after the hackfest). 

![](img/001.png)

During the 2 days, we alternated discussion, changes and tests, in order to find the better option.
Here's a list of the main enhancements from the initial solution:

- Reduce the message size from 30KB to 1KB. This is useful both in terms of performance and cost. 

- Send logs to a Table API, so that they could avoid to open a huge document and update it for every message

- Aggregate functions whenever possible, to reduce the number of stages "Funcion-Event Hub". The best option in this case could be to have only one big funcion.

- Use partitions for all data stores

- Read messages in batch from Event Hubs to the Functions. We focused on this aspect during the second part of the hackfest, working remotely. Some very simple tests clearly showed the improvements related to a batch approach.

![](img/batch.png)


These changes had an important impact on the performace of the whole solution. 


//TODO: here it would be interesting to add their latest performance data, if possible. 



## Product feedback

We faced some issues during the hackfest, did some research and maths to provide detailed information to the customer. Here's a list of feedback we can share:

- [Azure functions: how customer thought they could implement Azure functions](https://microsoft.sharepoint.com/teams/CSETechFeedback/Lists/CSE%20Feedback%20Form/DispForm.aspx?ID=225&e=5a1bc27532e14d50947b6978c6ae133e)
- [Azure Cosmos DB documentation may not be consistent between Cosmos DB the engine and its different API subsets](https://microsoft.sharepoint.com/teams/CSETechFeedback/Lists/CSE%20Feedback%20Form/DispForm.aspx?ID=222&e=92d2d90bce2f4e96a7686d4c5a7693fd)
- [Service Bus: Hard to find documentation on scalability and performance targets](https://microsoft.sharepoint.com/teams/CSETechFeedback/Lists/CSE%20Feedback%20Form/DispForm.aspx?ID=223&e=52dc91051b4b42188e831a8be66b9ddf). We evaluated service bus in terms of cost and performance, compared to Event Hubs. Due to the number and the size of the messaged, it could be a better option for this solution, but we didn't find clear data on its scalability.
- [Azure functions: customer would like to move to more recent events from event hubs](https://microsoft.sharepoint.com/teams/CSETechFeedback/Lists/CSE%20Feedback%20Form/DispForm.aspx?ID=224&e=b675c44328c144e687fba29ccad3fb6b). De Lijn would like to move the cursor to event hubs partitions to the end of it when the functions can't keep up with the load. 

## People

We had 4 CSE people present at De Lijn's venue for the hackfest and 4 people on the customer side:

- Benjamin Guinebertière - Microsoft
- Erica Barone - Microsoft
- Jan Tielens - Microsoft
- Tim Park - Microsoft
- Joachim Knoops - De Lijn
- Peter Collette - De Lijn
- Robrecht De Langhe - De Lijn
- Danny - De Lijn

We also had important and useful remote support from local TSPs:

- Karim Vaes Microsoft
- Mathieu Van den Mooter Microsoft


![](img/people1.jpg)