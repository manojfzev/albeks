# Deploy ALB for EKS Cluster #

cluster_name=(cluster name)

oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

### Determine whether an IAM OIDC provider with your cluster's issuer ID is already in your account.

aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

### If output is returned, then you already have an IAM OIDC provider for your cluster and you can skip the next step. If no output is returned, then you must create an IAM OIDC provider for your cluster.

eksctl utils associate-iam-oidc-provider --cluster=$cluster_name --approve

### Authenticate with EKS cluster

aws eks update-kubeconfig --name $cluster_name

### Deploy cert manager in EKS

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.13.5/cert-manager.yaml

### Execute yaml files in permission dir first if role already created if not then execute below commands also remember to update the trust relationships with oidc id if role already exsist.

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<AWS-Account-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

### Apply rest of the yaml files using kubectl apply -f <filename> in order as per below.

01-v2_7_2_full.yaml (Update --cluster-name in container args section)

02-v2_7_2_ingclass.yaml

03-ingress.yaml

### Then deploy the application using below yaml files

nginx-deployment-v2.yaml

nginx-deployment.yaml

## References

https://github.com/RobinNagpal/kubernetes-tutorials/tree/master/06_tools/007_alb_ingress/01_eks
https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html 
https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
