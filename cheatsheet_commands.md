# Kubernetes Command Cheatsheet

| Command | Description |
| ------- | ----------- |
| `minikube start` | Starts the Kubernetes Cluster |
| `kubectl get <nodes | pods | deployment | services>` | General Info about the specified objects  |
| `kubectl get <node | pod | deployment> <name>` | Info about the specified object |
| `kubectl create deployment <name> --image=<image-url>` | Deploys Image to cluster
| `kubectl proxy` | Exposes API on Cluster for manual communication
| `kubectl exec <podname> -- <command>` | Executes terminal command in specified pod (Very helpful: `kubectl exec -ti <podname> -- bash` starts a terminal session) |
| `kubectl expose deployment/<deploymentname> -type="<type>"` | Exposes deployment API to the internet. Available types: <ul><li>ClusterIP: Exposes API to the Cluster</li><li>NodePort: Exposes API to the internet under `NodeIP:NodePort` (NodePort can be set by `--port <port>`</li><li>LoadBalancer: Load Balancing requests to Service</li><li>ExternalName: Maps CNAME Record to node IP</li></ul> All types in this are supersets of their predecessor|
| `kubectl label pods <podname> <label>=<value>` | Adds a label to a pod |
| `kubectl delete <node | pod | deployment | service> <name>` | Deletes the specified object |
| `kubectl scale deployments/<deploymentname> --replicas=<number>` | Scales the deployment to the specified number of replicas |
| `kubectl set image <deployment>=<imageurl>` | Updates the specified deployment (It's reccomended to have at least 2 replicas for this) |
| `kubectl apply (-f <file> | -k <dir>)` | Applies config files (JSON or YAML) <br/> optional: `-l <label>=<value>` to target specific labeled pods | 