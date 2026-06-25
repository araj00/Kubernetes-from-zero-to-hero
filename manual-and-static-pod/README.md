# Summary related to manual Scheduling and static pod

## Manual Scheduling

```javascript
- For manual scheduling, nodename under spec field is used to create pod under that node name. It is different from nodeselectorterms which are used in nodeaffinity.
```

## Static Pod

```javascript
- To create a static pod, just ssh into the nodes in production or docker exec in local. Cd into /etc/kubernetes/manifests and create a new yaml file for a pod and exit from that node.
- kubelet will create the pod automatically and to remove that pod, you have to remove the files from /etc/kubernetes/manifests/file.yaml.
- Static pods are managed directly by kubelet rather than kube-apiserver
```

## Important Concepts

### Mirror Pods:-

When kubelet creates a static pod, it also creates a "mirror pod" in the Kubernetes API:

```javascript
- Allows you to see the static pod via kubectl get pods
- Mirror pod is read-only - you cannot modify or delete it via API
- Mirror pod reflects the state of the actual static pod
- Named as: <pod-name>-<node-name>
```
