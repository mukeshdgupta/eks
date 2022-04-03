# <h1>Cluster Notes</h1>


# <h2>CONSOLE CREDENTIALS: Import your EKS Console credentials to your new cluster</h2>

```
c9builder=$(aws cloud9 describe-environment-memberships --environment-id=$C9_PID | jq -r '.memberships[].userArn')
if echo ${c9builder} | grep -q user; then
	rolearn=${c9builder}
        echo Role ARN: ${rolearn}
elif echo ${c9builder} | grep -q assumed-role; then
        assumedrolename=$(echo ${c9builder} | awk -F/ '{print $(NF-1)}')
        rolearn=$(aws iam get-role --role-name ${assumedrolename} --query Role.Arn --output text) 
        echo Role ARN: ${rolearn}
fi

```
```
eksctl create iamidentitymapping --cluster eksworkshop-eksctl --arn ${rolearn} --group system:masters --username admin

```
```
kubectl describe configmap -n kube-system aws-auth

```
# <h2>Commands</h2>

```
eksctl create cluster -f eksworkshop.yaml
kubectl get nodes # if we see our 3 nodes, we know we have authenticated correctly
kubectl get svc,po,deploy



```
# <h2>Export the Worker Role Name</h2>

```
STACK_NAME=$(eksctl get nodegroup --cluster eksworkshop-eksctl -o json | jq -r '.[].StackName')
ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile


```

# <h2>EC2 Spot Instances in managed node groups </h2>

```
eksctl create managednodegroup --cluster eksworkshop-eksctl --instance-types m5.xlarge,m4.xlarge,m5a.xlarge,m5.large,m5ad.large,m5a.large --managed --spot --name spot-4vcpu-16gb --asg-access --nodes-max 10
or 
eksctl create managednodegroup --cluster eksworkshop-eksctl --instance-types m5ad.xlarge,m5.large,m5a.xlarge,m4.xlarge --managed --spot --name spot-4vcpu-16gb --asg-access --nodes-max 10
kubectl get nodes --selector=eks.amazonaws.com/capacityType=SPOT
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
kubectl edit deployment cluster-autoscaler -n kube-system
<-- modify <YOUR CLUSTER NAME> -->
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
kubectl get no -l eks.amazonaws.com/capacityType=SPOT
kubectl delete deployment nginx-spot-demo

eksctl delete nodegroup on-demand-4vcpu-16gb --cluster eks-spot-managed-node-groups

eksctl delete nodegroup spot-4vcpu-16gb --cluster eks-spot-managed-node-groups

eksctl delete cluster eks-spot-managed-node-groups

```



