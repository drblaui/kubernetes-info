# Kubernetes YAMLs

Basic explanation how to create Config Files for Kubernetes and how they can interact with eachother.

## ConfigMap

Allows creating environment variables or volume files for pods. Can be reused.
See [`config_map_example.yaml`](examples/config_map_example.yaml) for example structure.
Using ConfigMaps in pods can be done in 4 ways:

- Inside a container command and args
- Environment variables for a container (see `pod_example.yaml` (TODO: Link and Line))
- mounted as a read-only volume in a container (see `pod_example.yaml` (TODO: Link and Line))
- Code inside a Pod that reads the ConfigMap via Kubernetes API
