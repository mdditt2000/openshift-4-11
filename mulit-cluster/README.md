## OpenShift Multi-Cluster

This document demonstrates OpenShift Multi-Cluster using F5 BIG-IP. This document focuses on **standalone deployment**. Container Ingress Services (CIS) is only deployed in one of the OpenShift clusters as shown in the diagram

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/mulit-cluster/diagram/2023-06-14_15-14-41.png)

## Why OpenShift Multi-Cluster

My environment has two cluster as shown in the diagram above:

* OpenShift 4.11
* OpenShift 4.13

I would like to distribute traffic between the two cluster. While evaluating OpenShift 4.13 before upgrading/rebuilding OpenShift 4.11. Steps used to configure OpenShift multi-cluster using CIS standalone deployment. 

#### Step 1 Fetch KubeConfigs from ConfigMaps for ExternalCluster

Follow the documentation for [creating cluster secrets](https://github.com/F5Networks/k8s-bigip-ctlr/tree/multiCluster/docs/config_examples/multicluster/rbac)

**OpenShift-4-11 Cluster**

```
# oc create secret generic openshift-4-11 --from-file=kubeconfig=/openshift/ipi/auth/kubeconfig
secret/openshift-4-11 created

```

**OpenShift-4-13 Cluster**

```
# oc create secret generic openshift-4-13 --from-file=/openshift/ipi/ipi/ipi/auth/kubeconfig
secret/openshift-4-13 created

```

https://github.com/F5Networks/k8s-bigip-ctlr/blob/multiCluster/docs/config_examples/multicluster/rbac/kube-config-secret-example.yaml


```
# Run the following command to create the secret using the cluster's kube-config file
# kubectl create secret generic <secret-name> --from-file=kubeconfig=<kube-config yaml file name>
# kubectl create secret generic kubeconfig1 --from-file=kubeconfig=kube-config1.yaml
```

[Latest Documentation of Multi-Cluster](https://github.com/F5Networks/k8s-bigip-ctlr/tree/multiCluster/docs/config_examples/multicluster)