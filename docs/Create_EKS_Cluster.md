## Chapter 3 : Create Kubernetes Cluster Using Amazon EKS

### 1. Create AWS Resources - IAM

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


### 2. Create AWS Resources VPC

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

### 3. Create EKS Cluster

Create EKS using below command
 
 ```
 aws eks create-cluster --name ekspoc --role-arn arn:aws:iam::012345678910:role/ekspoc --resources-vpc-config subnetIds=subnet-6782e71e,subnet-e7e761ac,securityGroupIds=sg-6979fe18
 
 ```
 
 Above step will create eks cluster and display below output
 
 ```
 {
    "cluster": {
        "name": "ekspoc",
        "arn": "arn:aws:eks:us-east-1:account:cluster/ekspoc",
        "createdAt": 1555033659.174,
        "version": "1.12",
        "roleArn": "arn:aws:iam::account:role/ekspoc",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-0138cd554f0",
                "subnet-0e79e96f"
            ],
            "securityGroupIds": [
                "sg-0d0feee0c2f"
            ],
            "vpcId": "vpc-01bdccd9"
        },
        "status": "CREATING",
        "certificateAuthority": {},
        "platformVersion": "eks.1"
    }
}

```

It takes about 5 minutes before your cluster is created. You can ping the status of the command using this CLI command:

```
aws eks --region us-east-1 describe-cluster --name ekspoc --query cluster.status

```

To delete the cluster, use

```
aws eks delete-cluster --name ekspoc
```

To access eks cluster, setup CI tools and configure with newly created eks cluster

### 4. Setup CLI tools for EKS cluster

* Configure aws-iam-authenticator - A tool to authenticate to Kubernetes using AWS IAM credentials
* Download and configure aws-iam-authenticator 
** https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html 
* Configure kubectl 
** https://kubernetes.io/docs/tasks/tools/install-kubectl/ 

Once the status changes to “ACTIVE”, we can proceed with updating our kubeconfig file with the information on the new cluster so kubectl can communicate with it.

To do this, we will use the AWS CLI update-kubeconfig command (be sure to replace the region and cluster name to fit your configurations):

```
aws eks --region us-east-1 update-kubeconfig --name ekspoc
```

You should see following output
```
added new context arn:aws:eks:us-east-1:account:cluster/ekspoc to /Users/user/.kube/config
```

Check status of cluster:

```
kubectl get svc
```

Given aws-iam-authenticator is in path, configured correctly and kubeconfig is updated with cluster name, you should see below output

```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   80m

```

### 5. Launching Kubernetes worker nodes
Now that we’ve set up our cluster and VPC networking, we can now launch Kubernetes worker nodes. To do this, we will again use a CloudFormation template.

Follow steps mentioned at https://logz.io/blog/amazon-eks/

Note down ARN of worker nodes

Download aws-auth-cm.yaml
```
curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-09/aws-auth-cm.yaml 
```

Update ARN role of worker nodes in aws-auth-cm.yaml and deploy in cluster

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::Account:role/ekspoc-worker-node-NodeInstanceRole-1NYSOJ5910EPV
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
```

Once ARN is updated, apply above configmap:

```
kubectl apply -f aws-auth-cm.yaml
```

You should see below output

```
configmap/aws-auth created
```
 
 #### 5. Configuration file for kubectl
 
 Replace Endpoint URL, Certs, args (Name of cluster), 
 
 Template of Kubeconfig
 
 ```
 apiVersion: v1
clusters:
- cluster:
    server: <endpoint-url>
    certificate-authority-data: <base64-encoded-ca-cert>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
kind: Config
preferences: {}
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "EKS-cluster"

 ```
 
 Check status of cluster, nodes should be part of cluster now
 
 ```
 kubectl get nodes --watch
 ```

 
 ### 7. Deploy application on eks cluster
 
 
 ### References
 https://logz.io/blog/amazon-eks/
 https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html
 
