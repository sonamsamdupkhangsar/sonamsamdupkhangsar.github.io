# OpenAPI Contracts
Once you have the OpenAPI yaml file that describes the endpoint, requests and the expected responses there will be a good want for verifying both consumer and the provider implementations are compliant with the openapi.yaml spec or any thing you have both agreed on.  For this there is [Pact Broker](https://docs.pact.io/pact_broker) that can record the consumer interaction for a given service and the provider can verify that it can meet the consumer expectations.

Pact Broker is a service that will store consumer contract or "pacts" on the broker.  It can be used by the provider to validate to ensure compliance with the consumer behavior. A service consumer of a Rest api can create pacts locally and push them to the pact broker.  Once the pact is registered that describes the behavior a provider of that service can run their integration test locally or in a build pipeline for service validation.
 





