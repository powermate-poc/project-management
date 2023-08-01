# ADR: Authorization And Authentication

## Status

Accepted

## Decision Drivers

We want to grant each of our users access to his own data and make sure other user's cannot.
Not only is this important from a data privacy perspective, also from a UX point of view only this makes sense.
Therefore we need to decide on how we want to setup our aws infrastructure to provide authentication and authorization for our application users.

## Context

First of all it's important to know the difference between authentication (authn) and authorization (authz).
Authn means making sure it is really you, while authz means to evaluate what kind of permissions the already authenticated user has.
Like in our case our user should have the permission to fetch only his own data and should be forbidden to fetch any data of other users.

**AWS cognito user pools** could help us for the authn part. Eventually we could add an external identity provider like Google/Apple/etc. later. We then could use **AWS cognito identity pools** or another lambda for the authz part. Here are some pros and cons i copied from [here](https://github.com/vaquarkhan/vaquarkhan/wiki/AWS-Custom-Lambda-authorizer---authentication-for-amazon-api-gateway-for-microservice)

## Options

### AWS Cognito

#### :heavy_plus_sign: Pros

AWS SDK handles everything for you and you cannot make much mistake in your authentication process. Fine grained access control for AWS resources via IAM. An extra lambda function in front of every API is not required for authentication.

#### :heavy_minus_sign: Cons

Need to use AWS SDK specifically on client side. Programmers have to add this into their toolchain and make use if it during development. Adds extra complexity. Fine grained access control for resources is not really required since the only access that is required is for API gateway.

#### Monolithic AWS Lambda with internal routing

This approach, though possible, has two major consequences:

- Having the routing in the AWS Lambdas logic, requires to have the AWS Lambda implementing some kind of web framework to or to handle the routing manually
- Given the above, the to deplyoing AWS Lambda would be big in size and responsibility

This approach, is not considered valid and therefore disregarded.

### Custom Lambda authorizer

An API Authorizer is a Lambda function that performs authentication and authorization on requests prior to AWS API Gateway execution. The Lambda returns an IAM policy that either permits or blocks the API requests that contain a particular authorization token. The returning IAM policy is then cached by the API Gateway so it can be later reused for up to 1 hour.

#### :heavy_plus_sign: Pros

You can have your authentication mechanism the way you want it. Ultimate control over authentication and authorization. You can have the UI call the APIs with a standard token (JWT) and the flow for developers remains same. No extra consideration of AWS SDK.

#### :heavy_minus_sign: Cons

Authentication requires a lot of thinking and effort to build. Chances of missing some crucial aspects are always there. Its like reinventing the wheel. Why do it when Amazon has already done it for you.

Note : The authorizer is an intercepting mechanism provided so that you can add custom logic into lambda function and call in authorize calls. The custom logic may use rules based authorization.

## Decision

Since we already have 2 lambda function's setup with CI/CD and infrastructure code, it should be pretty easy for us to setup another one. "Ultimate control over authentication and authorization" sounds very promising too. So this is how we're gonna do it:

![authn_authz_architecture](../../images/authn_authz_architecture.png)

1. The user logs into the app and acquires an Amazon Cognito JWT ID token, access token, and refresh token. [[2]](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)
2. A RestAPI request is made  to our api gateway and a bearer token (our JWT from step 1) is passed in the headers
`Authorization: Bearer {JWT_TOKEN}`
3. The gateway sends this token to our authorizer lambda function. We will have to implement this in a own repo with it's own pipeline.
4. the lambda will verify the token through a JWT library using the aws cognito public key. This key is downloaded and cached on initial invocation of the authorizer lambda function [[1]](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)
5. the lambda looks up the cognito user group the user is in via the JWT token which encodes that information. It then looks up in some database, if there is some IAM policy that's mapped to that user group. We could use dynamoDB or S3 buckets here (TBD).
6. the IAM policy is then being returned to the API gateway (optionally together with a context)

> The context is a map containing key-value pairs that you can pass to the upstream service. It can be additional information about the user, the service, or anything that provides additional information to the upstream service. [[1]](https://aws.amazon.com/blogs/security/building-fine-grained-authorization-using-amazon-cognito-api-gateway-and-iam/)

7. the API Gateway policy engine evaluates the IAM policy

> Note: Lambda isn’t responsible for understanding and evaluating the policy. That responsibility falls on the native capabilities of API Gateway. [[1]](https://aws.amazon.com/blogs/security/building-fine-grained-authorization-using-amazon-cognito-api-gateway-and-iam/)

8. the initial request is forwarded to the service api (together with the context)

### Amazon cognito user pool

The Amazon cognito user pool will handle authentication (authn). We can also implement different cognito user groups.

> Amazon Cognito allows you to use groups to create a collection of users, which is often done to set the permissions for those users.
> [...]
> As a best practice, you should assign users to groups and use group membership to allow or deny access to your API services.
> [...]
>The JWT is used to identify what group the user belongs to, as mapping a group to an IAM policy will display the access rights the group is granted.
 [[1]](https://aws.amazon.com/blogs/security/building-fine-grained-authorization-using-amazon-cognito-api-gateway-and-iam/)

<details><summary>Terraform code for aws cognito user pool</summary>

```hcl
resource "aws_cognito_user_pool" "users" {
  name = "${var.project_name}_user_pool"
  username_attributes = ["email"]
  auto_verified_attributes = ["email"]
  password_policy {
    minimum_length = 6
  }

  verification_message_template {
    default_email_option = "CONFIRM_WITH_CODE"
    email_subject = "Account Confirmation"
    email_message = "Your confirmation code is {####}"
  }

  schema {
    attribute_data_type      = "String"
    developer_only_attribute = false
    mutable                  = true
    name                     = "email"
    required                 = true

    string_attribute_constraints {
      min_length = 1
      max_length = 256
    }
  }
}
```

</details>

E.g. we can have a group for development and one for production. The authorization lambda will later check which group the user is in.
It can provide a simple frontend with a login/registration form can send verification emails, etc. Once the user authenticates through his mobile app it will receive a [JWT](https://jwt.io) token.

> When a user signs into your app, Amazon Cognito verifies the login information. If the login is successful, Amazon Cognito creates a session and returns an ID, access, and refresh token for the authenticated user. You can use the tokens to grant your users access to your own server-side resources or to the Amazon API Gateway. Or you can exchange them for temporary AWS credentials to access other AWS services. [[2]](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)

#### Pricing

It has a permanent free tier that allows up to 50000 montly active users [[3]](https://aws.amazon.com/cognito/pricing/)

### lambda authorizer

This will be another lambda function that we can deploy using a CI/CD and setup the infrastructure in our iac repository.

<details><summary>Here's some dirty sample code I generated using ChatGPT</summary>

```go

package main

import (
    "context"
    "fmt"
    "github.com/aws/aws-lambda-go/events"
    "github.com/aws/aws-lambda-go/lambda"
    "github.com/dgrijalva/jwt-go"
    "strings"
)

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    tokenString := request.Headers["Authorization"]
    if tokenString == "" {
        return events.APIGatewayProxyResponse{}, fmt.Errorf("no Authorization header provided")
    }

    // Remove "Bearer " prefix from token string
    tokenString = strings.TrimPrefix(tokenString, "Bearer ")

    // Parse JWT token
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        // Retrieve public key from AWS Cognito user pool
        region := "your_region"
        userPoolID := "your_user_pool_id"
        keySetURL := fmt.Sprintf("https://cognito-idp.%s.amazonaws.com/%s/.well-known/jwks.json", region, userPoolID)
        keySet, err := jwt.ParseRSAPublicKeyFromPEM([]byte(keySetURL))
        if err != nil {
            return nil, err
        }
        return keySet, nil
    })

    if err != nil {
        return events.APIGatewayProxyResponse{}, fmt.Errorf("failed to parse JWT token: %v", err)
    }

    if !token.Valid {
        return events.APIGatewayProxyResponse{}, fmt.Errorf("invalid JWT token")
    }

    // Token is valid, you can now access its claims (payload) using the token.Claims variable
    // ...
    // TODO: retrieve policy from policy store and return it

    return events.APIGatewayProxyResponse{
        StatusCode: 200,
        Body:       "{IAM_POLICY}",
    }, nil
}

func main() {
    lambda.Start(handler)
}
```

</details>

### policy store

We can easily setup a S3 bucket or dynamoDB to store our policies for the different user groups.
Let's have a look at an example policy.

```json
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"Powermate-API",
         "Effect":"Allow",
         "Action":"execute-api:Invoke",
         "Resource":[
# arn:aws:execute-api:{regionId}:{accountId}:{apiId}/{stage}/{httpVerb}/[{resource}/[{child-resources}]]
            "arn:aws:execute-api:eu-central-1:{accountId}:{apiId}/*/{deviceId}/data", # Any verb
            "arn:aws:execute-api:eu-central-1:{accountId}:{apiId}/*/GET/devices/{deviceId}/sensors/*/historics" # only GET
         ]
      }
   ]
}
```

Notice the `{deviceId}` variable which we can use to grant access to data coming from the device belonging to our user. The authorizer lambda will replace this placeholder value with the deviceId coming from the cognito user attributes. Once we have the previsioning process setup at the end we will have another api call to register a device. Somewhere we need to store the deviceId <-> user mapping. We could use another S3 table of the dynamoDB for that.

## Sources

1. <https://aws.amazon.com/blogs/security/building-fine-grained-authorization-using-amazon-cognito-api-gateway-and-iam/>
2. <https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html>
3. <https://aws.amazon.com/cognito/pricing>
