## OpenShift Multi-Cluster

This document demonstrates OpenShift Multi-Cluster using F5 BIG-IP. This document focuses on **standalone deployment** using **NodePort**. Container Ingress Services (CIS) is only deployed in OpenShift 4.11 cluster as shown in the diagram

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/mulit-cluster/diagram/2023-06-14_15-14-41.png)

## Why OpenShift Multi-Cluster

My environment has two cluster as shown in the diagram above:

* OpenShift 4.11
* OpenShift 4.13
* Similar Pods deployed in both cluster

I would like to distribute traffic between the two cluster. While evaluating OpenShift 4.13 before upgrading/rebuilding OpenShift 4.11. Steps used to configure OpenShift multi-cluster using CIS **standalone deployment**. CIS configured in HA deployment will come next. 

#### Step 1 Fetch KubeConfigs from OpenShift-4-13 Cluster

Copy the KubeConfig from **OpenShift-4-13** to **OpenShift-4-11** so that CIS can monitor the services in remote clusters. CIS uses KubeClient

**OpenShift-4-11 Cluster**

View the copy of KubeConfig for both clusters

```
[root@ocp-installer auth]# ls
kubeadmin-password  kubeconfig  kubeconfig-openshift-4-13
[root@ocp-installer auth]#
```

Create secrets using the following command 

```
oc create secret generic openshift-4-13 --from-file=kubeconfig=/openshift/ipi/auth/kubeconfig-openshift-4-13
secret/openshift-4-13 created
```

```
# oc get secret
NAME                       TYPE                                  DATA   AGE
openshift-4-11             Opaque                                1      43h
openshift-4-13             Opaque                                1      35h
[root@ocp-installer route]#
```

**Note:** Since CIS is only deployed in OpenShift-4-11, no need to share KubeConfig  with OpenShift-4-13

#### Step 2 Deploy CIS and RBAC

Deploy CIS and RBAC for OpenShift 4-11 and OpenShift 4-13

**OpenShift-4-11 Cluster**

```
# oc create -f f5-bigip-ctlr-deployment.yaml
deployment.apps/k8s-bigip-ctlr-deployment created
```

```
oc create -f bigip-ctlr-clusterrole.yaml
```
**OpenShift-4-13 Cluster**

```
# oc create -f external-cluster-rabc.yaml
```

#### Step 3 Deploy Cafe application in both Clusters



#### Step 3 Deploy OpenShift Route