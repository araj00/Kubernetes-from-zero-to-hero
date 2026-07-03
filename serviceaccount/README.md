# Summary related to serviceaccount

## ServiceAccount Tokens (Pre-1.24 vs Post-1.24)

1. Before Kubernetes 1.24:

- Tokens were automatically created as Secrets
- Non-expiring by default (security concern)
- Manual cleanup required

2. After Kubernetes 1.24:

- Tokens are generated on-demand using TokenRequest API
- Time-limited by default (more secure)
- Created with kubectl create token command
- Use automountServiceAccountToken to false if you want to disable the auto mounting of service account in objects

## Useful Commands:-

kubectl create serviceaccount sa-name
kubectl create token sa-name --duration=3600s // token validity for 1 hour of given service account

## API Structure:-

The general structure for api is /<root>/<group>/<version>/namespaces/<namespace>/<resourceType>/<name>.

Examples:-

- GET /api/v1/namespaces/default/pods – List Pods (core group)
- POST /apis/apps/v1/namespaces/prod/deployments – Create a Deployment (apps group)
- GET /apis/rbac.authorization.k8s.io/v1/clusterroles/admin – Get a ClusterRole (non-namespaced)

## Labs:-

- [Configure RBAC and ServiceAccount Token Management](https://killercoda.com/cka-mock-practice/scenario/GitLab-CICD-RBAC)