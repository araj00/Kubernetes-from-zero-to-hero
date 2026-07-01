# Summary related to PersistentVolume and PersistentVolumeClaim

## When to use PersistentVolumes:-

- Database storage: MySQL, PostgreSQL, MongoDB data directories
- File uploads: User-generated content that needs to persist
- Log aggregation: Centralized logging with persistent storage
- Static assets: Web server content that survives pod restarts
- Shared storage: Multiple pods reading from the same volume (ReadWriteMany)

## Storage Types:-

- Local volumes: High performance, node-specific (like this scenario)
- NFS: Shared network storage, supports ReadWriteMany
- Cloud volumes: AWS EBS, Google Persistent Disk, Azure Disk
- CSI drivers: Container Storage Interface for vendor-specific storage

## Access Modes:-

- ReadWriteOnce (RWO): Single node can mount as read-write
- ReadOnlyMany (ROX): Multiple nodes can mount as read-only
- ReadWriteMany (RWX): Multiple nodes can mount as read-write

## Key Takeaways:

- PVCs abstract storage: Developers request storage via PVC without knowing underlying details
- Binding is automatic: Kubernetes matches PVCs to suitable PVs based on requirements
- Local volumes have restrictions: Pods are constrained to specific nodes
- Volume lifecycle: PVC deletion behavior depends on persistentVolumeReclaimPolicy (Delete, Retain, Recycle)
- Volume mounts: Each container can mount multiple volumes at different paths

## Labs:-

- [PersistentVolumeClaim with Dynamic Provisioning](https://killercoda.com/cka-mock-practice/scenario/PersistentVolumeClaim-Dynamic)
- [PersistentVolumeClaim - Mount Storage to Nginx Deployment](https://killercoda.com/cka-mock-practice/scenario/PersistentVolumeClaim-Mount-Storage)
- [Restore MySql with persistent data](https://killercoda.com/cka-mock-practice/scenario/restore-mysql-persistent-data)