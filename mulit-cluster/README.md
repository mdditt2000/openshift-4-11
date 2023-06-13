## OpenShift Multi-Cluster

This document demonstrates OpenShift Multi-Cluster using F5 BIG-IP. This document focuses on **standalone deployment**. Container Ingress Services (CIS) is only deployed in one of the OpenShift clusters as shown in the diagram

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/mulit-cluster/diagram/2023-06-13_13-20-02.png)

## Why OpenShift Multi-Cluster

My environment has two cluster OpenShift 4.11 and newly installed OpenShift 4.13. I would like to distribute traffic between the two cluster before taking the OpenShift 4.11 offline to upgrade or rebuild without traffic disruption. Since OpenShift 4.13 is the latest version, I would continue to run CIS on OpenShift 4.11 until satisfied with the newer OpensShift version. Moving CIS to OpenShift 4.13 is relatively simple. Another option is run Active/active CIS which is covered in the next document

#### Step 1 Fetch KubeConfigs from ConfigMaps for ExternalCluster

Follow the documentation for [creating cluster secrets](https://github.com/F5Networks/k8s-bigip-ctlr/tree/multiCluster/docs/config_examples/multicluster/rbac)



```
This will work, pls make sure you can retrieve resource using "oc --kubeconfig=/openshift/ipi/ipi/ipi/auth/kubeconfig get nodes "
```

```
oc create secret generic cluster-1 --from-file=kubeconfig=/root/kubeconfig
where /roo/kubeconfig is /openshift/ipi/ipi/ipi/auth/kubeconfig
```

https://github.com/F5Networks/k8s-bigip-ctlr/blob/multiCluster/docs/config_examples/multicluster/rbac/kube-config-secret-example.yaml


```
# Run the following command to create the secret using the cluster's kube-config file
# kubectl create secret generic <secret-name> --from-file=kubeconfig=<kube-config yaml file name>
# kubectl create secret generic kubeconfig1 --from-file=kubeconfig=kube-config1.yaml
```

[Latest Documentation of Multi-Cluster](https://github.com/F5Networks/k8s-bigip-ctlr/tree/multiCluster/docs/config_examples/multicluster)