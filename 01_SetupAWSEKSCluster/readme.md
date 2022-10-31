# Setup AWS EKS Cluster

## Deployment Options

<p align="center"> 
<img width="350" alt="image" src="https://user-images.githubusercontent.com/8760590/198692204-640713b3-e4ba-4725-bfdf-811d56615a8e.png">
</p>

## Options 
- [ ] Init an EKS stack with `eksctl` [here](https://github.com/rodriggj/aws-eks/blob/master/01_SetupAWSEKSCluster/readme.md#eks-procedures)
- [ ] Init an EKS stack with `aws cli` [here](https://github.com/rodriggj/aws-eks/blob/master/01_SetupAWSEKSCluster/readme.md#aws-cli-procedure)

## AWS Dependencies 
- [ ] Require `kubectl` installation [documentation](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- [ ] Require `eksctl` installation [documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [ ] Require `aws cli` installation [documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

------------

## Procedure

### EKS Procedures

1. Create the EKS cluster and nodes. To do this run the following command replacing the _my cluster_, and _region code_ with the appropriate params. 

```s
eksctl create cluster --name my-cluster --region region-code --fargate
```

> AWS Console: Check AWS Cloudformation, and AWS EKS dashboards to see resulting activity. 

2. View the Kubernetes resources. To do this run the following command once the Step 1 above has completed. 

```s
kubectl get nodes -o wide
```

> AWS Console: Again, you can see this information presented on the AWS EKS dashboard. 

3. Delete your cluster and nodes. To do this run the following command again providing the _my cluster_ and _region-code_ params you set in Step 1 above. 

```s
eksctl delete cluster --name my-cluster --region region-code
```

> NOTE: If you are using the `aws cli` or the `AWS Console` to build an EKS cluster and provision nodes there are additional steps needed to configure the EKS cluster (IAM Role, Policy, Attach a Policy, _kubeconfig_ file, etc). So be aware of the difference in procedure depending on the tooling choice selected. 


## Reference
> Documentation: _AWS EKS - Getting Started with Amazon EKS_ [documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)

<small><small>[Back to Top](https://github.com/rodriggj/aws-eks/blob/master/01_SetupAWSEKSCluster/readme.md#options)</small></small>

-----------

### AWS CLI Procedure 

1. Create a dedicated VPC

```s
aws cloudformation create-stack \
  --region us-west-2 \
  --stack-name torque-sandbox-gjr \
  --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
```

2. Create a Policy for EKS User. Create a file called `eks-cluster-role-trust-policy.json` within your root working directory and enter the following contents: 

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

3. Create a Role. 

```s
aws iam create-role \
  --role-name torqueEKSClusterRole \
  --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
```

4. Attach the policy to the role 

```s
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
  --role-name torqueEKSClusterRole
```

5. Create an EKS Cluster

+ Nav to _EKS_ service on AWS Console [here](https://console.aws.amazon.com/eks/home#/clusters)
+ Click _Create Cluster_
+ Name: _gjr-torque-eks-cluster_
+ Cluster Service Role: _torqueEKSClusterRole_
+ VPC: Choose the vpc id from Step 1 above. 
+ Configure Logging: Click _Next_
+ Wait until cluster becomes _Active_ 

6. Configure your computer to communicate with your Cluster. For this you will create a _kubeconfig_ file.

```s 
aws eks update-kubeconfig --region us-west-2 --name gjr-torque-eks-cluster
```

+ Test your configuration. 

```s
kubectl get svc
```

> NOTE: By default, the config file is created in ~/.kube or the new cluster's configuration is added to an existing config file in ~/.kube. You can check to see that this file is created. You can view this file by executing the following: 

```s
nano /Users/gabrielrodriguez/.kube/config
```

7. Create Nodes on your EKS Cluster. To do this you will have to create a `Fargate` profile which will begin by once again have to repeat steps similar to 2 - 4 above. This role and policy will allow Fargate Nodes to be deployed to the EKS service. 

+ Create a Policy file called `pod-execution-role-trust-policy.json` and save in your root working directory. 


```json
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Condition": {
           "ArnLike": {
              "aws:SourceArn": "arn:aws:eks:*:377535795562:fargateprofile/gjr-torque-eks-cluster/*"
           }
        },
        "Principal": {
          "Service": "eks-fargate-pods.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
```

+ Create a pod execution **ROLE**

```s
aws iam create-role \
  --role-name AmazonEKSFargatePodExecutionRole \
  --assume-role-policy-document file://"pod-execution-role-trust-policy.json"
  ```

+ Attach the policy to the role. 

```s
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSFargatePodExecutionRolePolicy \
    --role-name AmazonEKSFargatePodExecutionRole
```

> NOTE: `region-code` is replaced with `*` so I can make this policy available in all regions. The # sequence is my account number. And the `/my-cluster/` is replaced with the EKS cluster name that we created previously. 

8. When this is complete, navigate to the EKS console and find your cluster [here](https://console.aws.amazon.com/eks/home#/clusters). Select your cluster. 

9. Configure a _Fargate Profile_

+ Under the _Compute_ tab, scroll to the _Fargate Profiles_ and click _Add Fargate Profile_
+ Name: gjr-torque-profile
+ Pod Execution Role: Find the `AmazonEKSFargatePodExecutionRole` you created previously
+ Subnets: Deselect any subnets that are public. Only private subnets are supported running Fargate
+ Click _Next_
+ Namespace: _default_
+ Click _Next_
+ Click _Create_

> Don't continue till the Fargate profile status on changes to _Active_ on the Service home dashboard.

10. Configure _CoreDNS_ on Fargate. 

+ Under the _Compute_ tab, scroll to the _Fargate Profiles_, select _gjr-torque-profile_, click _Add Fargate Profile_
+ Name: _CoreDNS_
+ Pod execution role: _AmazonEKSFargatePodExecutionRole_
+ Subnets: Deselect all public subnets
+ Namespace: _kube-system_
+ Click _Match labels_, then _Add label_, Enter key: _k8s-app, & value: kube-dns, click _Next_
+ Click _Create_

11. Run the following command to remove the default `eks.amazonaws.com/compute-type: ec2` annotation from the CoreDNS pods. 

```s
kubectl patch deployment coredns \
    -n kube-system \
    --type json \
    -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'
```

> NOTE: The system creates and deploys two nodes based on the Fargate profile label you added. You won't see anything listed in Node Groups because they aren't applicable for Fargate nodes, but you will see the new nodes listed in the Overview tab.

12. View Resources. 

+ Choose Clusters, and select your cluster's name. 
+ Click the _Compute_ tab. You see the list of Nodes that were deployed for the cluster. You can choose the name of a node to see more information about it.
+ Click _Resources_ tab. You see all of the Kubernetes resources that are deployed by default to an Amazon EKS cluster. Select any resource type in the console to learn more about it.

13. Delete the Resources. 
+ Nav to the _Clusters_, then _Compute_ tab and delete any nodes or node-groups
+ Delete the _Fargate Profile_
+ Delete the Cluster
+ Delete the VPC AWS CloudFormation Stack
+ Delete the IAM roles that were created. 


## Reference
> Documentation: _AWS EKS - Getting Started with Amazon EKS_ [documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

<small><small>[Back to Top](https://github.com/rodriggj/aws-eks/blob/master/01_SetupAWSEKSCluster/readme.md#options)</small></small>