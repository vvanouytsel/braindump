#aws #cloud 
## IAM

For physical users:

* Specify `Policies` to `Users`

* Specify `Policies` to `Groups`

* Add `Users` to `Groups`

For services:

* Add `Policies` to `Roles`

* Add `Roles` to `Services`

* Always use IAM Roles to provide permissions to services

e.g. Never configure AWS credentials in your EC2 instance, instead attach the necessary permissions to your instance. (example: `IAMReadOnlyAccess` if you want to use `aks iam list-users`)

You can simulate your policies using the [IAM Policy Simulator](https://policysim.aws.amazon.com/home/index.jsp?#)

## EC2

You can get metadata about your EC2 instance directly via an API call:

```bash

$ curl http://169.254.169.254/latest/meta-data/

  

ami-id

ami-launch-index

hostname

iam/

instance-action

instance-id

...

```

## Networking

AWS reserves the first 4 and the last IP address of each subnenet.

```bash

# example: 10.0.0.0/24

# 10.0.0.0 reserved as network address

# 10.0.0.1 reserved by AWS for the VPC router

# 10.0.0.2 reserved by AWS for mapping Amazon-provided DNS

# 10.0.0.3 reserved by AWS for future use

# 10.0.0.255 broadcast ip, which is not supported in AWS VPC, thus it is reserved

```

## CLI

* Log in to AWS cli using sso

```bash

$ aws sso login

```

* Set AWS_PROFILE environment variable

```bash

$ export AWS_PROFILE=Pipelines-FullAdmin

```

* Get information of the user making the request

```bash

$ aws sts get-caller-identity

  

{

"UserId": "AIDAJQABLZS4A3QDU576E",

"Account": "BLABLABLA"

"Arn": "arn:aws:iam::123456789012:user/MyUserName"

}

```

## SSM

* Port forward port 9090 on the AWS instance to your local port 8085

```bash

$ aws ssm start-session --target i-0649a4e9c9d068003 --region eu-west-1 --document-name AWS-StartPortForwardingSession --parameters '{"portNumber":["9090"],"localPortNumber":["8085"]}'

```

## CloudWatch

Using CloudWatch Insights you can query your logs.

* A ugly way to show 5XX errors in ingress-nginx-controller

```text

fields @timestamp, @message

| filter kubernetes.namespace_name like "ingress"

| filter @message like "HTTP/2.0\" 5"

```

## References

* [SaaS factory EKS reference](https://github.com/aws-samples/aws-saas-factory-eks-reference-architecture)