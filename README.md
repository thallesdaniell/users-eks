# Giving users access to the EKS


## const role
`POLICY="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::XXXXXXXXXXXXXX:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"`

## create role 
`ROLE=$(aws iam create-role --role-name KubectlRole --assume-role-policy-document "$POLICY" --output text --query 'Role.Arn')`

## create file policy
`echo '{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": "eks:*", "Resource": "Describe*" } ] }' > /tmp/iam-role-policy`
 
 ## update role with policy
`aws iam put-role-policy --role-name KubectlRole --policy-name FullAccessPolicy --policy-document file:///tmp/iam-role-policy`


## check assume role
`aws sts assume-role --role-arn $ROLE --role-session-name test`

## edit aws-auth
`kubectl edit configmap -n kube-system aws-auth`

## add mapRoles and mapUsers
```
data:
  mapRoles: |
    - rolearn: arn:aws:iam::XXXXXXXXXXXXXX:role/KubectlRole
      username: KubectlRole
      groups:
      - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::XXXXXXXXXXXXXX:user/user-test@aws.com
    username: user-test@aws.com
    groups:
      - system:masters
```

## update kubeconfig 
`aws eks update-kubeconfig --name housi --region us-east-1 --role-arn $ROLE`
