# Lab 2 — Running a React App on a 3-Node Kubernetes Cluster

## 1. My Image

Docker Hub image deployed:

`hellofrom07/hello-app:v1`

The Docker image was built locally, tested successfully and pushed to Docker Hub.

## 2. Pods Across Nodes

After applying `hello-app.yaml`, four Pods were running successfully.

The Pods were distributed across two worker nodes:

- `desktop-worker` — 2 Pods
- `desktop-worker2` — 2 Pods

All four Pods showed `1/1 Running` status.

## 3. The Service and Its Endpoints

The application was exposed using a LoadBalancer Service.

Service details:

- Service name: `hello-app`
- Type: `LoadBalancer`
- Cluster IP: `10.96.94.46`
- External IP: `172.18.0.7`
- Service port: `8080`
- Target port: `80`

The EndpointSlice showed all four Pod IP addresses registered on port 80.

## 4. Browser

The application was successfully accessed at:

`http://localhost:8080`

The browser displayed the message "Hello from Kubernetes" and showed responses from different Pod hostnames.

The screenshot is saved as `screenshot.png` in the repository root.

## 5. The Four Traps

### Trap 1 — The Local Image

The local image `hello-app:v1` worked in my Docker Desktop Kubernetes cluster because it uses a shared containerd image store. In a typical multi-node cluster, nodes may not have the image locally, which can cause `ImagePullBackOff` if the image is not available from a registry or loaded onto the nodes.

### Trap 2 — The Selector Typo

I changed the Service selector from `app: hello-app` to `app: helo-app`. The EndpointSlice showed no endpoints because the Service could not find matching Pods. After correcting the selector, all four Pod endpoints appeared again.

### Trap 3 — The Wrong targetPort

I changed `targetPort` from 80 to 8080. The EndpointSlice still showed all four Pods, but the application stopped responding because Nginx listens on port 80. Restoring `targetPort: 80` fixed the problem.

### Trap 4 — The EXTERNAL-IP Address

The Service displayed `172.18.0.7` as its EXTERNAL-IP. In my Docker Desktop Kubernetes environment, I accessed the application through `http://localhost:8080` instead of using that IP address.

## 6. Your YAML vs the Generated YAML

Two fields present in the generated Deployment YAML but absent from my YAML file are:

1. `strategy: {}` — Kubernetes uses its default Deployment update strategy when this field is omitted.
2. `status: {}` — Kubernetes manages the resource status automatically, so it does not need to be specified in the configuration file.

My YAML works without these fields because Kubernetes supplies the defaults and manages the resource status.
