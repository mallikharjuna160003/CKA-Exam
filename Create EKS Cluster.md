1. create cluster role policy and store the policy in json file
```sh
cat >eks-cluster-role-trust-policy.json <<EOF
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
EOF
```
2. create eks cluster role
```sh
aws iam create-role --role-name myAmazonEKSClusterRole --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
```
3. attach the cluster role
  ```sh
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy --role-name myAmazonEKSClusterRole
```
4. create cluster
```sh
 eksctl create cluster --name my-cluster --region us-east-1 --version 1.29 --vpc-private-subnets subnet-09f0a82af2a3ddcf1,subnet-01ce2d0713b5923db --without-nodegroup
```
5. setup for open id connect to allow service accounts to access the iam roles to for aws resources api calls from k8s.
```sh
cluster_name=my-cluster
echo $oidc_id
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
6. Create service account
```sh
cat >my-service-account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
EOF
kubectl apply -f my-service-account.yaml
account_id=$(aws sts get-caller-identity --query "Account" --output text)
oidc_provider=$(aws eks describe-cluster --name my-cluster --region $AWS_REGION --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///")
export namespace=default
export service_account=my-service-account
cat >trust-relationship.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$account_id:oidc-provider/$oidc_provider"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "$oidc_provider:aud": "sts.amazonaws.com",
          "$oidc_provider:sub": "system:serviceaccount:$namespace:$service_account"
        }
      }
    }
  ]
}
EOF
aws iam create-role --role-name my-role --assume-role-policy-document file://trust-relationship.json --description "my-role-description"
aws iam attach-role-policy --role-name my-role --policy-arn=arn:aws:iam::$account_id:policy/my-policy
kubectl annotate serviceaccount -n $namespace $service_account eks.amazonaws.com/role-arn=arn:aws:iam::$account_id:role/my-role
aws iam get-role --role-name my-role --query Role.AssumeRolePolicyDocument
aws iam list-attached-role-policies --role-name my-role --query AttachedPolicies[].PolicyArn --output text
export policy_arn=arn:aws:iam::111122223333:policy/my-policy
aws iam get-policy --policy-arn $policy_arn
aws iam get-policy-version --policy-arn $policy_arn --version-id v1
kubectl describe serviceaccount my-service-account -n default
# create managed nodegroups
eksctl create nodegroup   --cluster my-cluster   --region us-east-1   --name my-nodegroup   --node-ami-family Ubuntu2004   --node-type t3.medium   --nodes 3   --nodes-min 2   --nodes-max 4   --ssh-access   --ssh-public-key my-key-pair

```


