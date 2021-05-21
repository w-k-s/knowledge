# Mobile Development Lifecycle

This is my opinion on how mobile development should be done. I will take the example of a Service Booking application (e.g. A mobile app that lets you book a cleaning service for x hours) since that has been the majority of my experience; however, I believe this workflow applies to any kind of commercial or content-based mobile application.

The workflow consists of the following steps:

1. Backend Developer(s) expose an endpoint e.g. `/api/v1/orderHistory` in the order-history-service
2. Mobile Architect(s) expose an endpoint in the Backend-For-Frontend (BFF) to fetch the order History . The BFF could be RESTful, GraphQL e.t.c. This endpoint calls the order-history-service to get the data and format it to suit the frontend requirements.
3. Mobile Architect(s) update the SDK project to add a `getOrderHistory` method that fetches the order history from the BFF. Once the PR is merged into the master branch, the pipeline (having ensured that all unit tests pass and code coerage is >90%) will publish an updated version of the SDK to the company's artifact repository manager (e.g. [`Nexus`](https://www.vogella.com/tutorials/Nexus/article.html),[`Artifactory`](https://jfrog.com/artifactory/),[`Github Package Manager`](https://docs.github.com/en/packages/quickstart))
4. Mobile Architect(s) update the sdk dependency in their dependency management tool of choice (`npm`,`gradle`,`spm`,`cocoapods`). Implement logic to either reload the order history or to retrieve it from the cache.
5. Mobile Frontend Developer(s) design the UI to display the order history as well as to write integration and UI tests.

The remainder of this article will elaborate on the following aspects of this workflow:

1. Advantages / Disadvantages of this workflow
2. Mobile SDK architecture
3. Mobile SDK Pipeline
4. Responsibilities of Mobile Architect vs Mobile Frontend Developer
5. Mobile Team Structure 

## 1. Advantages / Disadvantages of this workflow

| Advantages                                                   | Disadvantages                            |
| ------------------------------------------------------------ | ---------------------------------------- |
| Models and networking logic are separate from ViewModels and Views | Slightly more complicated build pipeline |
| Mobile Architect can ensure good code coverage on SDK implementation. Mobile Frontend can focus on delivering a good UI on a range of devices; have more time to write UI tests |                                          |
| Mobile Developers focus on their strengths: those interested in design focus on frontend. Those interested in architecture focus on architecture |                                          |
| If the organisation offers its mobile SDK to third-party developers, then this approach gives internal developers to  test out the SDK themselves to ensure that it's easy-to-use. |                                          |

## 4. Responsibilities of Mobile Architect vs Mobile Frontend Developer

| Mobile Architect                                             | Mobile Frontend Developer                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| - Commits changes to the BFF and the SDK                     | - Commits changes to the UI                                  |
| - Specializes in: optimising performance, reducing battery consumption, building offline-first experiences and dealing with memory leaks | - Specializes in building  customized components and UIs.  Responsible for delivering a modern, user-friendly experience |

## 5. Mobile Team Structure 

```yaml
Development Team:
  Squads:
  Orders:
- 
```

