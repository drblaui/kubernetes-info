# Kubernetes YAMLs

Basic explanation how to create Config Files for Kubernetes and how they can interact with eachother.

## ConfigMap

Allows creating environment variables or volume files for pods. Can be reused.
See [`config_map.example.yaml`](examples/config_map.example.yaml) for example structure.
Using ConfigMaps in pods can be done in 4 ways:

- Inside a container command and args
- Environment variables for a container, see [`pod.example.yaml`](examples/pod.example.yaml#L12-L17) (Lines 12 - 17)
- mounted as a read-only volume in a container, see [`pod.example.yaml`](examples/pod.example.yaml#L18-L31) (Lines 18 - 31)
- Code inside a Pod that reads the ConfigMap via Kubernetes API

Choosing the right method is basically personal flavor, however ConfigMaps mounted as Volumes are the only way the Pod will realize that the values in that ConfigMap have updated. So if you work with **mutable** ConfigMaps you should think about mounting it as a **Volume**. ConfigMaps set as **Environment** will **not autorefresh**  and need a complete **Pod restart** to apply new value.

### Immutable ConfigMaps

ConfigMaps are mutable by default. This means changing a ConfigMap (either by hand or by any kind of code) may cause unwanted side effects. Worst Case you change a ConfigMap and your whole Cluster collapses onto itself. This is where immutable ConfigMaps come into play. Immutable ConfigMaps are impossible to change, they contain Values you intent to never change. Using immutable ConfigMaps doesn't only save you from killing your Cluster, it's actually also better for your performance, because immutable ConfigMaps mounted as Volumes will not be checked periodically for Updates, so your cluster can relax a little bit.

To make your ConfigMap immutable simply add `immutable: true` in the `.yaml` for your ConfigMap.

To make a immutable ConfigMap mutable, you have to delete the ConfigMap and recreate it as mutable. You should also restart your pods after changing back to mutable ConfigMaps.

## Secrets

You *could* add Secrets as a part of ConfigMaps, in fact since Secrets are **not stored encrypted by default**, you don't really see a difference, however separating Secrets from ConfigMaps is a good idea if you either want to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) them later or just want more structure and use good practice.

Secrets aren't simply just a nice alternative to ConfigMaps for sensitive data, they are configured the same way. You can set Secrets either as files in a volume or as environment variables.

You can create Secrets easily in one of two ways.

- Using the `kubectl` command:
  1. Create a `.txt` file containing your data (e.g. `echo -n 'foo' > ./secret.txt`)
  2. Run `kubectl create secret generic secret-name --from-file=secret.txt` (You can use multiple files here, e.g. if you have a username and password you can combine them in one secret, just make sure to add `--from-file=<file>` for each file)
  2.1
  Secrets have keys. Per default they will be named after the file they are from (e.g. `secret.txt` will be named `secret`). If you want custom keys, replace `--from-file=<file>` with `--from-file=<key>=<file>`
  2.2
  If you don't want to create a file for everything, you can also directly pass key value pairs with `--from-literal=<key>=<value>`. Normally you can just pass strings without quotes, but if that string contains any special characters (`$, \, *, ", !`) you should wrap the value in single quotes.
- Using a file:
  1. Convert strings into base64 (e.g. `echo -n 'admin' | base64`)
  2. Paste output with `key:value` into your secrets file (see [`secret.example.yaml`](examples/secret.example.yaml#L7-L8))
  3. To apply the secret run `kubectl apply -f ./secret.yaml`

To **verify** the secrets run `kubectl get secrets` to get infos about all secrets or use `kubectl describe secrets/<name>` to get information about a specific secret. If you use a `.yaml` file you can also use `kubectl get secret <secretname> -o yaml` to get the full yaml file you used.

To **delete** secrets you can use `kubectl delete secret <secretname>`.

To **decode** encoded secrets you can use `kubectl get secret <secretname> -o jsonpath='{.data}`. This will give you normal JSON Output like `{"username":"YWRtaW4=","password":"MWYyZDFlMmU2N2Rm"}` which you can in turn decode using `echo '<encoded_string>' | base64 --decode`.

To **edit** a secret simply use `kubectl edit secret <secretname>`. This will open a editor in which you can edit all values.

Using Secrets as environment variables or volumes is the same way as explained with ConfigMaps.

> Tip: Secrets can also be marked as immutable!
