## OpenShift Multi-Cluster

This document demonstrates OpenShift Multi-Cluster using F5 BIG-IP. This document focuses on **standalone deployment**. Container Ingress Services (CIS) is only deployed in one of the OpenShift clusters as shown in the diagram



```
This will work, pls make sure you can retrieve resource using "oc --kubeconfig=/openshift/ipi/ipi/ipi/auth/kubeconfig get nodes "
```

```
oc create secret generic cluster-1 --from-file=kubeconfig=/root/kubeconfig
where /roo/kubeconfig is /openshift/ipi/ipi/ipi/auth/kubeconfig
```

https://github.com/F5Networks/k8s-bigip-ctlr/blob/multiCluster/docs/config_examples/multicluster/rbac/kube-config-secret-example.yaml

```
Here is the latest MultiCluster Image : docker.io/nandakishoref5/k8s-bigip-ctlr:MCLatest , Will share you the final codefreeze image once all fixes are done. 
```

```
# Run the following command to create the secret using the cluster's kube-config file
# kubectl create secret generic <secret-name> --from-file=kubeconfig=<kube-config yaml file name>
# kubectl create secret generic kubeconfig1 --from-file=kubeconfig=kube-config1.yaml
```

[Latest Documentation of Multi-Cluster](https://github.com/F5Networks/k8s-bigip-ctlr/tree/multiCluster/docs/config_examples/multicluster)