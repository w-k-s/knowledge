# Preparing for Initial Public Launch

I had the fortune of working on a project before its initial launch to the public. 

This page documents my learnings from that experience. It documents what you need to do from Sprint 0 to the last sprint.


(I'll probably need to organise this in a more sensible way later on)

--- 

Releasing a product to the public market requires a lot of planning.
A public launch that goes well should have the following characteristics:

- Developers, QA, Design and all the various departments within the company have been using the product for ages now and can feel how much more stable and mature the product is now since the early days of development. All that's left now is for the launch-date to arrive.

- The team feels confident about 3rd party integrations because they've been using their services for a while. Edge cases have been identified and have been worked out with the vendor and/or have been handled adeuately.

- Developers, DevOps, QA and Designers feel confident about the product. 
	- Developers and QA feel confidence because end-to-end tests run nightly with full integrations and the tests are passing.
	- Developers and DevOps feel confidence because of the extensive monitoring and alerts. They've proven themselves to be helpful in the many months of development.
	- Designers feel confident because they + external testers have used the product in different devices, languages, screen sizes and everything has given positive feedback of the experience.

- Smooth launch of the product depends on:
	- clear requirements from management
	- an understanding of Minimum Viable Product
	- Organisation skills of team

## Sprint 0

### Deploy all environments, especially production

- All environments should be deployed from day 1.
- A QA Engineer can not mark a task as being done until a feature works as expected on all environments.
- Non-production environments can be scheduled to automatically shut down during non-working hours to save on costs.

### CI/CD pipeline should be up and running

- Developers should follow git flow. 
- Changes to `develop` are pushed to the uat environment
- Changes to `master` are pushed to the prod environment

### End-to-end tests run after deployment in each environment

- The QA Engineers task within a sprint is to write down the end-to-end tests (e.g. Gherkin) that will verify the functionality in all environments.
- End to end tests should verify performance from day 1. I will write more a separate document about this but, in brief, initially every API should complete in 300ms otherwise the test fails. APIs that respond sooner should always be expected to respond at that time otherwise the test fails. APIs that take longer than 300ms should be refactored to improve performance if possible that very sprint or the next.

### End-to-end tests take security into account

- From day 1, a policy of security conciousness should be adopted
- End-to-end tests should try bad input, sql injections, test rate limiting etc

### 3rd Party integrations 

- Implemented as early as possible
- Tested regularly as part of nightly end-to-end tests so that edge cases can be discovered and handled elegantly.

### Dog-feeding

- The team should use the app from day 1.
- Get the app deployed and get your team to use it.
- This is usually easy if the app is say a game or a shopping app but not so much if it's a government or a banking app. Let users have a bit of fun with it e.g if its a banking or government app, let your team add some easter eggs or add some funny data to keep it interesting. Of course, there need to be rules.