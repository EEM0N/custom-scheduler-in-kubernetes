# Custom Kubernetes Scheduler

## Service Account

The custom Kubernetes scheduler requires a **ServiceAccount** to interact with the Kubernetes API. The ServiceAccount is created in the `kube-system` namespace, allowing the scheduler to perform operations such as scheduling pods and managing resources within the cluster.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-k8s-scheduler
  namespace: kube-system
