# Ingress Controller

### OIDC provider
# trust and authentication provider

REGION_CODE=us-east-1
CLUSTER_NAME=amazon-clone
ACC_ID=""

 
eksctl utils assoociate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

# Download IAM policy
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.3.0/docs/install/iam_policy.json

# creaate IAM policy

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json


# create IAM Role and k8 ServiceAccount

eksctl create iamserviceaccount \
--cluster= $CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

# Install Load Balancer controller drivers

helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName= $CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller