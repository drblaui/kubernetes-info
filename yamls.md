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

## Pods

Pods are the smallest component of any kubernetes cluster. They hold at least one container, but can also hold multiple containers if you feel the need to group them. Volumes mounted for the containers are also contained in a pod. Each pod has an own IP address under which you can reach the containers/deployments/services/etc belonging to that pod. Pods to support different container runtimes, however docker is mostly used. You can think of a pod as a **shared memory system** since all containers in a pod share namespaces and filesystem(s).

You can configure Containers in a Pod like you know them from Docker or any other container runtime. For some examples see [pod.example.yaml](examples/pod.example.yaml).

If you want to *create* a Pod from a `.yaml` file simply use the `kubectl apply -f <file>` command. *Ideally* you provide an internet link to a yaml file (thats hosted on GitHub for example), but you can also use a local file, just make sure you aren't in a running pod first.

<!-- TODO: Links-->
While you can create a Pod using a config file, it's rather unusual to do so. Normally Pods are created by a Deployment or a Job.

Pods aren't there forever (unless you run a container that's supposed to never stop). A Pod runs it's lifecycle until it's finished executing.

## Nodes

Nodes are physical or virtual machines that hold pods, the `kubelet` - which is responsible for running containers - and a container runtime (e.g. Docker). Normally a kubelet on the node goes through the trouble of registering said node to the control plane, however you could also manually add nodes, but I wouldn't reccomend it.

## Deployment

Deployments can be seen as a form of state management for pods. A Deployment holds a specific state you want you pod to be in, but a deployment can also change states (i.e Updating, Rerolling, etc) of your pod.

Deployments can create pod **replicas**, which basically just means running multiple "clone" instances of a specific pod, that all receive the same IP address and act the same. Replicas are very useful for load balancing or even updating your application.

To create a deployment simply use `kubectl apply -f <file>`. You can either use an actual file on the machine or provide a link to the location of the file (e.g. a GitHub file).

### Scaling

Scaling is the process of creating replicas by hand to either prepare your app for an update or just simply to have load balancing. You can specify the exact scale you want to by using `kubectl scale deployment/<name> --replicas=<number>` or you can let kubernetes handle the up- and downscaling by using `kubectl autoscale deployment/<name> --min=<number> --max=<number> --cpu-percent=<number>` (but you need to enable [horizontal pod autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) first).

### Updating

The best part about kubernetes is that you can update your software with virtually no downtime (if you have replicas that is). Before updating it is recommended to scale your deployment up to at least 2 replicas. Changing anythin in the [spec.template](./examples/deployment.example.yaml#L15-L24) part will automatically trigger a rollout, however you can also update deployments by hand with `kubectl set image deployment/<name> <app-label-name>=<new-image>`. This will sequentially update pods and replicas while making sure at least one pod with either the new or the old version is available and ready. You can also edit the spec file directly using `kubectl edit deployment/<name>`.

### Rollback

Sometimes updating was a mistake. Either you have a faulty image or just made a typo. You can rollback to a stable version very easily by using `kubectl rollout undo deployment/<name>`. However if you want to go pack several steps, you can utilize the rollout history. Using `kubectl rollout history deployment/<name>` you see all revisions you did to your deployment and can go back to any revision by simply appending `--to-revision=<revision>` to the undo command.
