
### Calico Installation

Once cluster is setup, install calico using below command:

```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/v1.3/calico.yaml
```

Validate that daemon sets came online

```
kubectl get daemonset calico-node --namespace kube-system
```

It should give below outcome

```
NAME          DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
calico-node   3         3         3         3            3           <none>          38s

```

To delete Calico from EKS clsuer

```
kubectl delete daemonset calico-node --namespace kubesystem
```

Output:

```
daemonset.extensions "calico-node" deleted
```

### Create Network Policy:

https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/stars-policy/


#### Reference
https://docs.aws.amazon.com/eks/latest/userguide/calico.html
