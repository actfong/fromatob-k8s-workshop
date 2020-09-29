## Setting the scene

### Create a Cluster on Google Kubernetes Engine (GKE)

So we don't accidentally interact with one of the production or testing cluster, we will run all of our commands inside of a Docker container. Please start it by running:
```
docker run -ti quay.io/stephanlindauer/k8s-maintenance:fromatob
```

We are using a CLI called `gcloud` to interact with the Google Cloud API. Please authenticate with your `@fromAtoB.com` account by running:
```
gcloud auth login
```

We have a dedicated project for this workshop so we have a playground where don't have to worry damaging other systems.
```
gcloud config set project a2b-dev-stephan  
```
Set `compute/zone` have to specify them with every command.
`europe-west1` means our infrastructure will live in St. Ghislain, Belgium.
```
gcloud config set compute/zone europe-west1-b
```
Now `gcloud` is pointing here:
![Google datacenter Belgium](https://storage.googleapis.com/gweb-uniblog-publish-prod/images/StG.max-1000x1000.jpg "Google data center Belgium")

At this point our tools are configured. Let's create infrastructure!

Start by picking a name for your cluster. I suggest:
```
CLUSTER_NAME=$(gcloud config list account --format "value(core.account)" | cut -d'@' -f1 | tr -d .)
```
Happy with it?
```
echo $CLUSTER_NAME
```

Let's spin up a cluster:
```
gcloud container clusters create $CLUSTER_NAME --machine-type "g1-small"

# no need to set this default:
# --num-nodes "3"
```

This might actually take some time. We can use it to explore the Google Cloud Console. Please go to <https://console.cloud.google.com/home/dashboard?project=a2b-dev-stephan> and try to find the page that shows your cluster. Can you also find the instances that are part of your cluster?

Is your cluster ready? Let's talk to it! Configure `kubectl` to interact with the Kubernetes API of your cluster.  Run:
```
gcloud container clusters get-credentials $CLUSTER_NAME
```

Can you see the resources defined at your Kubernetes API?
```
kubectl get nodes
```
```
kubectl get all
```

For future use of `kubectl` and `gcloud` within the company please be aware what context you are working in. To find out how those two tools are configured run the following commands:
```
gcloud config configurations list
```
```
kubectl config current-context
```

### Power usage ;)
- Use `watch` to monitor for changes with `kubectl`
- Set `alias k=kubectl` to save some keystrokes

### What you have learned in this section

1. Wire your `gcloud` client to the correct Google Cloud project
2. How to create a container-cluster on Google Kubernetes Engine (a.k.a. **GKE**)
3. Wire your `kubectl` client to the correct GKE cluster

