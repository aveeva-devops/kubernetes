This chapter will cover various kubernetes controllers. This includes
* ReplicaSet
* ReplicationController
* Deployment
* StatefulSets
* DaemonSet
* Garbage Collection
* TTL Controller for Finished Resources
* CronJob

### Replication Controller
The Replication Controller is the original form of replication in Kubernetes.  It’s being replaced by Replica Sets, but it’s still in wide use, so it’s worth understanding what it is and how it works.

A Replication Controller is a structure that enables you to easily create multiple pods, then make sure that that number of pods always exists. If a pod does crash, the Replication Controller replaces it.

Replication Controllers also provide other benefits, such as the ability to scale the number of pods, and to update or delete multiple pods with a single command.

You can create a Replication Controller with an imperative command, or declaratively, from a file.  For example, create a new file called rc.yaml and add the following text:


### ReplicaSet
https://www.mirantis.com/blog/kubernetes-replication-controller-replica-set-and-deployments-understanding-replication-options/
