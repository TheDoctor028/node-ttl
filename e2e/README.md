# E2E

The E2E tests are meant to validate how Node TTL interacts with Cluster Autoscaler. This is required because Node TTL relies heavily on Cluster Autoscaler to do the heavy lifting. Node TTL merely drains the node and makes it unschedulable while Cluster Autoscaler together with the cloud controller removes the node and backing VM.

Everything is run locally to simplify the e2e tests. Cluster Autoscaler is configured to use the [Kubemark provider](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/proposals/kubemark_integration.md) to scale the nodes. [Kubemark](https://github.com/kubernetes/kubernetes/blob/master/test/kubemark/pre-existing/README.md) is a mocked cloud provider which allows creating hollow nodes in a Kubernetes cluster which behave similar to real nodes but do not actually run the workloads. This is great because it saves on resources when the output of the workload is actually not needed. For each Kubemark node a Kubemark Pod has to be created in the cluster, this Pod will run mocked components and register the Node with the cluster.

Cluster Autoscaler works a bit different with Kubemark compared to how it was initially meant to be used. Cluster Autoscaler will count a single ReplicationController resource as a node instead of having a ReplicationController with mulitple replicas. This is so that it can control which Node will be removed as it has to remove a specific Pod.

> The reason the Kubemark Pod is created with a ReplicationController and not a Deployment has to do with who the Kubemark controller is implemented. It was most likely implemented early on in Kubernetes development and has yet to be updated. Checking the [implementation](https://github.com/kubernetes/kubernetes/blob/0b2e54123f233d87c3a132c94d62d0e551bf60ba/pkg/kubemark/controller.go#L151-L158) it becomes obvious that the Kubemark Pod has to be created with a ReplicationController.

To run Cluster Autoscaler with the Kubemark provider it has to be built locally with it enabled. This is because it has been [removed from the default build](https://github.com/kubernetes/autoscaler/pull/1323) of Cluster Autoscaler. Another interesting caveat that is good to know is the method Cluster Autoscaler and Kubemark interact. In a normal Kubernetes cluster the cloud controller that is baked into Kubernetes or runs side by side will be responsible to remove the Node resources from the cluster when the backing VM has been removed. When running Kubemark the Kubemark controller is not actually not started by Kubernetes. It is instead run by Cluster Autoscaler as can be seen in the [Kubemark provider](https://github.com/kubernetes/autoscaler/blob/1a459646fc795825e4df8285e61151bcf9ab22dc/cluster-autoscaler/cloudprovider/kubemark/kubemark_linux.go#L363). This also means that the Cluster Autoscaler becomes responsible to delete the Nodes when the backing Kubemark Pods have been removed.