# Testing Ruby Microservices - 2023-06-24 11:43:14.623420424 -0300 -03 m=+0.030232228

tags: #talks #testing

## Testing types

- Unit: tests every single piece of code, should probably have 100%
coverage on the specific file, but lacks integration with real 
dependencies. Generally, should be really fast but not trustworthy

- In-app Integration Tests: tests the whole service, avoiding mocks
and stubs, but still using in-memory dependencies. Should be fast
and trustworthy, but not as fast as unit tests

- Full-Blown Integration Tests: Tests multiples services, including
real dependencies, using integration with external partners. Should
and could be used even in a production environment, 
with real data. They are expensive and slow, but trustworthy, have 
problems when the data changes and are hard to maintain. 

- Contract Tests: Ensure the contract between two services is 
maintained. HTTP contract tests are the most common, but there are
others. They are fast, trustworthy and easy to maintain, but
they are not as complete as full-blown integration tests.

## Possible outcomes

* E2E tests for all api endpoints and various scenarios
* Unit tests with a whole lotta better mocks 
* Mocking at the outer integration level (repository, http, etc)
* Pact Specs run and handled separately
* Running synthetic transactions for major use cases
