# Summary related to manual Scheduling

```javascript
- For manual scheduling, nodename under spec field is used to create pod under that node name. It is different from nodeselectorterms which are used in nodeaffinity.
- To create a static pod, just ssh into the nodes in production or docker exec in local. Cd into /etc/kubernetes/manifests and create a new yaml file for a pod and exit from that node.
- kubelet will create the pod automatically and to remove that pod, you have to remove the files from /etc/kubernetes/manifests/file.yaml
```