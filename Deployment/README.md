# Summary related to Deployments

## Useful commands:-

```
kubectl set image (rs,deployments) {obj_name} {container_name}={specified image name} --> To change the image in a deploymen and replicaset

kubectl annotate deployment {obj_name} kubernetes.io/change-cause="reason"

kubectl scale deployment {deploy_name} --replicas={desired number of replicas} ---> To scale the number of replicas in deployments
```