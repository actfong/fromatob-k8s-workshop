## Part 2: Pods

### Concept ###
In Docker, the **atomic unit** of scheduling is a *container*. In Kubernetes, the atomic unit of scheduling is a *Pod*.

A Pod is actually an abstraction that contains one or more containers. Within a Pod, containers share the same network and storage resources.

<img src="images/k8s-pod.png" width="600" height="450"/>

The containers on a Pod are **co-located** and **co-scheduled**

*Co-located*: If a Pod contains multiple containers, all its containers will run on the same Node (VM that acts as a minion) instead of spread over multiple Nodes.

*Co-scheduled*: A Pod's status is only `running` when all its containers are up and running.

Like all resources in K8s, a Pod can be defined in a manifest file

### Manifest ###

As mentioned in part 1, the way we deployed by running `kubectl run` is known as the *imperative way*.
The imperative way has a few drawbacks:

- If you want to provide more customizations, the command becomes too long
- You can't keep track of changes with version control.
- The imperative way doesn't express the relationship between Pods / Replicas / Deployments

Therefore, in general, the *declarative way* is the preferred way to work with K8s.

The application that we will be working with is called *tasman* and its source code can be found [here](https://github.com/actfong/tasman). It is a Sinatra application running on port 4567.

Get the example manifest file by cloning this repository:
```
git clone https://github.com/fromatob/k8s-workshop.git
cd k8s-workshop/manifests
cat tasman-pod.yml
```
Have a look into how the Pod configuration is assembled.

Now we have two options for deploying the `tasman-pod.yml` file:
```
kubectl create -f {your-file.yml}               # creates a kubectl resource
# or
kubectl apply -f {your-file.yml}                # update/create a kubectl resource
# where -f indicates the path to the manifest file
```

Resources in K8s (such as Pods) can be created and/or updated based on a manifest file, as shown above.

While `create` only **creates** a new resource, `apply` **creates** or **updates** a resource, depending on whether it already exists. So you can see `apply` is the more universal command that we will use from now on.

Inspect the newly created Pod with:
```
kubectl get pods                               
kubectl describe pod {name-of-pod}
```

Once you look into the pod with `describe`, you should see the values provided with your manifest, such as `Name`, `Labels` and the specification for your `Containers`.
Please note that there are more fields than what you configured in your YAML file.

### I just deployed a Pod, now what? ###

Now that you have deployed a pod, you probably want to do "fun stuff" with it, e.g. hit your application with HTTP requests, scale it up and down , maybe do a rolling update etc.

So, knowing that your application runs on port 4567, how would you access it? And how could you scale it up?

Maybe using your Node's IP and attach `:4567` to it? And maybe we can use `kubectl scale` and pass a pod to it? How about you give it a try and come back in a few minutes?

Have a poke with `kubectl`. See which commands you can run. And you can always rely on `kubectl describe pod {pod-name}` to check the state of your pod.

<details>
<br/>
<summary>Possible Solution</summary>
Well, here is fun fact for you. You CAN'T do any of the above! :)

You can't access a Pod from outside the cluster. Neither can you scale a Pod.

That is also why when it comes to web-applications, <i>no one would deploy a Pod on its own</i>.

<br/>
<img src="images/seriously-who-does-that.jpg" width="400" height="400"/>
<br/>

<p>
Because of the limitations, a Pod is meant to be deployed with higher level constructs, such as <b>ReplicationController</b> or <b>Deployment</b>. (see the next sections)
</p>

<p>
I might have wasted a few minutes of your time, letting you type a manifest etc.... But at least, for the rest of your life, you will never deploy your application as a stand-alone <i>Pod</i>
</p>
</details>


### Anything else I could do with a Pod? ###

#### Executing commands ####
Sure, there is something you can do. How about running commands within your Pod?

From Docker, you might remember:
```
docker exec {options} {container-name} {command-name}
# e.g
# docker exec -it my-container /bin/bash
```

You could do the same with `kubectl`

```
kubectl exec -it {pod-name} {command}
kubectl exec -it {pod-name} -c {container-name} {command}         # for a multi-container pod
```

If it is a multi-container Pod, add the `-c` option and provide the name of the container, within which you would like to execute a command. The name of containers can be obtained with `kubectl describe pod {pod-name}` and look within the `containers` section.

##### Mini Challenge #####

With the info provided, could you get into our Pod's container(s) to run commands? Can you find out what processes are running in the container?

<details>
<summary>Possible Solution:</summary>

<pre>
kubectl exec -it {pod-name} -- /bin/sh
# Once you are in:
ps auxf
# Does the process list make sense to you?
</pre>

</details>


#### Access from within the cluster ####
As you may have tried, we couldn't send http-request to our application.

However, even though we can't access our Pod from outside the cluster (without a `Service` object, more on that later), we CAN access it from inside the cluster!

So how could we get inside the cluster, send a HTTP-request to our Pod and prove that our Pod is running?


First of all, when you ran `kubectl describe` on the Pod you just deployed, you should have seen its values for `IP` and `Container Port`. Please take note of these values.

Then, you might remember with `kubectl run` from the previous section, with which we could easily deploy a Pod.

So how about we deploy a very simple pod with one container, which is small in size and has `wget` installed. Then use that to hit our previously deployed application? [Busybox](https://hub.docker.com/_/busybox/) and [Linux Alpine](https://hub.docker.com/_/alpine/) would be good candidates for this job.


##### Mini Challenge 1 #####

With the info provided, could you send a HTTP request to the application that we just deployed using a `busybox` image?
Have a look at `kubectl run --help`. It might contain an example that you find useful.

<details>
<summary>Possible Solution:</summary>

<pre>
kubectl run -it busybox --image=busybox --restart=Never -- /bin/sh
# Once you are in the container
wget {application-ip}:{application-port}
</pre>

</details>


##### Mini Challenge 2 #####

As previously discussed, a `Node` is basically a VM where you deploy your containers to. In order to run these containers, it utilizes a container runtime (such as Docker) to pull images and run them as containers.

Are you able to SSH into your `Node` and prove to yourself that it has indeed pulled the required image and has it running as a container?

<details>
<summary>Possible Solution:</summary>

<pre>
# Get a list of pods and the nodes that they are running on:
kubectl get pods -o wide
<br>
# Then use `gcloud compute ssh` (You can check Google Cloud Console for the parameters)
gcloud compute ssh {user}@{node}
<br>
# Once you have ssh'ed into the Node:
docker images                           # should list the image you deployed
docker ps                               # should list the container(s) within the pod you deployed

Why are you seeing so many images and running containers though?
</pre>
</details>

---

### What you have learned in this section ###

In this section, you have learned:

1. Pods in K8s are the most atomic units. It contains one or more containers, which share resources within a Pod.
2. The structure of a Pod manifest
3. Create / update a K8s resource with `kubectl create` and/or `kubectl apply`
4. Execute commands within a Pod with `kubectl exec`
4. A Pod is by default only accessible from within the cluster


In the next section, we will have a look at the `Service` Object, which allows us to access our Pods from within and from outside the cluster.
