# Test 3rd Party Libraries or APIs

When your microservice interacts with a 3rd party library or API, it's important to write tests around the library or API for the following reasons:

1. To discover any edge-cases that are not handled by the API/Library

   - e.g. You might use a library to generate PDF documents
   - However a business requirement might be that the PDF documents should support Arabic (which is written from right to left)
   - Writing a test will help you discover these limitations early

2. The list of tests will also provide a summary of functions that you need from the library

   - So if in the future you need to switch to a new library, you know what to look for.

3. You can immediately check is a newer version of the library/API has any breaking changes.

## How to write tests for 3rd Party Libaries

- You should not directly use 3rd party library codes and classes in your own application.
- Instead you should use create an interface and use the library to implement that interface.
- Use the interface in your application's code and use the library in the impleemntation.
- This enables you to easily swap out the implementation of the interface to use a different 3rd party library.

- Write tests for the implementation of the interface.

## How to write tests for 3rd Party APIs

- Do not directly call the 3rd Party API from your micrservice.
- Instead, create a gateway service that interacts with the 3rd party API
- Other services in your microservices cluster should communicate via the gateway
- Your gateway microservice should have integration tests that make calls to the 3rd party APIs sandbox.
- These integrations test run on a schedule in the CI/CD pipeline.
- This has the disadvantage of increased latency so i need to think about this one but i'm just mentioning it because this is a brain dump.
