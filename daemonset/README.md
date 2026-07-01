# Summary related to Daemonset

A DaemonSet ensures that all (or some) nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed, those Pods are garbage collected. Deleting a DaemonSet cleans up the Pods it created.

## Key Characteristics:

- One Pod per Node: Automatically schedules exactly one Pod on each eligible node
- No Replica Count: Unlike Deployments, DaemonSets don't use replicas field
- Automatic Scheduling: New nodes automatically get DaemonSet Pods
- Node Selector Support: Can target specific nodes using nodeSelector or affinity
- Toleration Support: Can run on tainted nodes (like control planes)

## Real-World Use Cases

1. Monitoring and Logging

- Deploy Prometheus Node Exporter on all nodes
- Run Fluentd or Filebeat for log collection
- Install Datadog, New Relic agents cluster-wide
- Collect metrics from every node

2. Network and Storage

- Deploy CNI network plugins (Calico, Weave, Cilium)
- Run storage plugins (Ceph, GlusterFS clients)
- Install load balancer components (MetalLB speakers)
- Network policy enforcement agents

3. Security and Compliance

- Deploy security scanning tools (Falco, Sysdig)
- Run vulnerability scanners on each node
- Install compliance monitoring agents
- Implement runtime security tools

4. Cluster Services

- kube-proxy (Kubernetes networking)
- DNS caching services
- Service mesh data plane (Istio, Linkerd)
- Ingress controller components

5. Performance and Optimization

- Deploy GPU drivers on GPU nodes
- Install node tuning daemons
- Run cache optimization tools
- System resource managers

## Labs:-
- [Daemonset with resources request](https://killercoda.com/cka-mock-practice/scenario/create-daemonset)