# Summary related to security context

In Kubernetes, the securityContext allows you to fine-tune privileges and access control at both the Pod and Container levels. However, not all fields are applicable in both scopes, and it’s crucial to understand which attributes can be set where—especially when crafting security policies or debugging unexpected behavior. The table below offers a clear breakdown of field applicability, helping you write minimal and secure pod specs with precision.


| Field                      | Pod Level | Container Level | Notes                                                                 |
|---------------------------|-----------|------------------|-----------------------------------------------------------------------|
| `runAsUser`               | ✅         | ✅                | Container overrides Pod                                               |
| `runAsGroup`              | ✅         | ✅                | Container overrides Pod                                               |
| `runAsNonRoot`            | ✅         | ✅                | Container overrides Pod                                               |
| `fsGroup`                 | ✅         | ❌                | Pod-level only; affects volume ownership                              |
| `allowPrivilegeEscalation`| ❌         | ✅                | Container-level only                                                  |
| `privileged`              | ❌         | ✅                | Container-level only                                                  |
| `readOnlyRootFilesystem`  | ❌         | ✅                | Container-level only                                                  |

✅ = Supported  
❌ = Not supported

## Pod-Level Fields (`spec.securityContext`)

* **`runAsUser`**: Sets the UID for all containers in the Pod. Example: `1000`. No username mapping is required inside the container.

  > Recommended: Use a non-zero UID (e.g., `1000`) to avoid running as root.

* **`runAsGroup`**: Sets the primary GID for container processes. Useful for managing file permissions and shared access.

  > Recommended: Set an appropriate non-root GID to match mounted volumes or app requirements.

* **`fsGroup`**: Assigns a supplemental GID to all processes, granting group ownership to mounted volumes. Files created inside those volumes will belong to this group.

  > Recommended: Set to allow non-root users access to shared mounted volumes.

* **`runAsNonRoot`**: Enforces non-root execution. If omitted and the image defaults to UID 0 (root), the pod fails to start. Acts as a safeguard against unintentional root access.

  > Recommended: Always set to `true` to enforce non-root user execution.

---

## Container-Level Fields (`spec.containers[].securityContext`)

* **`allowPrivilegeEscalation`**: Prevents processes in the container from gaining additional privileges (e.g., via `setuid` binaries).
  In Linux, **privilege escalation** means a process can gain more powerful permissions than it started with — even if it didn’t start as root. This can happen through special mechanisms like **setuid binaries** (binaries that run with the file owner’s permissions).
  Suppose there's a binary owned by `root` with the `setuid` bit set. Even if a **non-root** user runs it, it will execute with **root privileges**.
  Without protection, your container could elevate its privileges even if it starts with a safe UID.

  > Recommended: Set to `false` unless the application explicitly needs privilege escalation.

* **`privileged`**: Grants full host access to the container — including kernel capabilities, device access, and other unrestricted system-level controls. Equivalent to “no isolation.”

  > Recommended: Avoid setting to `true` unless necessary for privileged workloads (e.g., low-level networking or storage operations).

* **`readOnlyRootFilesystem`**: Mounts the container’s root filesystem as read-only. Prevents any write operations to system directories like `/`, `/etc`, `/var`, helping protect the container from tampering or accidental changes.

  > Recommended: Set to `true` for stateless containers or apps that don’t require root filesystem writes.

## What Are Linux Capabilities?

Instead of giving full root access to a container, Linux capabilities allow you to **drop unnecessary privileges** and optionally **add only the ones required**.

This aligns with the **principle of least privilege**, helping reduce the attack surface and prevent containers from performing dangerous operations.

---

### Best Practice with Linux Capabilities

By default, containers — even when not running as root — are granted a minimal set of Linux capabilities such as `CHOWN`, `SETUID`, `NET_BIND_SERVICE`, and `KILL` to enable essential operations like changing file ownership or binding to low-numbered ports.

However, if you explicitly drop **all** capabilities using `capabilities.drop: ["ALL"]`, even these default privileges are removed. As a result, many standard operations will fail — demonstrating that simply running as root (UID 0) doesn't equate to full privilege without the necessary capabilities.

> **Always tailor capabilities to your application's needs.** Start with none and add only what is absolutely required.


---

## Common Linux Capabilities

| Capability     | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `NET_ADMIN`    | Modify network interfaces, routing tables, and firewall rules                                   |
| `SYS_ADMIN`    | Very broad; includes mounting filesystems, changing kernel params, system configuration changes |
| `SYS_TIME`     | Set system clock, modify hardware clock, manage time sync services (e.g., NTP)                  |
| `CHOWN`        | Change file ownership using `chown`                                                             |
| `SETUID`       | Change user ID of a running process                                                             |
| `DAC_OVERRIDE` | Override standard file permission checks (Discretionary Access Control)                         |