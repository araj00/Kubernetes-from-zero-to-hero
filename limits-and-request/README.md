# Summary related to limit

1. To use kubectl top nodes, metrics server needs to be installed. CAdvisor runs on kubelet which in returns gives the logs of nodes and pods. Kubelet aggregates this data and later used by pull method of metrics-server. Push model is not used here

2. Limit, request are used for containers and not pods.
3. LimitRange is a resource type which creates the default rule for resource limit and request if not specified during deployment

4. In LimitRange, field limits under spec field is of list type, so that admin can create different limit range for different resource consumers like pod, container, persistentvolumeclaims 