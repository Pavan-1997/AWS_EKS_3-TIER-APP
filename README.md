# AWS_EKS_3-TIER-ARCHITECTURE

## THREE TIER ARCHITECTURAL MODEL:

- FRONT-END - UI - Presentation Layer

- BACK-END - Logic Layer

- DATABASE - Data Layer


## HIGH LEVEL DESIGN:

- Workflows -> Components

- User > Registration, Login > Catalogue - AI, Robots > Rating for Robot > Cart > Payments - Integrated with any payment gateway > Shipping details > Order completed user gets a Order ID

- We can do it as a Monolithic or Microservice architechture 

- UI -> Angular (Presentation)

- Logic -> Microservice (Cart, Catalogue, Payments, Shipping, User)

- DB -> Redis (In memory datastore), MongoDB (User details), MySQL (Ratings)

- OIDC - Integrating service accounts of Persistant volumes to IAM roles/policies (Redis -> Stateful Set -> Persistant Volume -> EBS)

---

ACCESS-KEY
SECRET-KEY

Region - us-west-1

eksctl create cluster --name=eks-robot-shop-server --region=us-west-1 --zones=us-west-1a,us-west-1c --without-nodegroup

eksctl create nodegroup --cluster=eks-robot-shop-server --region=us-west-1 --name=eksdemo-ng-public --node-type=t2.medium --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=10 --ssh-access --ssh-public-key=AWS-KEYPAIR-NC --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

export cluster_name=eks-robot-shop-server

oidc_id=$(aws eks describe-cluster --region us-west-1 --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

eksctl utils associate-iam-oidc-provider --region us-west-1 --cluster $cluster_name --approve


curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
	
eksctl create iamserviceaccount \
  --cluster=$cluster_name \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::009403810934:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

sudo apt update

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

chmod +x get_helm.sh
./get_helm.sh

helm version

helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$cluster_name --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-west-1 --set vpcId=vpc-080eba0d104598a17

kubectl get deployment -n kube-system aws-load-balancer-controller


EBS CSI - when a PVC is created then the EBS volume is automatically created and attached to the Redis Stateful Set 

eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster $cluster_name \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
	
eksctl create addon --name aws-ebs-csi-driver --cluster $cluster_name --service-account-role-arn arn:aws:iam::009403810934:role/AmazonEKS_EBS_CSI_DriverRole --force


kubectl create ns robot-shop

cd /home/ubuntu/three-tier-architecture-demo/EKS/helm

helm install robot-shop --namespace robot-shop .

kubectl get pods


kubectl get svc -n robot-shop

You can see the LB


Using Ingress

cd /home/ubuntu/three-tier-architecture-demo/EKS/helm

kubectl apply -f ingress.yaml

kubectl get svc -n robot-shop

Go to the Load Balancers and access the DNS
