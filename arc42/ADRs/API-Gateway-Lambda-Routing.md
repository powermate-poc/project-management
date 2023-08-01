# ADR: How do we route our REST API requests?

## Status

Accepted

## Decision Drivers

- Scope Leads
- Developer Team

## Ressource

<https://aws.amazon.com/de/blogs/compute/best-practices-for-organizing-larger-serverless-applications/>

## Context

The powermate apps functionality, e.g. showing current gas consumption, needs data from the time series database to do so. To have an outside single point of contant, i.e. a facade, incoming requests to powermates internal infrastructure are sent through the AWS API Gateway. The API Gateway enables to handle requests, e.g. querying data time series data from the AWS Timestream database, to be handled with AWS Lambdas.

Having AWS Lambdas as API Gateway downstream opens a discussion how to structure the routing to different ressources e.g. current (e.g. /current) or historic (e.g. /historc)  gas consumption via one single monolithic AWS Lambda that handles routing itself, or to split the functionality with the API Gateway internal routing, having e.g. /current or /historc being split into two separate AWS Lambdas.

## Options

### Monolithic AWS Lambda with internal routing

This approach, though possible, has two major consequences:

- Having the routing in the AWS Lambdas logic, requires to have the AWS Lambda implementing some kind of web framework to or to handle the routing manually
- Given the above, the to deplyoing AWS Lambda would be big in size and responsibility

This approach, is not considered valid and therefore disregarded.

### API Gateway internal Routing with Multiple Downstream AWS Lambdas

This approach, though consequenting more implementation expense in the beginning, has to major "pros":

- The implementation is no monolith
- Having multiple AWS Lambdas handling different ressource requests, will lead to separation of concerns with which maintenance and extensions down the line are facilitated
- Total Deplyoment could take longer due to multiple deplyoments, though
  - can be parallelized
  - routing endpoints can be deployed isolated

## Decision

API Gateway internal Routing with multiple AWS Lambdas
