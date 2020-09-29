## Part 6: Scaling your application

### Concept

From the previous sections, we have seen that a Pod can be managed by a RC, RS or a Deployment.

`RC` and `RS` have the sole purpose to ensure that a number of Pods are up, while a `Deployment` wraps around `RC`'s/`RS`'s and takes care of rollouts and rollbacks.

In K8s, if you want to scale the number of Pod-replica's, this message has to be sent to the object which manages your Pods (RC/RS/Deploy).

### Simple manual scaling

In case you want to scale up the number of pods, a simple way would be to use the `kubectl scale` command and refer to the resource (RC/RS/Deploy) that you want to scale:

```
# General syntax
kubectl scale --replicas={desired-number} {resource-type}/{resource-name}

# Scale up a deployment resource to a desired amount of replica's
kubectl scale --replicas={desired-number} deploy/{deploy-name}

# In case you want to scale up a RC rather than Deployment
kubectl scale --replicas={desired-number} rc/{rc-name}

# Scale up a deployment resource to a desired amount of replica's,
# only if the current-replica's amount is 2
kubectl scale --replicas={desired-number} --current-replicas=2 deploy/{deploy-name}
```

If you have deployed with the example manifest from the previous section, you should have 4 Pods running in your deployment. Could you scale it up to 6?

<details>
<summary>Possible Solutions:</summary>
<pre>
kubectl scale --replicas=6 deploy/tasman
# or
kubectl scale --replicas=6 --current-replicas=4 deploy/tasman
</pre>
</details>
<br/>

#### Inspect your resource after scaling
After scaling your Deployment, you can inspect your Deployment object with `kubectl describe` and pay attention to the `Events` section at the bottom. One of the messages should be that the ReplicaSet has been scaled.

The `Replicas` attribute should have been updated as well to your new desired number of replicas. You could also inspect the `RS` to see which `Events` have occurred after you perform the `kubectl scale` command.

And obviously, when you list pods with `kubectl get pods`, you should see the newly created Pods.

However, when the demand for your application grows, these objects (RC, RS and Deployment) are not capable to scale up the number of pods dynamically. For that, you need a `HorizontalPodAutoscaler` (a.k. `hpa`).

### Autoscale with HPA

The `HorizontalPodAutoscaler` (a.k.a. `HPA`) allows us to scale up/down the number of Pod-replicas based on CPU-utilization.

A `HPA` can be created with the `kubectl autoscale` command.

As mentioned before, to scale up/down, a message should be sent to objects like RC, RS or Deployment. So when you create a `HPA`, a reference to such resource must be provided. You should also specify the maximum number of pods that a HPA can apply.

In our case, a Deployment named *tasman* was the highest level construct that we have deployed (remember, it wraps around RS's which in trun wraps around Pods), so it makes sense for us to create a HPA that targets a Deployment resource.

The structure of a HPA for our deployment would look like this:
```yml
# tasman-hpa.yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: tasman
spec:
  scaleTargetRef:
    apiVersion: apps/v1beta1
    kind: Deployment
    name: tasman
  minReplicas: 3
  maxReplicas: 8
  targetCPUUtilizationPercentage: 85

```

Let's apply it with 

```
kubectl apply -f tasman-hpa.yaml
```

In the example above, we created a HPA that has a reference to the Deployment named "tasman".
From that moment, the [`controller manager`](https://kubernetes.io/docs/admin/kube-controller-manager/) in the K8s Master will continously observe CPU usage of Pods managed by this Deployment and scales up/down, in order to match the CPU-target that is set in the `HPA` object.

If you have followed all instructions, the amount of Pods should have gone from 6 to 3 by now. You can see that by looking at the `Events` in your Deployment with `kubectl describe deploy {deploy-name}`. Also pay attention that the `Replicas` attribute in your Deployment has changed.

And lastly, you can list the HPA's with `kubectl get hpa` and find out more with `kubectl describe hpa {hpa-name}`.

When you inspect with `kubectl describe hpa`, please pay attention to the attribtues `Reference` and `Metrics`.

What happens if we eventually run out of nodes?

---

### What you have learned in this section
1. Scale the number of Pods of a Deployment
2. Create a HorizontalPodAutoscaler 
3. A HPA allows you to scale your application dynamically, based on memory and CPU-utilization.

---

### The End

Yes, this is it! By now, you should be *dangerous* enough to create a cluster for your containerized application, deploy updates (or rollbacks), scale it and make it accessible to the rest of the Internet!

<img src="images/dangerous.jpeg" width="225" height="225"/>

A quick recap:

1. The most atomic unit in K8s is a Pod. A Pod can have one or more containers.
2. Service (`svc`) objects enable access to your Pods from outside your cluster.
3. ReplicationController (`rc`) and ReplicaSet (`rs`) are meant to keep a number of pod-replicas running.
4. `Deployment` takes care of rolling-updates and rollbacks.
5. Scaling can be done manually (`kubectl scale`) or automatically (through a HPA)

Further reading and fiddling:

- [12 factor apps](https://12factor.net/)
- [minikube](https://github.com/kubernetes/minikube)
- [kubectx](https://github.com/ahmetb/kubectx)
