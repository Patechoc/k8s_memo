# k8s_memo
Notes from following the official tutorial and documentation https://kubernetes.io/docs/concepts/

## Basic commands

For most common commands, check these [notes copied from the tutorial](./commands.md)


## Configuring multiple clusters

For configuring multiple clusters (Google Cloud, Azure Kubernetes Cluster, ...), check this link: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

A few commands that come handy:

* `grep -i kubeconfig ~/.bashrc`  # See where my configuration files are located
* `kubectl config view`  # To see all 'contexts' = all tuples {cluster, user, namespace} available
* `kubectl config --kubeconfig=/path/to/config-demo view`  # to see the configurations within oen config file
* `kubectl config --kubeconfig=config-demo view --minify`  # To see only the configuration information associated with the current context, use the `--minify` flag.