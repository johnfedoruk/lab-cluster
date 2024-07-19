# Wiser Labs

A local repository to quickly bring up and tear down infrastructure projects for testing purposes.

> ⚠️ **Warning**
>
> Make sure you are on the correct container clsuter before running the clustercmd command! Failure to do so could take down production systems!

## How to use

The project contains an executable bash script, `clustercmd`, which can be used to manage the infrastructure.

```plaintext
Usage: ./clustercmd [OPTION]

Options:
  --build, -b      Build the cluster
  --destroy, -d    Destroy the cluster
  --rebuild, -r    Rebuild the cluster
  --help, -h       Display this help message
```

## Experiments

### Priority Experiment

This experiment is meant to test the ability of a priority class to wipe out lower rank priority classes.

#### Experiment Setup

Create a cluster with the following specs:

- Location type: Regional
- Zone: europe-west1
- Nodes: 3 (1 per zone)
- Machine type: e2-standard-2
- Image type: Container-Optimized OS with containerd
- Version: 1.29.6-gke.1038001
- Node auto-provisioning: Disabled

### Test 1

Control test. There are enough resources for all pods to be scheduled.

Checkout branch [test/priority-1](https://github.com/wearewiser/lab-cluster/tree/test/priority-1) and rebuild the cluster.

```
git checkout test/priority-1
./clustercmd --rebuild
```

View the pods across all namespaces.

```
kubectl get pods --all-namespaces
```

All pods should be running. Sufficient resources are available for them.

### Test 2

Increasing resources for **test** pods.

Checkout branch [test/priority-2](https://github.com/wearewiser/lab-cluster/tree/test/priority-2) and simply build cluster to apply changes to resource requests.

```
git checkout test/priority-2
./clustercmd --build
```

View the pods across all namespaces.

```
kubectl get pods --all-namespaces
```

Some pods in the **devl** namespace will be rebuilt to make room for the **test** pods, and some pods in the **devl** namespace will be stuck in *pending* as there are insufficient resources for all pods to be running with the given constraints.

> **Note**
>
> If you notice that some of the **test** pods are pending, then that indicates that distribution of the **prod** pods are preventing the **test** pods from being scheduled across the clusters. To address this, build the cluster so that the **test** pods are built before the higher priority **prod** pods.
>
> ```
> ./clustercmd --rebuild
> ```

### Test 3

Increasing resources for **prod** pods.

Checkout branch [test/priority-3](https://github.com/wearewiser/lab-cluster/tree/test/priority-3) and simply build cluster to apply changes to resource requests.

```
git checkout test/priority-3
./clustercmd --build
```

View the pods across all namespaces.

```
kubectl get pods --all-namespaces
```

Some pods in the **test** namespace will be rebuilt to make room for the **prod** pods, while pods in the **test** namespace will be stuck in *pending* as there are insufficient resources for all pods to be running with the given constraints. Some pods in the **devl** namespace should now be pending, but some of these pods will still be running given their smaller resource constraints, taking up remaining space across the nodes.

### Test 4

Increasing resources for **devl** pods.

Checkout branch [test/priority-4](https://github.com/wearewiser/lab-cluster/tree/test/priority-4) and rebuild cluster to apply changes to resource requests. A rebuild is necessary because the applied changes to resources will not evict the running pods as they will be all unschedulable.

```
git checkout test/priority-4
./clustercmd --rebuild
```

View the pods across all namespaces.

```
kubectl get pods --all-namespaces
```

Some pods in the **test** namespace will be stuck in *pending* as there are insufficient resources for all pods to be running with the given constraints, and all pods in the **devl** namespace should now be pending given there is not enough resources for them to be running on the clsuter.