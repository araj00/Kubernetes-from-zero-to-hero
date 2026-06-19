# Summary related to services

There are 4 types of services in kubernetes:-

1. ClusterIP - Used in internal communication like pod to pod communicate on same node
2. NodePort - Used to call service from external sources. User need to have the specific ip address of the node to call.
3. LoadBalancer - Used to balance the request from the outside world to a healthy pod running in a node under the nodeport service
4. ExternalName - Used to call external service from pod without knowing exact address of the resource like database connection from applications running in a pod