## Chapter 4 : HealthCheck,Labels and PODS

This chapter will cover below topics:

* HealthCheck
* Labels
* Pods

### HealthCheck

With a few lines of YAML, you can turn your Kubernetes pods into auto healing wonders. 
The right combination of liveness and readiness probes used with Kubernetes deployments can:

```
    1. Enable zero downtime deploys
    2. Prevent deployment of broken images
    3. Ensure that failed containers are automatically restarted
```

#### Readiness probes
The kubelet uses readiness probes to know when is container is ready to start accepting traffic. 
A pod is considered ready when all of its containers are ready.
With readiness probes, Kubernetes will not send traffic to a pod until the probe is successful. 
When updating a deployment, it will also leave old replica(s) running until probes have been successful on new replica. 
That means that if your new pods are broken in some way, they’ll 
never see traffic, your old pods will continue to serve all traffic for the deployment

#### Liveness probes
In many cases, it makes sense to complement readiness probes with liveness probes. 
Despite the similarities, they actually function independently. 
While readiness probes take a more passive approach, liveness probes will actually attempt to restart a container* if it fails.

Here’s what this might look like in a real life failure scenario. Let’s say our API encounters a fatal exception when processing a request.

    1. Readiness probes fails
    2. Kubernetes stops sending traffic to the pod
    3. Liveness probes fails
    4. Due to liveness configuration, kubernetes starts the failed container.
    5. Readiness probes succeeds
    6. kubernetes starts sending traffic to the pod again

Lets take an example of yaml file with readiness probes and livenss probes

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
        livenessProbe:
         httpGet:
           path:/
           port: 8080
         initialDelaySeconds: 15
         timeoutSeconds: 30
         periodSeconds: 15
        readinessProbe:
         httpGet:
           path:/
           port: 8080
         initialDelaySeconds: 15
         timeoutSeconds: 30
         periodSeconds: 15

```

Pretty incredible, right? This is the kind of automated healing that makes Kubernetes incredible to work with.
All of the above is made possible with just a few lines of YAML.

##### Probes for HTTP Services

The most noticeable impact these probes can have will be on HTTP services. 
The readinessProbe and livenessProbe attributes should be placed at the same level 
as the name and image attributes for your containers in a deployment. 
For one of our services, the probe configuration looks something like:

```
    livenessProbe:
         httpGet:
           path:/
           port: 8080
         initialDelaySeconds: 15
         timeoutSeconds: 30
    readinessProbe:
         httpGet:
           path:/
           port: 8080
         initialDelaySeconds: 15
         timeoutSeconds: 30
```

The attributes are pretty straightforward here. The key ones to pay attention to are:
###### initialDelaySeconds 
How long to wait before sending a probe after a container starts. 
For liveness probes this should be safely longer than the time your app usually takes to start up. 
Without that, you could get stuck in a reboot loop. On the other hand, this value can be lower 
for readiness probes as you’ll likely want traffic to reach your new containers as soon as they’re ready.
###### timeoutSeconds 
How long a request can take to respond before it’s considered a failure. 
###### periodSeconds 
How often a probe will be sent. 
The value you set here depends on finding a balance between sending too many probes to your service or 
going too long without detecting a failure. 
In most cases we settle for a value between 10 and 20 seconds here.

##### Probes for Background Services
Although readiness and liveness probes are remarkably straightforward for services that expose an HTTP endpoint, 
it takes a bit more effort to probe background services. Here’s an example of probes we’re currently using for one of our background services:

```
readinessProbe:
  exec:
    command:
      - test
      - '`find alive.txt -mmin -1`'
  initialDelaySeconds: 5
  periodSeconds: 15
livenessProbe:
  exec:
    command:
      - test
      - '`find alive.txt -mmin -1`'
  initialDelaySeconds: 15
  periodSeconds: 15
```

Our background service touches alive.txt every 15 seconds, and the probes test to ensure that file has been modified within 1 minute. 
This ensures that not only is the service running, it’s continuing to function as expected.
The exec option for probes is incredibly powerful. 
We’re using a fairly simple command in the above example, but the sky’s the limit here. 
Each service is unique and this allows for a flexible way to ensure that things remain functional.
