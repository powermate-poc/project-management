# ADR: How do we structure, document and deploy our API?

## Status

Accepted

## Decision Drivers

- Scope Leads
- Developer Team

## Context

The frontend-facing backend, namely the API, has to be structured, implemented, configured, documented and deployed. This ADR specifies and discusses decisions on the following topics:

- API configuration
- API documentation
- API deployment

## Options

### OpenAPI Specification + AWS API Gateway Integration

One possible solution to structure, configure and document an API is to use an [OpenAPI Specification](https://swagger.io/specification/). This solution provides the benefit of having one single source of truth that everyone can reference in a standardized way. Specifying ones API in an OpenAPI Specification also allows for automatic generation of documentation and client libraries. The OpenAPI Specification can also be integrated with AWS API Gateway, which allows for easy deployment and management of the API.

### Custom API Documentation

Another option is to write custom API documentation. This option provides the benefit of having full control over the documentation and the ability to write documentation in a way that is easy to understand for the target audience. However, this option requires more effort and is more error-prone than using an OpenAPI Specification.

## Decision: OpenAPI Specification

The team has decided to use an OpenAPI Specification to structure, configure, document and deploy the frontend-facing backend API. This decision is based on the following benefits:

- Having a single source of truth of the API
- Automatic generation of documentation, namely a Swagger UI, which is a web-based UI that allows for easy documentation/testing of the API
- One main benefit is the integration with the AWS API Gateway, allowing us to directly deploy the API as specified in the OpenAPI Specification. The AWS API Gateway integration also allows to specify which AWS Lambda serves which endpoint and having these associations directly deployed too and documented too.
- Automatic generation client libraries (more future-proofing, as this is not currently used)

## Consequences

Due to the benefits already being mentioned, this section will focus on neutral or negative consequences.

1. The deployment of the API is now coupled to the AWS API Gateway and AWS Lambdas.
2. The OpenAPI Specification has to be written and maintained specifically in addition to the code.
3. Giving the OpenAPI Specification to the *frontend scope team* allows them to easily test the API and generate client libraries. However, this also means that the *frontend scope team* has to be aware of the OpenAPI Specification and how to use it.
