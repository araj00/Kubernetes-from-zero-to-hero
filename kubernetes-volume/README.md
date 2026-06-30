# Summary related to DownwardAPI:-

- The **Downward API** allows containers to consume information about themselves without calling the Kubernetes API
- **resourceFieldRef** exposes resource requests and limits from containers as environment variables
- The **divisor** field controls the unit of measurement (1m for millicores, 1Mi for mebibytes, 1Ki for kibibytes)
- **containerName** specifies which container's resources to reference (required when referencing another container)
- Environment variables can be consumed by any process inside the container

## DownwardAPI use cases

- Resource monitoring: Sidecars that monitor and report resource usage
- Auto-configuration: Apps that adjust behavior based on allocated resources
- Logging context: Include Pod metadata in application logs
- Service discovery: Use Pod IP and namespace for registration
- License compliance: Applications that validate resource allocations
- Autoscaling triggers: Custom metrics based on actual vs. requested resources

# Labs:-

1. [DownwardAPI lab](https://killercoda.com/cka-mock-practice/scenario/downward-api-env-resource) - Configure Downward API environment variables to expose container resource specifications to a sidecar monitoring container.
