# kubernetes-basics-course

This is an entry level course to learn the basics of Kubernetes. Here you can find exercises and notes made during the course.

# Concepts

These are some of the basic concepts of Kubernetes with their explanation and main commands.

## Nodes

Are the machines (physical or virtual) on which Kubernetes runs.

### Components

- Kubelet: An agent that runs on each node in the cluster and ensures that containers are running in a pod.
- Kube-proxy: Handles network proxying and load balancing.
- Container runtime: The software responsible for running containers (e.g., Docker, containerd).

### Types of Nodes

- Master Nodes: These manage the Kubernetes cluster and its workload. They run the control plane components like the API server, scheduler, and controller manager.
- Worker Nodes: These are the nodes where the application workloads (pods) are actually run.

### Commands

- Get nodes

```
kubectl get nodes
```

## Namespaces

Provide a way to divide cluster resources and manage different environments or projects within the same cluster.

### Use Cases

- Resource Quota Management: Ensuring teams or projects don't exceed their allocated resources.
- Security: Isolating resources to enhance security.
- Organization: Helping manage and organize objects in the cluster.

### Default Namespaces

- default: The default namespace for objects without a namespace.
- kube-system: The namespace for objects created by the Kubernetes system.
- kube-public: A namespace that is readable by all users (including those not authenticated). It is mostly reserved for cluster usage.

### Commands

- Get namespaces

```
kubectl get namespaces
```

- Get specific namespace

```
kubectl get namespaces namespace-name
```

- Get namespaces by label

```
kubectl get namespaces -l name=namespace-1
```

- Create namespace using manifest

```
kubectl create -f namespace-1.yaml
```

- Create namespace in the terminal without manifest

```
kubectl create namespace namespace-2
```

- Describe namespace

```
kubectl describe namespaces namespace-1
```

- Annotating an existing namespace

```
kubectl annotate --overwrite namespace namespace-name annotation-key=annotation-value
```

- Change current context to specific namespace

```
kubectl config set-context --current --namespace=namespace-1
```

## Pods

Are the smallest unit of deployment in Kubernetes, encapsulating one or more containers with shared storage/networking and a specification for how to run them.

### Characteristics

- One or more containers: A pod can contain one or more containers (usually one). These containers share the same network namespace, including the IP address and port space.
- Shared Storage and Networking: Containers in a pod share storage volumes and networking.
- Lifecycle: Pods have a lifecycle; they are created, start, and eventually terminate.

### Common Use Cases

- Single Container Pods: Most pods contain a single container. This is the most common use case.
- Multi-Container Pods: Sometimes, a pod contains multiple containers that need to work together, sharing resources and information.

### Commands

- Get pods in the current namespace

```
kubectl get pods
```

- See all the pods across all `namespaces` you will need to pass the `-A` flag:

```
kubectl get pods -A
```

- Get pods from specific namespace

```
kubectl get pods --namespace=namespace-1
```

- create the pod:

```
kubectl create -f pod-1.yaml
```

- Create a pod from terminal

```
kubectl run pod-2 --image=nginx:1.14.2 --namespace=namespace-2
```

- Describe a pod

```
kubectl describe pod pod-1 --namespace=namespace-1
```

- Update an existing pod with changes in the yaml file

```
kubectl apply -f pod-1.yaml
```

- Delete a pod

```
kubectl delete pod pod-2 --namespace=namespace-2
```

## ReplicaSets

Are responsible for maintaining a stable set of replica Pods running at any given time. Their primary purpose is to ensure that a specified number of pod replicas are running at all times.

### How it works

- You define the desired state in a ReplicaSet manifest (YAML or JSON file), including the number of replicas.
- The ReplicaSet controller watches the current state of the pods and, if necessary, adds or removes pods to match the desired state.

### Use cases

- Ensuring high availability by maintaining multiple replicas of a pod.
- Scaling applications up or down by changing the number of replica.

### Commands

- Create the ReplicaSet:

```
kubectl create -f replicaset-1.yaml
```

- Delete a ReplicaSet:

```
kubectl delete replicasets replicaset-1
```

## Deployments

Are a higher-level abstraction that manages ReplicaSets and provides declarative updates to applications. Deployments offer additional features and capabilities beyond what ReplicaSets can provide on their own.

### How it works

- You define the desired state of your application, including the number of replicas, the container image, and update strategy, in a Deployment manifest.
- The Deployment controller creates or updates the ReplicaSet to match the desired state.
- It supports rolling updates and rollbacks, making it easier to manage application updates with minimal downtime.

### Use cases

- Performing rolling updates to gradually replace old pods with new ones.
- Rolling back to a previous version in case of a failed update.
- Automatically scaling applications based on load.

### Commands

- Create a deployment:

```
kubectl create -f deployment-1.yaml
```

- Describe a deployment:

```
kubectl describe deployments deployment-1
```

- Rolling Out an Update to a Deployment

Ex. This image is that we set before as image: nginx:1.14.2, we are going to want to update that.

```
kubectl set image deployment/deployment-1 nginx=nginx:1.16.1
```

- See the deployment history:

```
kubectl rollout history deployment/deployment-1
```

- Roll back last change in the history of deployment:

```
kubectl rollout undo deployment/deployment-1
```

## Service

A Kubernetes Service is an abstraction that defines a logical set of Pods and a policy by which to access them. Services enable communication between different components of an application or with external clients. Here are key points about services:

### Types of Services

- ClusterIP: Exposes the service on an internal IP in the cluster. This type makes the service only reachable from within the cluster.
- NodePort: Exposes the service on each Node’s IP at a static port. A NodePort service makes the service accessible from outside the cluster using `<NodeIP>:<NodePort>`.
- LoadBalancer: Exposes the service externally using a cloud provider’s load balancer. This service type automatically creates a NodePort and a ClusterIP service and assigns a public IP to the load balancer.
- ExternalName: Maps the service to the contents of the externalName field by returning a CNAME record with its value. It doesn’t create any proxying or forwarding rules.

### Components

- Selector: Specifies the Pods targeted by the service.
- Endpoints: A set of IP addresses and ports representing the Pods targeted by the service.
- Service Discovery: Kubernetes supports DNS-based service discovery. A DNS entry is created for each service, allowing Pods to communicate with services using their DNS names.

### Example

In order to test the service creation we first created a simple application "hello-world" that can be found in the directory with the same
name where we created a Dockerfile and a index.html. The Dockerfile only use a nginx image and copy the html. To create the image the
following command was used.

```
nerdctl --namespace k8s.io build --tag hello-world:latest .
```

Then, a hello-world-deployment.yaml file was created to specify the fields of the app deployment. Check the file for more details. Note that
the spec.containers[0].image was set to hello-world:latest, which is the tag of the image created in the previous step. To create the
deployment the following command was used:

```
kubectl create -f deployment.yaml
```

With the output looking something like this:

deployment.apps/hello-world-deployment created

When you do not specify which type of service you want to expose, by default, Kubernetes uses the ClusterIP service type,
to expose your deployment.

We use the declarative method of exposing our deployment by executing the below command:

```
kubectl expose deployment hello-world-deployment --name hello-world-service --port=8080 --target-port=80 -n namespace-1
```

Here, since we've not specified the type of Service, Kubernetes will choose the ClusterIP Service Type.

If the command is successful, you should see the output: `service/hello-world-service exposed`

You can check the details of the service you just created by executing the command,

```
kubectl describe service hello-world-service -n namespace-1
```

## Ingress

Ingress is a Kubernetes resource that manages external access to services within a cluster, typically HTTP and HTTPS. Ingress provides a way to define rules for routing traffic to the services based on various conditions, such as the request path or host. Here are key points about ingress:

### Ingress Controller

An ingress controller is responsible for fulfilling the ingress rules. It’s not a part of the Kubernetes core but must be deployed to manage ingress resources. Popular ingress controllers include NGINX, HAProxy, Traefik, and others.

### Ingress Resource

- Host: Specifies the host (domain) for which the rules apply.
- Paths: Defines the paths and their corresponding backend services. Each path specifies the service name and port to forward the request to.
- TLS: Provides SSL/TLS termination for secure connections. It defines the secret containing the TLS certificate and key.

### Example

Continuing with the service example, what if we wanted this service to be accessible to us externally?

Kubernetes offers you Ingress controllers, so that you can safely expose it to your customer, without them (or you) having to remember the port number the service was exposed on.

For that, we're first going to setup an NGINX Ingress Controller by executing the below command,

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/cloud/deploy.yaml
```

Wait for the ingress pods to come up and running.

```
kubectl get pods --namespace=ingress-nginx
```

We will now create an ingress resource declaratively that maps to `<your_hostname>.127.0.0.1.sslip.io`

```
kubectl create ingress hello-world-ingress --class=nginx --rule="<your_hostname>.127.0.0.1.sslip.io/*=hello-world-service:80" -n namespace-1
```

This can also be created using a yaml file as below,

```
kubectl apply -f ingress.yaml -n namespace-1
```

You should see the output: `ingress.networking.k8s.io/demo-localhost created`

Forward a local port to the ingress controller.

```
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8081:80
```

You should now see Hello World!!! displayed on your browser when you access the link, http://<your_hostname>.127.0.0.1.sslip.io:8081
