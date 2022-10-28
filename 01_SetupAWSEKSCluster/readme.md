# Setup AWS EKS Cluster

## Deployment Options

<p align="center"> 
<img width="350" alt="image" src="https://user-images.githubusercontent.com/8760590/198692204-640713b3-e4ba-4725-bfdf-811d56615a8e.png">
</p>

## AWS Dependencies 
- [ ] Require `kubectl` installation [documentation](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- [ ] Require `eksctl` installation [documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [ ] Require `aws cli` installation [documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## Procedure
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
