[Kind](https://sigs.k8s.io/kind) (or Kubernetes in Docker) is one of the lightweight solutions for running your own Kubernetes cluster locally.

There are many ways to install Kind. In this guide, we’ll focus on the `go install` option. For other installation methods, refer to the [Kind installation options](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

---

### **Prerequisites**

1. [Go](https://golang.org/) 1.17 or later
2. [Docker](https://www.docker.com/) or [Podman](https://podman.io/)
3. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

---

### **Installation & Usage**

#### Installing with `go install`

```sh
$ go install sigs.k8s.io/kind@v0.31.0
```

> **Note:**  
> When installing with Go, the Kind binary will be located in the `bin` directory under `$(go env GOPATH)`. If you encounter the error `kind: command not found`, update your `PATH` environment variable.

---

#### Creating a Cluster (Docker)

> Ensure Docker is up and running.

Create a simple cluster configuration file. Here, we’ll specify the number of nodes:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create the cluster:

```sh
$ kind create cluster --name=coconut --config=kind-config.yaml
```

> We use the `--name` and `--config` flags to override the default cluster name and specify a custom cluster configuration.

---

#### Listing Clusters

```sh
$ kind get clusters
coconut
```

---

#### Interacting with Your Cluster

```sh
$ kubectl cluster-info

Kubernetes control plane is running at https://127.0.0.1:44801
CoreDNS is running at https://127.0.0.1:44801/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

---

### **Deploy Your First App**

Once your cluster is up and running, let’s create a simple HTTP service and expose an Nginx server using an Ingress controller.

#### 1. Check Node Status

```sh
$ kubectl get nodes

NAME                    STATUS   ROLES           AGE   VERSION
coconut-control-plane   Ready    control-plane   11m   v1.35.0
coconut-worker          Ready    <none>          10m   v1.35.0
coconut-worker2         Ready    <none>          10m   v1.35.0
```

#### 2. Deploy the Nginx Ingress Controller

```sh
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

> In this configuration, Ingress acts as the cluster’s reverse proxy and load balancer. For more details, see the [Ingress documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/).

Wait for the Ingress controller pod to be running:

```sh
$ kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/name=ingress-nginx \
  --timeout=90s
```

Output:

```
pod/ingress-nginx-controller-56dc4b4c6-4622t condition met
```

#### 3. Deploy Your Test Application

Create a YAML manifest for Pods, Services, and Ingress:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
  - command:
    - /agnhost
    - serve-hostname
    - --http=true
    - --port=8080
    image: registry.k8s.io/e2e-test-images/agnhost:2.39
    name: foo-app
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
  - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
  - command:
    - /agnhost
    - serve-hostname
    - --http=true
    - --port=8080
    image: registry.k8s.io/e2e-test-images/agnhost:2.39
    name: bar-app
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
  - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: /foo
        backend:
          service:
            name: foo-service
            port:
              number: 8080
      - pathType: Prefix
        path: /bar
        backend:
          service:
            name: bar-service
            port:
              number: 8080
```

Apply the manifest:

```sh
$ kubectl apply -f echo-http.yaml
```

Verify the resources are created:

```sh
$ kubectl get pods,svc,ingress
```

#### 4. Access Your Ingress Locally

Forward the Ingress controller’s port 80 to your local machine:

```sh
$ kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
```

Test your routes in a new terminal:

```sh
$ curl http://localhost:8080/foo
$ curl http://localhost:8080/bar
```

> **Expected output:** The hostname of the `foo-app` or `bar-app` pod.

---

### **Troubleshooting**

- **Check Ingress controller logs:**
  ```sh
  $ kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
  ```
- **Check Ingress events:**
  ```sh
  $ kubectl describe ingress example-ingress
  ```
- **Verify services and pods:**
  ```sh
  $ kubectl get svc,ep
  ```
