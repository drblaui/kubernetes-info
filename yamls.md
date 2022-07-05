# Kubernetes YAMLs

Basic explanation how to create Config Files for Kubernetes and how they can interact with eachother.

## ConfigMap

Allows creating environment variables or volume files for pods. Can be reused.
See [`config_map_example.yaml`](examples/config_map_example.yaml) for example structure.
Using ConfigMaps in pods can be done in 4 ways:

- Inside a container command and args
- Environment variables for a container, see [`pod_example.yaml`](examples/pod_example.yaml#L12-L17) (Lines 12 - 17)
- mounted as a read-only volume in a container, see [`pod_example.yaml`](examples/pod_example.yaml#L18-L31) (Lines 18 - 31)
- Code inside a Pod that reads the ConfigMap via Kubernetes API

Choosing the right method is basically personal flavor, however ConfigMaps mounted as Volumes are the only way the Pod will realize that the values in that ConfigMap have updated. So if you work with **mutable** ConfigMaps you should think about mounting it as a **Volume**. CofigMaps set as **Environment** will **not autorefresh**  and need a complete **Pod restart** to apply new value.

### Immutable ConfigMaps

ConfigMaps are mutable by default. This means changing a ConfigMap (either by hand or by any kind of code) may cause unwanted side effects. Worst Case you change a ConfigMap and your whole Cluster collapses onto itself. This is where immutable ConfigMaps come into play. Immutable ConfigMaps are impossible to change, they contain Values you intent to never change. Using immutable ConfigMaps doesn't only save you from killing your Cluster, it's actually also better for your performance, because immutable ConfigMaps mounted as Volumes will not be checked periodically for Updates, so your cluster can relax a little bit.

To make your ConfigMap immutable simply add `immutable: true` in the `.yaml` for your ConfigMap.

To make a immutable ConfigMap mutable, you have to delete the ConfigMap and recreate it as mutable. You should also restart your pods after changing back to mutable ConfigMaps.
