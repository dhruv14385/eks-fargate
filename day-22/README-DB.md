# 2048 App  

Summary: Create EKS cluster using Fargate nodes.  

```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

Link kubectl to EKS cluster. The command below helps point kubectl to the cluster we want, out of many available clusters. 
```
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster 
```

Create Fargate profile

```
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

Deploy the deployment, service and Ingress. Basically, this application code is taken from link below.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
Download json file for IAM policy for Application Load Balancer. 
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json 
```
Create IAM policy from this json file 
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json 
```
Associate this policy with EKS 
```
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve 
```
Create IAM role for EKS. Update your AWS account number and region. 
```
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2 
```
Install helm 
```
sudo snap install helm --classic 
```
Add EKS chart repository to helm 
```
helm repo add eks https://aws.github.io/eks-charts 
```
Get update for the repository above 
```
helm repo update eks 
``` 

Check repo list 
```
helm repo list 
```
Install ALB controller 
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=<region> --set vpcId=<your-vpc-id>
```

Check if the controller above is installed  
```
kubectl get deployment -n kube-system aws-load-balancer-controller 
```
Use Load Balancer DNS name to access the application.


![Screenshot 2023-08-03 at 7 57 15 PM](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/93b06a9f-67f9-404f-b0ad-18e3095b7353)
