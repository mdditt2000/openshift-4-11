## OpenShift Multi-Cluster

This document demonstrates OpenShift Multi-Cluster using F5 BIG-IP. This document focuses on **standalone deployment**. Container Ingress Services (CIS) is only deployed in one of the OpenShift clusters as shown in the diagram

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/mulit-cluster/diagram/2023-06-14_15-14-41.png)

## Why OpenShift Multi-Cluster

My environment has two cluster as shown in the diagram above:

* OpenShift 4.11
* OpenShift 4.13

I would like to distribute traffic between the two cluster. While evaluating OpenShift 4.13 before upgrading/rebuilding OpenShift 4.11. Steps used to configure OpenShift multi-cluster using CIS standalone deployment. 

#### Step 1 Fetch KubeConfigs from ExtendedConfigMaps for ExternalCluster

Create the secrets from the kubeconfig. This allows CIS to access the external OpenShift clusters

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

Create the ExtendedConfigMap

```
# oc create -f global-spec-config.yaml
configmap/global-spec-config created
```

ConfigMap [repo](https://github.com/mdditt2000/openshift-4-11/blob/main/mulit-cluster/openshift-4-11/extendedConfigMap/global-spec-config.yaml)