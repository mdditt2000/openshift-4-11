# Easy F5/NGINX Integrating using OpenShift 4.11

The purpose of this document is to demonstrate how easy it is to integration **F5 BIG-IP and NGINX technologies using OpenShift 4.11**. This guide simplifies the solution by providing examples and step by step guidance using **Operators and OVN-Kubernetes**

F5 BIG-IP and NGINX provides a solutions called **IngressLink** that use both **F5 BIG-IP Container Ingress Services (CIS)** and **NGINX Ingress Controller** deployed in OpenShift 4.11. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of **F5 BIG-IP and NGINX** and **fostering better collaboration across NetOps and DevOps teams**. The diagram below demonstrates the architecture

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-38-38.png)

Demo on YouTube [video]()

On this page you’ll find:

* Links to the GitHub repositories for all the requisite software
* Documentation for the solution(s)

## Configure F5 BIG-IP HA
This document demonstrates **High Availability (HA) F5 BIG-IP's working with OVN-Kubernetes**. Using OVN-Kubernetes with F5 BIG-IP and static routes removes the complexity of creating VXLAN tunnels or using Calico

**Step 1:**

### Step 1: Deploy OpenShift using OVNKubernetes

Deploy OpenShift Cluster with **networktype** as **OVNKubernetes**. Change the default to **OVNKubernetes** in the install-config.yaml before creating the cluster

### Step 2: Verify gateway mode set to shared

```
# oc get nodes
NAME                        STATUS   ROLES    AGE   VERSION
ocp-pm-trw88-master-0       Ready    master   68m   v1.24.0+3882f8f
ocp-pm-trw88-master-1       Ready    master   67m   v1.24.0+3882f8f
ocp-pm-trw88-master-2       Ready    master   67m   v1.24.0+3882f8f
ocp-pm-trw88-worker-d6zsg   Ready    worker   50m   v1.24.0+3882f8f
ocp-pm-trw88-worker-k7lsd   Ready    worker   55m   v1.24.0+3882f8f
ocp-pm-trw88-worker-vdtmb   Ready    worker   55m   v1.24.0+3882f8f
```

```
# oc logs -f ovnkube-node-2bcx7 ovnkube-node -n openshift-ovn-kubernetes|grep "gateway_mode_flags"
+ gateway_mode_flags='--gateway-mode shared --gateway-interface br-ex'
```

### Step 3: Configure BIG-IP Routes

Configure static routes in BIG-IP with node subnets assigned for the three worker nodes in the OpenShift cluster. Get the node subnet assigned and host address

```
# oc describe node ocp-pm-trw88-worker-d6zsg |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/host-addresses: ["10.192.125.172"]
k8s.ovn.org/node-subnets: {"default":"10.129.2.0/23"}

# oc describe node ocp-pm-trw88-worker-k7lsd |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.174/24"}
k8s.ovn.org/node-subnets: {"default":"10.131.0.0/23"}

# oc describe node ocp-pm-trw88-worker-vdtmb |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/host-addresses: ["10.192.125.177"]
k8s.ovn.org/node-subnets: {"default":"10.128.2.0/23"}
```

Add static routes to BIG-IP via tmsh for all node subnets using 

```
tmsh create /net route <node_subnet> gw <node_ip>
```
```
tmsh create /net route 10.129.2.0/23 gw 10.192.125.172
tmsh create /net route 10.131.0.0/23 gw 10.192.125.174
tmsh create /net route 10.128.2.0/23 gw 10.192.125.177
```
View static routes created on BIG-IP

![routes](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ovn-kubernetes-ha/diagram/2022-10-12_13-30-34.png)

**Note:** Manually sync the BIG-IP so the routes are deployed on the standby

### Step 4: Configure egress from OpenShift cluster to BIG-IP

Configure egress from OpenShift cluster to BIG-IP using k8s.ovn.org/routing-external-gws annotation on namespace where the application is deployed as shown in the diagram above. Use the BIG-IP floating self-IP address for the **routing-external-gws: 10.192.125.62**

```
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    k8s.ovn.org/routing-external-gws: 10.192.125.62 ##BIG-IP floating self-interface address rotatable to the OpenShift nodes
  labels:
    kubernetes.io/metadata.name: default
  name: cafe
```
routing-external-gws [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ovn-kubernetes-ha/demo-app/cafe/name-cafe.yaml)

**Setup complete!** Deploy CIS and create OpenShift Routes

### Step 5: Deploy CIS for each BIG-IP

F5 Controller Ingress Services (CIS) called **Next Generation Routes Controller**. Next Generation Routes Controller extended F5 CIS to use multiple Virtual IP addresses. Before F5 CIS could only manage one Virtual IP address per CIS instance.

Add the following parameters to the CIS deployment

* Routegroup specific config for each namespace is provided as part of extendedSpec through ConfigMap.
* ConfigMap info is passed to CIS with argument --route-spec-configmap="namespace/configmap-name"
* Controller mode should be set to openshift to enable multiple VIP support(--controller-mode="openshift")

### BIG-IP 01

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
  "--bigip-username=$(BIGIP_USERNAME)",
  "--bigip-password=$(BIGIP_PASSWORD)",
  "--bigip-url=10.192.125.60",
  "--bigip-partition=OpenShift",
  "--namespace=nginx-ingress",
  "--pool-member-type=cluster",
  "--insecure=true",
  "--custom-resource-mode=true",
  "--as3-validation=true",
  "--log-as3-response=true",
]
```

### BIG-IP 02

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
  "--bigip-username=$(BIGIP_USERNAME)",
  "--bigip-password=$(BIGIP_PASSWORD)",
  "--bigip-url=10.192.125.61",
  "--bigip-partition=OpenShift",
  "--namespace=nginx-ingress",
  "--pool-member-type=cluster",
  "--insecure=true",
  "--custom-resource-mode=true",
  "--as3-validation=true",
  "--log-as3-response=true",
]
```

Deploy CIS in OpenShift

```
oc create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
oc create -f bigip-ctlr-clusterrole.yaml
oc create -f f5-bigip-ctlr-01-deployment.yaml
oc create -f f5-bigip-ctlr-02-deployment.yaml
oc create -f CustomResourceDefinition.yaml
```

CIS [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/ovn-kubernetes-ha/next-gen-route/cis)

Validate both CIS instances are running 

```
# oc get pod -n kube-system
NAME                                            READY   STATUS    RESTARTS   AGE
k8s-bigip-ctlr-01-deployment-7cc8b7cf94-2csz7   1/1     Running   0          16s
k8s-bigip-ctlr-02-deployment-5c8d8c4676-hjwpr   1/1     Running   0          16s
```