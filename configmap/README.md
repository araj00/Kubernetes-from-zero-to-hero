
### **Understanding Dynamic Updates with ConfigMaps and Secrets**

- Environment variables defined via `env.valueFrom.configMapKeyRef` or `secretKeyRef` are **evaluated only once** when the pod starts.
- Updating the underlying ConfigMap or Secret **does not affect** the values already injected as environment variables in a running container.
- ConfigMaps or Secrets mounted as **volumes** (without `subPath`) **do reflect updates dynamically** inside the container.
- Kubernetes handles dynamic updates using **symlinks** to new file versions, but the application must **re-read the files** to detect changes.
- When mounting individual keys using `subPath`, the file is **copied**, not symlinked, so updates to the ConfigMap or Secret **will not propagate**.
- To enable live updates without restarting pods, prefer **volume mounts without `subPath`** and ensure the application supports **hot reloading** or use a **config-reloader**.

Tip: Mounting ConfigMaps as files is perfect for shipping small static assets or config snippets without rebuilding images.

## Labs:-

- [Configmap as files](https://killercoda.com/cka-mock-practice/scenario/config-mount)
- [Configmap with initcontainer](https://killercoda.com/cka-mock-practice/scenario/init-container-one-piece)
- [Configmap and Secret](https://killercoda.com/cka-mock-practice/scenario/config-secret)
- [Mount Config Files and Readiness Probe](https://killercoda.com/cka-mock-practice/scenario/config-and-probes)