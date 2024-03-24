# 2048-App-Deployment-EKS
A brief about 2048 application deployment on EKS

PREQUISITES : 
below packages should be installed in your local machine/VM
KUBECTL, AWS CLI, HELM, EKSCTL

STEPS 
Create an EKS cluster using AWS fargate
  eksctl create cluster --name demo-cluster --region us-east-1 --fargate
  
After running this command, your kubeconfig file will be updated with the necessary authentication and configuration information to interact with the specified EKS cluster using tools like kubectl.

 aws eks update-kubeconfig --name demo-cluster-1 --region us-east-1


Create a fargate profile

 eksctl create fargateprofile \
--cluster demo-cluster-1 \
--region us-east-1 \
--name alb-sample-app \
--namespace game-2048


Deploy app using game-2048.yml

Kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml


Check the details of all the resources being created

kubectl get all -n game-2048
kubectl get ingress -n game-2048
kubectl get pods -n game-2048
kubectl get svc -n game-2048


The command you provided is used to associate the IAM OIDC provider with your Amazon EKS cluster. This association is necessary for your cluster to use AWS IAM for authentication and authorization purposes

 eksctl utils associate-iam-oidc-provider --cluster demo-cluster-1 --region us-east-1 --approve


Download Iam policy

 curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json


Create Iam Policy

 aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json


Create IAM role attach it to the policy

 eksctl create iamserviceaccount \
--cluster=<your-cluster-name> \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
-- region us-east-1 --approve


Install helm charts

sudo snap install helm --classic


add helm repo

 helm repo add eks https://aws.github.io/eks-charts


update the repo

 helm repo update eks


Install alb controller using helm repo

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=demo-cluster-1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-004XXXXXXXXXfe


Check the alb controller installation status

kubectl get deployment -n kube-system aws-load-balancer-controller

Access the application using output of the commnad $kubectl get ingress -n 


DELETE THE RESOURCES:

list the node groups

aws eks list-nodegroups --cluster-name demo-cluster-1 --region us-east-1


delete the node group

aws eks delete-nodegroup --cluster-name demo-cluster-1 --nodegroup-name <nodegroup-name> --region us-east-1


CHECK THE STATUS OF NODE GROUP ; status should be deleted

aws eks describe-nodegroup --cluster-name demo-cluster-1 --nodegroup-name ng-XXXX68 --region us-east-1


list fargate profile

 aws eks list-fargate-profiles --cluster-name demo-cluster-1 --region us-east-1


Delete the fargate profile

 aws eks delete-fargate-profile --cluster-name demo-cluster-1 --fargate-profile-name alb-sample-app --region us-east-1

Delete the cloud formation stack resources as well to avoid conflict when you try to create cluster next time.


delete the cluster

 aws eks delete-fargate-profile --cluster-name demo-cluster-1 --fargate-profile-name alb-sample-app --region us-east-1


delete VPC, NAT, Elastic IP and load Balancer as well
