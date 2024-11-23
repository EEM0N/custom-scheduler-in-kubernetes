# Custom Kubernetes Scheduler

## Service Account

The custom Kubernetes scheduler requires a **ServiceAccount** to interact with the Kubernetes API. The ServiceAccount is created in the `kube-system` namespace, allowing the scheduler to perform operations such as scheduling pods and managing resources within the cluster.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-k8s-scheduler
  namespace: kube-system

## ClusterRole

The **ClusterRole** for the custom Kubernetes scheduler defines the permissions required to interact with various Kubernetes resources. This role allows the scheduler to perform actions such as:

- Managing events, leases, and endpoints.
- Accessing nodes, pods, and other resources to schedule workloads.
- Interacting with resources like `replicasets`, `statefulsets`, and `services` to manage the cluster effectively.

The ClusterRole is designed to ensure the custom scheduler has the necessary privileges to perform its scheduling and management tasks.

### Example ClusterRole Definition

```yaml
# ClusterRole example for the custom-k8s-scheduler
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-k8s-scheduler
rules:
  - apiGroups: [""]
    resources:
      - events
    verbs: ["create", "patch", "update"]
  # Additional rules for other resources (nodes, pods, leases, etc.)
