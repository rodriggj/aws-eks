# Helm on EKS

## What is Helm
Helm is a tool for managing Kubernetes packages called charts. Helm can do the following:

+ Create new charts from scratch
+ Package charts into chart archive (tgz) files
+ Interact with chart repositories where charts are stored
+ Install and uninstall charts into an existing Kubernetes cluster
+ Manage the release cycle of charts that have been installed with Helm

## Architecture
For Helm, there are three important concepts:

+ The `chart` is a bundle of information necessary to create an instance of a Kubernetes application.
+ The `config` contains configuration information that can be merged into a packaged chart to create a releasable object.
+ A `release` is a running instance of a chart, combined with a specific config.

## Prerequisites
The following prerequisites are required for a successful and properly secured use of Helm.

+ A Kubernetes cluster
+ Deciding what security configurations to apply to your installation, if any
+ Installing and configuring Helm.

## Installation

1. If you have mac and with `brew` package manager you can execute the following command. 

```s
brew install helm
helm version
```

2. If you are on a Windows PC you will need to install _Helm_ using `Chocolatey` [documentation](https://chocolatey.org/install) or Scoop [documentation](https://scoop.sh/)

> Information: If you want to have a discussion on which package manager to install for a Windows environment you can view the following documenat for a comparison [here](https://www.onmsft.com/feature/scoop-or-chocolatey-which-windows-10-package-manager-should-you-use). Please make you own decision on which of the two package managers to use; Quali does not endorse one over another. 

## Implementing your First Helm Chart

1. Check to ensure there are no Helm charts currently in use on your system. 

```s
helm repo list
```

> Return Value: If no Helm charts are available then you will receive an error in your console such as `Error: no repositories to show`

2. To add a repo execute the following command

```s
helm repo add stable https://charts.helm.sh/stable
```

> Return Value: Executing the above code should render a console return value of `"stable" has been added to your repositories`

If you now run 

```s
helm repo list
```

> Return Value: Will render something like ...
```s
NAME    URL                          
stable  https://charts.helm.sh/stable
```

3. Helm charts can be updated within the repo so to receive any updates to the Helm chart run the following commnad.

```s
helm repo update 
```

> Return value: Will render something like ...

```s
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

4. To see what is available in terms of Helm charts in that particular repository you can run the following command: 

```s 
helm search repo
```

> Return value: Will render something like ...

<p align="center">
<img width="350" alt="image" src="https://user-images.githubusercontent.com/8760590/199389922-3071c28d-8f61-49b6-8da0-d17379b2c95e.png">
</p>

5. Most of the Helm charts in the stable repo are depreciated. So if we wanted to install a new repo for say the `redis` key-value database. We could 1. install the redis repo & 2. pull from the redis repo for the Helm chart for a redis installation. These can be found from the `Artifact Hub` [here](https://artifacthub.io/packages/helm/redis/redis)

<p align="center">
<img width="350" alt="image" src="https://user-images.githubusercontent.com/8760590/199390680-a0ce310f-4faa-4ce6-8f86-bdbe5ee0ffbb.png">
</p>


In this diagram you can see that if we want to iinstall redis we coud do so with the following commands: 

```s
helm repo add redis https://spy86.github.io/redis
```

You would then **have to have a Kubernetes Cluster running** on EKS or otherwise to execute the subsequent command which is 
```s
helm install my-redis redis/redis --version 0.1.1
```

> Error: If you do not have Kubernetes cluster running or you are not configured to communicate with the Kubernetes cluster you will see an error like `Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "https://F02418D580216C743A9501FA24C45B11.gr7.us-west-2.eks.amazonaws.com/version": dial tcp: lookup F02418D580216C743A9501FA24C45B11.gr7.us-west-2.eks.amazonaws.com on [2001:558:feed::1]:53: no such host`. Return to the SetupAWSEKSCluster instruction [here](https://github.com/rodriggj/aws-eks/tree/master/01_SetupAWSEKSCluster#options) to configure the cluster, then repeat the Helm installation command. 

6. If you want to see what is running on your K8 cluster, that was installed by helm run the following command: 

```s
helm ls
```

7. If for any reason you want to `unistall` any helm installation run the following command: 

```s
helm uninstall {my-redis}
```

> Vefify: Verify that the _unistall_ command executed by again running the `helm ls` command. 

## Reference 

- [ ] Helm Documentation [here](https://helm.sh/)
- [ ] Artifact Hub [here](https://artifacthub.io/)