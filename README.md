# Node TTL

Enforces a time to live (TTL) on Kubernetes nodes and evicts nodes which have expired.

## Background

Some Kubernetes clusters may replace nodes at a rapid rate due to frequent version upgrades or large variations in resource scaling. Other Kuberentes clusters however may have nodes that can be running for long periods of time. For the most part this is totally fine there are however benefits in limiting how long nodes can run within the cluster.

* Long running nodes tend to experience issues more frequently than new nodes.
* Restarting nodes limits how long a Pod can run which limits issues and tests frequent restarts.
* Having Pods reschedule can optimize their node placement and decrease the node count.
* New mutations to Pods will not take affect until a Pod is recreated.

Node TTL requires all nodes which should be considered to be labeled with a TTL duration. A node which has expired its TTL will first be cordoned and then drained. When the node is fully drained all Pods, excluding those created by DaemonSet, will be removed from the node. If required a new node will be added to cover the removed compute resources.Having properly configured Pod Disruptions Budgets is very important when using Node TTL as without it a Deployment could be at risk to have zero replicas running. The drain process will respect any configured Pod Disruption Budget.

> Node TTL requires [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) to be present in the cluster.

When the node is fully cordoned and drained no new Pods will be scheduled to that Pod. The Cluster Autoscaler will eventually considered the Node unneeded as it will be underutilized. When this occurs the Cluster Autoscaler will remove the underlying VM from the cloud provider which will result in the removal of the Node from the cluster. It may take some time for the Cluster Autoscaler to consider the Node to be unneeded. This duration can be decreased by configuring `scale-down-unneeded-time`, but it will also have an affect on the scale down of Nodes outside of TTL.

## Installation

Easiest method to install Node TTL is with the [Helm Chart](./charts/node-ttl). Out of the box it requires no configuration, although there are setting that can be tuned.

```shell
helm repo add https://xenitab.github.io/node-ttl/
helm install node-ttl
```

## Usage

When the Node TTL is installed all that is required is to label the nodes correctly. Each node that should be considered by Node TTL should have the label `xkf.xenit.io/node-ttl` where the value is the duration is the maximum life time of the node. The TTL value has to be a valid [duration string](https://pkg.go.dev/time#ParseDuration) with a valid unit. Valid time units are `ns`, `us`, `ms`, `s`, `m`, `h`. 

```yaml
apiVersion: v1
kind: Node
metadata:
  name: kind-worker
  labels:
    xkf.xenit.io/node-ttl: 24h
```

The following node will be considered for eviction after it has existed for more the 24 hours.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
