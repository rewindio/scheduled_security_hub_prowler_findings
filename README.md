# scheduled_security_hub_prowler_findings

[build badges]

- [Purpose](#purpose)
- [Deploying to AWS](#deploying-to-aws)
  - [Requirements](#requirements)
  - [Parameters](#parameters)
  - [deployment](#deployment)

## Purpose

This YAML template deploys a stack which allows for scheduled tasks that run Prowler (https://github.com/toniblyx/prowler) on a regular basis in the following regions: us-east-1, ca-central-1, eu-west-1, and eu-central-1. The Prowler findings get pushed to AWS Security Hub to allow for a comprehensive view of the findings.

## Deploying to AWS

The following steps outline how to deploy this stack to AWS.

### Requirements

The following requirements must be satisfied prior to deploying this stack:
- An existing ECS cluster for the tasks to run in (this stack does not create an ECS cluster)
- An existing VPC to run the ECS tasks from
- An existing public subnet to run the ECS tasks from
- A clone of this repository on your local machine

### Parameters

The following parameters are the required parameters for deploying the YAML template and have to be defined upon deployment:
- **ECSClusterName** - The name of the ECS cluster to be used
- **scheduleRate** - The rate at which the Prowler tasks will run (e.g. daily, weekly, monthly. Default is weekly). This must follow the syntax of a schedule expression (https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-schedule-expressions.html)
- **subnets** - The subnet ID(s) to be used by the Prowler tasks
- **verbose** - A flag to show debugging logs. Allowed values are **true** and **false**
- **vpcID** - The VPC ID to be used by the Prowler tasks

### Deployment

Deploying this stack from the YAML file requires the use of CloudFormation. There are two options for deploying:
1. You can deploy this via the AWS CloudFormation console. This would be deployed via creating a stack with new resources (standard) and providing the parameters required.
2. You can deploy this via the AWS CLI. Use the following command template to run the appropriate CLI command from the directory of the copy of this repository.
```
aws cloudformation deploy \
        --template-file ./prowler.yml \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides \
            ECSClusterName={the ECS cluster name}  \
            scheduleRate={the intended schedule rate}\
            subnets={the intended subnet} \
            verbose={true or false} \
            vpcID={the intended VPC} \
        --stack-name {the intended name for the stack (can be anything)} \
        --profile {the AWS profile to use} \
        --region {the AWS region to deploy to}
```

The following is an example command for deploying via AWS CLI:

```
aws cloudformation deploy \
        --template-file ./prowler.yml \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides \
            ECSClusterName=rewind-devops \
            scheduleRate="rate(7 days)" \
            subnets=subnet-0f94b054 \
            verbose=true \
            vpcID=vpc-e992e88f \
        --stack-name dj-test-prowler \
        --profile staging \
        --region us-east-1
```
