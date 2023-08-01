# ADR: How do we deploy our Lambdas?

## Status

Accepted

## Decision Drivers

- Scope Leads
- Developer Team

## Context

In powermate business logic, serverless functions, i.e. AWS Lambdas, handle requests and events to either query databases or write into them. The conflict this ADR addresses is how the function code of our Lambdas is deployed alongside the deployment of the Lambda infrastructure itself as code via Terraform.

## Options

### Manual Deployment

Though possible, this is approach is disregarded for obvious reasons. Manual deployment is not feasible for powermate.

### Serverless

The [Serverless Framework](https://www.serverless.com/), as all-in-one solution for AWS Lambdas, would consequent some kind of vendor lock-in and functions separate from our terraform logic. Hence, this alternative is disregarded.

### AWS SAM

The [AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) is a toolkit improving building and running serverless applications on AWS. SAM would require to built our Lambda applications via their own intern SAM ecosystem. This approach would infringe on freedom of development decision choices.

### S3 Buckets

Terraform has the option to deploy AWS Lambdas with directly fetching the function code from a AWS S3 storage bucket.

**Pros:**:

- Single Point of Truth where our Lambda function code is
- Easy rollbacks possible

**Cons:**:

- Workflow if function code changes is suboptimal:
  - CI/CD pipeline would upload new function code to S3
  - S3 upload event would trigger a Lambda function that uploads the new function code to the destination lambda
- Additional infrastructure and logic necessary

### Zip-Uploads

A different approach would entail that Terraform deploys AWS Lambdas with initial dummy code. This dummy code is then replaced with the lambdas function code via project specific pipelines that would build the function code on change and renew the corresponding AWS Lambda.

**Pros**:

- Split between infrastructure and business logic
- Distinct deployment steps for either infrastructure or logic
- Easy project-specific setup to deploy function code

**Cons**:

- No immediate and easy rollback possible
- When uploading the function code as .zip, it has to be assured that the AWS Lambda the function code is deployed to exists

## Decision

Zip-Uploads
