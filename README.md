# Custom Kubernetes Scheduler

## Service Account

The custom Kubernetes scheduler requires a **ServiceAccount** to interact with the Kubernetes API. The ServiceAccount is created in the `kube-system` namespace, allowing the scheduler to perform operations such as scheduling pods and managing resources within the cluster.

## ClusterRole

The **ClusterRole** for the custom Kubernetes scheduler defines the permissions required to interact with various Kubernetes resources. This role allows the scheduler to perform actions such as:

- Managing events, leases, and endpoints.
- Accessing nodes, pods, and other resources to schedule workloads.
- Interacting with resources like `replicasets`, `statefulsets`, and `services` to manage the cluster effectively.

The ClusterRole is designed to ensure the custom scheduler has the necessary privileges to perform its scheduling and management tasks.

## ClusterRoleBinding

### 1. **ClusterRoleBinding for custom-k8s-scheduler as kube-scheduler**

This **ClusterRoleBinding** grants the **custom-k8s-scheduler** **ServiceAccount** the necessary permissions from the **custom-k8s-scheduler** ClusterRole. It allows the custom scheduler to interact with resources like nodes, pods, events, leases, and more, which is required for scheduling tasks and managing cluster resources effectively.

### 2. **ClusterRoleBinding for custom-k8s-scheduler as volume-scheduler**

This **ClusterRoleBinding** binds the **custom-k8s-scheduler** **ServiceAccount** to the **system:volume-scheduler** role. This grants the custom scheduler the necessary permissions to handle volume-related scheduling operations, ensuring the scheduler can make decisions about where volumes should be bound within the cluster.

## Custom Scheduler Configuration with **NodeResourcesFit** plugin

The **ConfigMap** `custom-k8s-scheduler-config` holds the configuration for the custom Kubernetes scheduler. This configuration defines the scheduler's behavior, including the resource allocation strategy and the leader election mechanism.

### Key Components

1. **Scheduler Profile:**
   - The configuration defines a **scheduler profile** where the **NodeResourcesFit** plugin is used to prioritize nodes based on available resources (CPU and memory). 
   - The **scoringStrategy** is configured to give equal weight (1) to **cpu** and **memory**, and the scheduler prefers nodes with the **most allocated resources**.

2. **Plugins:**
   - The **NodeResourcesFit** plugin is enabled in the **score** section, making the custom scheduler evaluate nodes based on their resource allocation.
   - The **multiPoint** plugin is also enabled to run multiple scheduling steps.

3. **Leader Election:**
   - The custom scheduler is configured for **leader election** within the `kube-system` namespace to ensure high availability and prevent multiple schedulers from running concurrently.

4. **Client Connection Settings:**
   - The **clientConnection** parameters define the connection settings, allowing up to **200 bursts** and **100 QPS** (queries per second) for interacting with the Kubernetes API.

### Usage
This ConfigMap is applied to the `kube-system` namespace, where the custom scheduler will use it to manage pod scheduling operations based on the defined resource allocation strategy and leader election configuration.

## Custom Scheduler Configuration with **CustomCPUUtilizationScorer** plugin

1. **Plugin Configuration:**
   - **CustomCPUUtilizationScorer Plugin:**
     - A custom scoring plugin named `CustomCPUUtilizationScorer` is defined, with a CPU utilization threshold of `0.7` (70%). This means the scheduler will prefer nodes where CPU utilization is below 70%.
     - The plugin is enabled with a `weight` of 1, meaning it will have a minimal impact on the scheduling decision.

2. **Plugins Section:**
   - The `score` plugin section enables the custom CPU utilization scorer plugin and disables all other score plugins by setting `disabled: "*"`.
   - The `multiPoint` plugin also enables the custom scorer, indicating that it will be used for multiple points of the scheduling process.

### Usage

This configuration enables a custom scheduler with a plugin for CPU utilization scoring. It is useful for scenarios where you want to prioritize nodes with lower CPU utilization. The configuration also includes leader election for high availability and client connection settings to manage API request rates.

## Custom Kubernetes Scheduler Deployment

The **Deployment** configuration for the custom Kubernetes scheduler ensures that the scheduler is deployed and managed as a Kubernetes pod with the necessary configurations. This Deployment runs the custom scheduler with specific settings and parameters.

### Key Components

1. **Metadata:**
   - The `name` of the deployment is `custom-k8s-scheduler`, and it is deployed within the `kube-system` namespace. Labels are used to classify the scheduler under the `control-plane` tier and `scheduler` component.

2. **Replicas:**
   - The deployment defines **2 replicas** to ensure high availability. This means two instances of the custom scheduler are running simultaneously, providing fault tolerance and load balancing.

3. **Affinity and Anti-Affinity:**
   - **Node Affinity:** The scheduler avoids nodes that are labeled with `karpenter.sh/nodepool`, ensuring that the custom scheduler does not run on these nodes.
   - **Pod Anti-Affinity:** Ensures that the custom scheduler pods are scheduled on different nodes to avoid multiple schedulers being scheduled on the same node.

4. **Security Context:**
   - The pod runs as a non-root user (`runAsNonRoot: true`) to enhance security.
   - The security context also prevents privilege escalation and limits the privileges the container has (e.g., dropping all capabilities and disallowing running as root).

5. **Service Account:**
   - The scheduler uses the `custom-k8s-scheduler` service account to interact with the Kubernetes API.

6. **Container Configuration:**
   - The container runs the **kube-scheduler** binary, with flags to bind to all IP addresses (`--bind-address=0.0.0.0`), specify the configuration file (`--config`), and set the verbosity level to 5 (`--v=5`).
   - The scheduler image used is `registry.k8s.io/kube-scheduler:v1.29.11`.
   - **Probes:** 
     - **Liveness Probe:** Ensures that the scheduler is running properly by checking the `/healthz` endpoint.
     - **Readiness Probe:** Ensures that the scheduler is ready to receive traffic, also by checking the `/healthz` endpoint.
   
7. **Resources:**
   - **Requests and Limits:** Defines CPU and memory resource requests and limits for the custom scheduler container:
     - **CPU:** 500m
     - **Memory:** 512Mi
   - These values ensure that the scheduler container is allocated sufficient resources to operate correctly while preventing resource overuse.

8. **Volume Mount:**
   - The configuration file for the custom scheduler (`custom-k8s-scheduler-config.yaml`) is mounted as a volume under `/etc/kubernetes/custom-k8s-scheduler` using a **ConfigMap**. This allows the scheduler to use the custom configuration during its operation.

9. **Host Network and PID:**
   - The deployment does not use the host network or PID namespace (`hostNetwork: false`, `hostPID: false`), ensuring that the pod is isolated from the host network and processes.

### Usage

This Deployment creates and manages the custom Kubernetes scheduler with the required configuration, ensuring high availability, security, and correct operation of the scheduling process in the cluster.

## Sample Pod Using Custom Kubernetes Scheduler

This **Pod** configuration demonstrates how to schedule a pod using the **custom Kubernetes scheduler**.

### Key Components

**Scheduler:**
   - The `schedulerName` is set to `custom-k8s-scheduler`, meaning this pod will be scheduled by the custom scheduler rather than the default Kubernetes scheduler.

### Usage

This YAML creates a pod that is managed by the custom scheduler. The pod runs an Nginx container, and the custom scheduler is specifically assigned to this pod through the `schedulerName` field, which overrides the default scheduler for this pod.

# Verification and Status Check

After deploying the custom Kubernetes scheduler, use the following commands to verify the setup. Below are the actual command outputs to ensure the custom scheduler is running correctly and the cluster is operational.

### Checking Node Status

Run the following command to check the status of the nodes in the cluster:

```bash
vagrant@master-node:~/custom-scheduler-in-kubernetes$ kubectl get nodes -o wide
NAME            STATUS   ROLES           AGE     VERSION    INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master-node     Ready    control-plane   4h48m   v1.29.11   192.168.56.10   <none>        Ubuntu 22.04.4 LTS   5.15.0-102-generic   containerd://1.7.23
worker-node01   Ready    <none>          4h44m   v1.29.11   192.168.56.11   <none>        Ubuntu 22.04.4 LTS   5.15.0-102-generic   containerd://1.7.23
worker-node02   Ready    <none>          4h44m   v1.29.11   192.168.56.12   <none>        Ubuntu 22.04.4 LTS   5.15.0-102-generic   containerd://1.7.23
worker-node03   Ready    <none>          4h44m   v1.29.11   192.168.56.13   <none>        Ubuntu 22.04.4 LTS   5.15.0-102-generic   containerd://1.7.23
vagrant@master-node:~/custom-scheduler-in-kubernetes$ kubectl get pods -n kube-system -o wide | grep custom
custom-k8s-scheduler-5dbf95d5ff-ftkfj   1/1     Running   0               114m    172.168.158.7    worker-node02   <none>           <none>
custom-k8s-scheduler-5dbf95d5ff-x6bng   1/1     Running   0               114m    172.168.77.140   master-node     <none>           <none>
vagrant@master-node:~/custom-scheduler-in-kubernetes$
