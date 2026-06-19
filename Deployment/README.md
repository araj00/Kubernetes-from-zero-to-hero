# Summary related to Deployments

## Useful commands:-

```
kubectl set image (rs,deployments) {obj_name} {container_name}={specified image name} --> To change the image in a deploymen and replicaset

kubectl annotate deployment {obj_name} kubernetes.io/change-cause="reason"

kubectl scale deployment {deploy_name} --replicas={desired number of replicas} ---> To scale the number of replicas in deployments
```

**NOTE: When you imperatively update the image for a pod managed by a ReplicationController (rc), the RC does not perform a rolling update. Instead, it retains the existing pods, and only newly created pods will use the updated image.
The same applies to ReplicaSets (rs) when used without a Deployment—existing pods are not replaced immediately, and only new pods will have the updated image.**