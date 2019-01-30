## Chapter 3 : Manage Deployments 

This chapter will cover below topics:

* Hello world deployment on kubernetes
* Troubleshoot application 
* Expose your application
* Scale up and Scale down your app
* Update, Rolling updates and Rollback your existing deployment

### Hello world deployment on kubernetes

Create deployment hello-kubernetes.yaml file. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes
  template:
    metadata:
      labels:
        app: hello-kubernetes
    spec:
      containers:
      - name: hello-kubernetes
        image: aveevadevopsr/hello-kubernetes:1.5
        ports:
        - containerPort: 8080
```

Once file is created. Deploy application and expose it publicly using service given below. Read more about service in later part of this chapter.

```
kubectl apply -f hello-kubernetes_service.yaml
```

Service Yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes
```

Validate application:
```
kubectl get pods
```

This will show output as:
```
bash-3.2$ kubectl get pods
NAME                                READY   STATUS             RESTARTS   AGE
hello-kubernetes-7bf6fbdb57-8ct4z   1/1     Running            0          10m
hello-kubernetes-7bf6fbdb57-8d6t5   1/1     Running            0          10m
hello-kubernetes-7bf6fbdb57-h5v8t   1/1     Running            0          10m

### Troubleshoot application 

Below commands will help to troubleshoot app running on kubernetes:

```

Get logs of a specific pod

```
kubectl logs ${POD_NAME}

```

Get status of a single pod

```
kubectl get -w po ${POD_NAME}
```

Get Complete details about a POD_NAME

```
kubectl describe pods ${POD_NAME}
```

Get Name of container running inside the POD_NAME

```
kubectl get pod ${POD_NAME} -o jsonpath='{.spec.containers[*].name}'
```

Get logs of a specific container running inside pod

```
kubectl logs ${POD_NAME} ${CONTAINER_NAME}
```

If your container has previously crashed, you can access the previous container’s crash log with:

```
kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
```

Debug inside container running in pod:

```
kubectl exec ${POD_NAME} -c ${CONTAINER_NAME} -- ${CMD} ${ARG1} ${ARG2} ... ${ARGN}
Note: -c ${CONTAINER_NAME} is optional. You can omit it for pods that only contain a single container.
For example : kubectl exec ${POD_NAME} -c ${CONTAINER_NAME} -- ls -ltr /tmp
kubectl exec -it hello-world-deployment-677c9f4789-64lfn -c hello-world -- bash
```

Login inside a running container inside pod:

```
Kubectl exec -it ${POD_NAME} -c ${CONTAINER_NAME} -- bash
```

### Expose your application using service

Applications/pods inside a cluster can be exposed using service. Read https://cloud.google.com/kubernetes-engine/docs/concepts/service for more details about service and its types. Here we will use service type "LoadBalancer" to expose Hello-world using an aws LoadBalancer: 

Below YAMl file explains service, which can be deployed to expose the application "hello-kubernetes":

```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes
```
In above example:

* Deployment kind is "service"
* Service type is "LoadBalancer"
* LabelSelector is " app: hello-kubernetes"

Deploy this service using:
```
kubectl apply -f hello-kuberenets-service.yaml
```

This will expose deployed application using AWS ELB. Find out endpoint url using:
```
Kubectl get service
```

### Scale up and Scale down your app:

Scaling is accomplished by changing the number of replicas in a Deployment.
Scaling out a Deployment will ensure new Pods are created and scheduled to Nodes with available resources. Scaling will increase the number of Pods to the new desired state. Kubernetes also supports autoscaling of Pods, but it is outside of the scope of this tutorial. Scaling to zero is also possible, and it will terminate all Pods of the specified Deployment.
Running multiple instances of an application will require a way to distribute the traffic to all of them. Services have an integrated load-balancer that will distribute network traffic to all Pods of an exposed Deployment. Services will monitor continuously the running Pods using endpoints, to ensure the traffic is sent only to available Pods.
Scaling is accomplished by changing the number of replicas in a Deployment.
Once you have multiple instances of an Application running, you would be able to do Rolling updates without downtime. We'll cover that in the next module. Now, let's go to the online terminal and scale our application.

Update number of replicas from 3 to 5 deploy the application using. Update hello-kubernetes.yml and change number of replicas from 3 to 5. Deploy updated changes
```
Kubectl apply -f hello-kubernete.yaml 
```

Above script will increase pods from 3 to 5 replicas (pods). Lets try to delete one of POD and see if it comes automatically:

```
kubectl get pods
Kubectl delete pod ${POD_NAME}
```
We can see number of pods after some time, and number of pods will be same as before. Replication controller brought third pod back to previous state.

Manually Scale up existing deployment
```
Kubectl scale --replicas=6 -f 



.yaml
```

Manually Scale down 
```
Kubectl scale --replicas=1 -f hello-kubernetes.yaml
```
You can only horizontally scale applications if they are stateless (No local data and session storage).
Data is stored on persistent volumes or inside a database.

### Rolling updates and Rollback your existing deployment
Rolling updates allow Deployments’ update to take place with zero downtime by incrementally updating Pods instances with new ones. Rolling updates would not be enabled by default in Kubernetes. We need to configure rolling update and rolling strategy to make zero downtime deployments.

Users expect applications to be available all the time and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates. Rolling updates allow Deployments’ update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

Let’s take a simple deployment manifest.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes
  template:
    metadata:
      labels:
        app: hello-kubernetes
    spec:
      containers:
      - name: hello-kubernetes
        image: aveevadevopsr/hello-kubernetes:1.5
        ports:
        - containerPort: 8080
  ```
  
  This should work fine when you execute the following command and a deployment called hello-kubernetes will be created.
  
  ```
  kubectl apply -f hello-kubernetes.yml
  ```
