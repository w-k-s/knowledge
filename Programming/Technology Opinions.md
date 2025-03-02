# Technology Opinions

## API Development Frameworks

### Go

- I tried development without using a web framework, relying primarily on Gorilla Mux.
- The only issue I found was the ability to generate an openapi spec from the code.

## Authentication

### FusionAuth

- Really intuitive and well-documented. 
- Wish client credentials grant was part of the free version.

### Keycloak

- Pros: Full Featured and Free.
- Cons: Not well documented (though there is [this](https://wjw465150.gitbooks.io/keycloak-documentation/content/index.html) gitbook). I believe RedHat no longer supports it.

I know a lot of companies use it in production, but I'm not sure I could professionally recommend using Keycloak. I'd probably go with FusionAuth.


## (Headless) CMS

### Wordpress

- Never again! 

### Strapi

- Open Source, I believe it can be self-hosted in a docker image.
- Admittedly, I haven't worked with it directly myself but I was very impressed with the demo. Seemed to be really easy to set up.

### DecapCMS

- I haven't worked with DecapCMS myself but from what I understand:
	- It's geared towards server side rendering.
	- Content Writers write the content and commit to git.
	- Content is pulled by the website at the time of rendering.

## Databases

### PostgreSQL

- My go-to database. Is there anything it can't do?

### DynamoDB

- The fact that you need to anticipate your use-cases before-hand and model accordingly can not be under-stated.
- Consequently, I'm always a little weary about using DynamoDB. I think it works best for hot-data that will eventually be archived to a data warehouse.

## IaC

### ~Terraform~ OpenTofu 

- I love it. But it's not Infrastructure as Code, so I want to give CDK or Pulumi a try.