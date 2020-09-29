## Quick and easy

### Hello World
For our Hello World exercise, we will deploy an app containing a webserver (nginx) in a quick, easy an very dirty fashion.
In the following chapters we will go more into detail and will look into setting things up properly.

GOALS:
- Able to create, inspect and delete K8s resources (Pods, ReplicaSets, Deployments, Services)
- Get some insight into cluster operations
- Verify that your application is running

### Deploy our app

Create a Deployment, containing a ReplicaSet (of 3 pods), each of them containing a nginx container.
```
kubectl run --image=nginx hello-nginx --port=80 --labels="app=hello" --replicas=3
```

Expose your Pods within the hello-nginx Deployment outside the cluster.
```
kubectl expose deployment hello-nginx --port=80 --name=hello-http --type=LoadBalancer
```

Grab the "LoadBalancer Ingress".
```
kubectl describe svc hello-http
```

If you look at the events at the bottom, one of the messages would be `Ensuring load balancer`.

Once you see the message `Ensured load balancer`, you can proceed by accessing your app through the LoadBalancer Ingress address above.

Congrats! You just deployed an app on Kubernetes!

The approach that we took to deploy is known as the **imperative** way (*imperative vs declarative*: more on that later).

By now, you should have created the following resources:
- 3 Pods (a.k.a. `po`)
- 1 ReplicaSet (a.k.a. `rs`)
- 1 Deployment (a.k.a. `deploy`)
- 1 Service (a.k.a. `svc`)

Please take a look at the resources (`deployment`, `replicasets`, `pods` and `services`) you just deployed with the following commands:

```
kubectl get {resource-type}
kubectl get {resource-type} -o json
kubectl describe {resource-type} {resource-name}
```

Once you have familiarized with the attributes of the resources, please remove them all by:
```
kubectl delete svc hello-http
kubectl delete deployment hello-nginx
```

### Questions to ask yourself
In this chapter, we deployed an app and exposed it by running some commands (`run` and `expose`). As mentioned, this way is known as the **imperative** way.

Now imagine if you were to deploy with more configurations (such as more labels, mount volumes or even multiple pods etc), how would your command look like?

### What you have learned in this section

1. Familiarize with the key resources within Kubernetes (Pods, ReplicaSets, Deployments, and Services)
2. Learned how to interact with K8s resources by kubectl `get`, `describe` and `delete`
3. Quickly deployed all these resources in one go with `kubectl run`, which is the `imperative way` of deployment.

This was a very high-level overview of Kubernetes. From here on, we will build your knowledge from the bottom up, starting by looking at Pods in the next section.
