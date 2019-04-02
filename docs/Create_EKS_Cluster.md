## Chapter 3 : Create Kubernetes Cluster Using Amazon EKS

### Create AWS Resources - IAM

Create IAM user: This is used we will be using to create AWS resources 

```
aws iam create-user --user-name ekspoc
```

Create two additional EKS Policies

```
aws iam create-policy --policy-name EKS-Admin-policy --policy-document file://EKS-Admin-policy.json
aws iam create-policy --policy-name CloudFormation-Admin-policy --policy-document file://CloudFormation-Admin-policy.json
```

First is EKS-Admin-policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        }
    ]
}

```

Second is CloudFormation-Admin-policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*"
            ],
            "Resource": "*"
        }
    ]
}

```

Attach these policies to user

```
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::AccountNumber:policy/EKS-Admin-policy
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::AccountNumber:policy/CloudFormation-Admin-policy
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::aws:policy/IAMFullAccess
aws iam attach-user-policy --user-name ekspoc --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess

```

Create IAM role and attach below two policy: 
 
AmazonEKSClusterPolicy
AmazonEKSServicePolicy

Command line gives exceed size issue, if someone wants to try command line below:
```
aws iam create-role --role-name ekspoc --assume-role-policy-document file://AmazonEKSClusterPolicy.json
aws iam create-role --role-name ekspoc --assume-role-policy-document file://AmazonEKSServicePolicy.json
```

Be sure to note the Role ARN, you will need it when creating the Kubernetes cluster in the steps below.

For example : arn:aws:iam::AccountNumber:role/ekspoc


### Create AWS Resources VPC

We will create VPC using existing CFT provided by amazon

CFT Details
```
https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-
09/amazon-eks-vpc-sample.yaml

```

Create VPC:
```
aws cloudformation create-stack --stack-name "ekspoc-vpc" --template-url "https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-09/amazon-eks-vpc-sample.yaml"
```

Note down VPCId, Security groups and SubnetID's from above step

### Create EKS Cluster

Create EKS using below command
 
 ```
 aws eks create-cluster --name ekspoc --role-arn arn:aws:iam::012345678910:role/eks-service-role-AWSServiceRoleForAmazonEKS-J7ONKE3BQ4PI --resources-vpc-config subnetIds=subnet-6782e71e,subnet-e7e761ac,securityGroupIds=sg-6979fe18
 
 ```
 
 ### Setup CLI tools for EKS cluster
