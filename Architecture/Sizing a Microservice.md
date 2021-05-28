# Sizing a Microservice

- One of the challenging problems about microservices is how do you divide your application into microservices and what is the responsibility of each service?
- There are plenty of articles written about how to do this. Normally, they suggest that a service should contain a bounded context. 
- From my experience, this has never been good advice. Instead, I would say:

	1. If you have even the slightest bit of doubt about whether the bounded contexts are inter-related or co-dependent, then don't split these bounded contexts into separate micro services just yet. 

        Put them together in the same service to keep things simple, but architect the joined service in a way that you'll easily be able to split the two bounded-contexts into services if and when the need arises i.e. put the two bounded-contexts into separate modules that are imported by the application module. The modules should not depend on each other and they should never directly communicate with each other, only through a programmatic interface.

	    If you're able to do so (and you see benefits in doing so, see [when to split](#when-to-split-bounded-contexts-in-to-multiple-services)), then split the bounded-contexts into services. 

	    If you're unable to do so, then two bounded-contexts are probably too dependent of one another to belong in separate bounded-contexts. This will be apparent when:
		    
		- the two modules that you've made for each bounded-context have a lot of domain models in common.
		- you'll find that the two modules have complex interactions with each other that would be greatly simplified if the bounded contexts were in the same module.

	1. If the bounded contexts are hardly dependant of each other at all, then you can probably split the bounded-contexts into services e.g. authentication and notification can be separate services.

## My experience

- I worked for a team that developed a booking system for a terminal.

### Requirements

1. A message declares all the cargo that will arrive at the terminal. Application shall save list of cargo details.
	
2. Truckers shall be able to login to the system, select the cargo that they will pick up and book a time slot in which they will arrive at the terminal.
	
3. The application shall save the booking, and also notify the third-party terminal gate system.
	
4. The application shall allow the truckers to edit their booking (i.e. change the booking time and/or the cargo)
	
5. The third-party gate system shall be the source of truth of bookings.

The tricky part was that while the application was supposed to allow truckers to edit their booking, the terminal gate system did not support editing; a booking could either be created or deleted.

To further complicate matters, the gate system required that a bespoke API call be made for each piece of cargo in a booking. For example, If Trucker Joe wants to pick up a piano and chair, then the gate system required that the following API calls be made:

- create an appointment for Trucker Joe
- add a piano to Joe's appointment
- add a chair to Joe's appointment

A booking can only deleted when all of its cargo is removed. In other words, to cancel Trucker Joe's booking with the gate system, the following API calls need to be made:

- remove the chair from Joe's appointment
- remove the piano from Joe's appointment
- cancel Trucker Joe's appointment

If the call to the create a trucker appointment fails due to a timeout, a call needs to be made to check if the appointment did in fact get created. If the appointment was created, continue with the adding each piece of cargo to the appointment.

If adding a piece of cargo fails when creating an appointment, try the call again with exponential back off until a maximum of three tries and then send an email to the support team.

If removing a piece of cargo fails when cancelling an appointment, try the call again with exponential back off until a maximum of three tries and then send an email to the support team.

### Implementation

In our naivety, we decided that this booking system required 4 microservices. (Why? because we wanted to implement microservices properly and that meant services should be small and only do one thing): 
	
- **Cargo Service**: Responsible for handling cargo messages and saving the cargo details to a database.
- **Booking Service**: The service that handles the booking requests initiated by the trucker. This service calls out to the trucker appointment service and the cargo appointment service.
- **Trucker Appointment Service**: Interfaces with the gate system to create an appointment for the trucker.
- **Cargo Appointment Service**: Interfaces with the gate system to create an appointment for each piece of cargo.

It should have been obvious, given the way that the gate system works, that the trucker appointment service and cargo appointment service would have VERY complex interactions (especially in error scenarios). We only made things more complicated by splitting them into services. 

In fact, we made them more complicated by using an event-driven architecture.

### Lessons Learnt

With more experience, this is the architecture I would have gone with if I were asked to redo it:

1. Only build 2 services:

	- **Cargo Service**: Responsible for handling cargo messages and saving the cargo details to a database.
	- **Booking Service**: The service that handles the booking requests initiated by the trucker. The booking service has a programmatic interface to represent a gate system. The service imports a module that exposes an easy to use CR**U**D API for the 3rd party gate system. The programmatic interface is implemented using this module.

2. The gate system client module uses synchronous API calls rather than asynchronous message queues to communicate with the gate system. This makes the error scenarios much simpler to handle and to debug. 

	There was never actually a need to design the system to be event-driven. 
	We had built an event-driven architecture in the hopes of building a highly available system. But the system did not need to be highly-available - it needed to be consistent and correct. 

	It was much more important to build something simple that produced consistent output, than to try to achieve consistency in a needlessly complex event-driven distributed system.

### When To split Bounded Contexts into Multiple Services

So now let's say you have a single service that holds multiple bounded-contexts. You've discovered that the two contexts, do not have complex and frequent interactions with each other and they could quite comfortably live in separate microservices. The question is should you split them?

The answer to this question should be driven by data. Specifically, the data you need to consider are:

**Requests per second**: Is the combined service getting too many requests? This needs to be considered in conjunction with CPU Usage.

**CPU Usage**: Is having two bounded-contexts together placing a heavy load on the system. Does one of the bounded-contexts need to do more CPU-intensive work than the other?

**Failed Requests**: As a result of heavy load or high CPU usage, are many of the requests failing with errors like request timeout? 

If the answer to these questions is yes, then you have three options:

1. Horizontal Scaling: run the combined service on an additional node to balance the load better
2. Vertical Scaling: run the combined service on a node that has more memory and CPUs.
3. Split the two bounded-contexts into separate services

The correct choice needs to be determined on the basis of cost.
It obviously costs more to add an extra node to your cloud infrastructur (or enhance the existing ones), but creating and maintaining an extra service is more challenging than having them together: implementation and debugging both become more difficult, you'll need to set up devOps around the new service and you might even need to hire more developers. 

A lot of businesses don't mind simply throwing an extra server at the problem. 