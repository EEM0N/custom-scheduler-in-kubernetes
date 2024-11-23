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

## Custom Scheduler Configuration

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
