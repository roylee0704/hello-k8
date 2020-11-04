# hello-k8
a collections of k8 exp


## Getting Started

Make sure you are connected to any k8s cluster choices of yours. In this example, I'm connecting to k8s from aws-provider, eks.
 
```sh
# authenticate myself to aws provider
$ aws-vault exec roy@xyz

# download k8s context from eks
$ aws eks --region ap-southeast-1 update-kubeconfig --name your-cluster-name

# command k8s to connect to eks via downloaded context
$ kubectl config use-context arn:aws:eks:ap-southeast-1:xyx
```