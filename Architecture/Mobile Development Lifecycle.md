# Mobile Development Lifecycle

This is my opinion on how mobile development should be done. I will take the example of a Service Booking application (e.g. A mobile app that lets you book.a cleaning service for x hours) since that has been the majority of my experience; however, I believe this workflow applies to any kind of commercial or content-based mobile application.

The workflow consists of the following steps:

1. Backend Developer(s) expose an endpoint e.g. `/api/v1/orderHistory` in the order-history-service
2. Mobile Architect(s) expose an endpoint in the Backend-For-Frontend (BFF) to fetch the order History . The BFF could be RESTful, GraphQL e.t.c. This endpoint calls the order-history-service to get the data and format it to suit the frontend requirements.
3. Mobile Architect(s) update the SDK project to add a `getOrderHistory` method that fetches the order history. Once the PR is merged into the master branch (having ensured that all unit tests pass and code coerage is >90%), the pipeline will publish an updated version of the SDK to the company's artifact repository manager ([`Nexus`](https://www.vogella.com/tutorials/Nexus/article.html),[`Artifactory`](https://jfrog.com/artifactory/),[`Github Package Manager`](https://docs.github.com/en/packages/quickstart))
4. Mobile Frontend Developer(s) update the sdk dependency in their dependency management tool of choice (`npm`,`gradle`,`spm`,`cocoapods`) and use the new `getOrderHistory` method to display the order history on the User Interface.

The remainder of this article will elaborate on the following aspects of this workflow:

1. Mobile SDK architecture
2. Mobile SDK Pipeline
3. Responsibilities of Mobile Architect vs Mobile Frontend Developer
4. Mobile Team Structure 



