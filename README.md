# scheduled_security_hub_prowler_findings

- [Purpose](#purpose)
- [Deploying to AWS](#deploying-to-aws)
  - [Requirements](#requirements)
  - [Parameters](#parameters)
  - [Deployment](#deployment)

## Purpose

This YAML template deploys cross-referenced stacks which allows for scheduled tasks that run Prowler (https://github.com/toniblyx/prowler) on a regular basis in any intended region(s). The Prowler findings get pushed to AWS Security Hub to allow for a comprehensive view of the findings. The prowler.yml template contains all the common resources and must be deployed first and only needs to be deployed once. The event-rules.yml template contains the event rule resource which runs scheduled Prowler tasks in a given region and is dependent on outputs from the prowler.yml stack. Thus, event-rules.yml must be deployed after prowler.yml and can be deployed as many times as there are regions you intend to run scheduled Prowler tasks in.

## Deploying to AWS

The following steps outline how to deploy this stack to AWS.

### Requirements

The following requirements must be satisfied prior to deploying this stack:
- An existing ECS cluster for the tasks to run in (this stack does not create an ECS cluster)
- An existing VPC to run the ECS tasks from
- An existing public subnet to run the ECS tasks from
- A clone of this repository on your local machine

### Parameters

The following parameters are the required parameters for deploying the **prowler.yml** YAML template and have to be defined upon deployment:
- **EcsClusterName** - The name of the ECS cluster to be used
- **ScheduleRate** - The rate at which the Prowler tasks will run (e.g. daily, weekly, monthly. Default is weekly). This must follow the syntax of a schedule expression (https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-schedule-expressions.html)
- **Subnets** - The subnet ID(s) to be used by the Prowler tasks
- **VpcID** - The VPC ID to be used by the Prowler tasks

The following parameter is required for deploying the **event-rules.yml** YAML template and has to be defined upon deployment:
- **AwsRegion

### Deployment

Because these are cross-referenced stacks, the event-rules.yml stacks are dependent on outputs from the prowler.yml stack. Therefore, prowler.yml must be deployed first and then event-rules.yml can be deployed as many times as intended afterwards.

Deploying the prowler.yml stack from the YAML file requires the use of CloudFormation. There are two options for deploying:
1. You can deploy this via the AWS CloudFormation console. This would be deployed via creating a stack with new resources (standard) and providing the parameters required.
2. You can deploy this via the AWS CLI. Use the following command template to run the appropriate CLI command from the directory of the copy of this repository.
```
aws cloudformation deploy \
        --template-file ./prowler.yml \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides \
            EcsClusterName={the ECS cluster name}  \
            ScheduleRate={the intended schedule rate}\
            Subnets={the intended subnet} \
            VpcID={the intended VPC} \
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
            ECSClusterName=ProwlerCluster \
            scheduleRate="rate(7 days)" \
            subnets=subnet-1a2b3c4b \
            vpcID=vpc-1a2b3c4d \
        --stack-name ScheduledProwler \
        --profile staging \
        --region us-east-1
```

Once prowler.yml has been deployed, you can now deploy event-rules.yml as many times as there are regions you want to run scheduled Prowler tasks. For example, if you want to have scheduled Prowler tasks in us-east-1, us-east-2, us-west-1, and us-west-2, you would want to deploy event-rules.yml 4 times.

Deploying the event-rules.yml stack from the YAML file requires the use of CloudFormation. There are two options for deploying:
1. You can deploy this via the AWS CloudFormation console. This would be deployed via creating a stack with new resources (standard) and providing the parameters required.
2. You can deploy this via the AWS CLI. Use the following command template to run the appropriate CLI command from the directory of the copy of this repository.
```
aws cloudformation deploy \
        --template-file ./event-rules.yml \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides \
            AwsRegion={the intended AWS region}\
        --stack-name {the intended name for the stack (can be anything)} \
        --profile {the AWS profile to use} \
        --region {the AWS region to deploy to}
```

The following is an example command for deploying via AWS CLI:

```
aws cloudformation deploy \
        --template-file ./event-rules.yml \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides \
            AwsRegion=us-east-1
        --stack-name ScheduledProwlerUsEast1 \
        --profile staging \
        --region us-east-1
```
