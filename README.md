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
