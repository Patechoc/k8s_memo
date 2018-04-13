# k8s_memo

Notes from following the official tutorials and documentation:

* https://kubernetes.io/docs/concepts/
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## Basic commands

For most common commands, check these [notes copied from the tutorial](./commands.md)


## Configuring multiple clusters

For configuring multiple clusters (Google Cloud, Azure Kubernetes Cluster, ...), check this link: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

A few commands that come handy:

* `grep -i kubeconfig ~/.bashrc`  # See where my configuration files are located
* `kubectl config view`  # To see all 'contexts' = all tuples {cluster, user, namespace} available
* `kubectl config --kubeconfig=/path/to/config-demo vuse-context exp-scratch` # To switch to the context {cluster=..., user=..., namespace=...} with name `exp-scratch`.
* `kubectl config --kubeconfig=/path/to/config-demo view`  # to see the configurations within oen config file
* `kubectl config --kubeconfig=/path/to/config-demo view --minify`  # To see only the configuration information associated with the current context, use the `--minify` flag.



## To explore

* [Kompose](http://kompose.io/): A conversion tool to go from Docker Compose to Kubernetes.
* [Helm](https://helm.sh/): the package manager for Kubernetes.