# RedHat OpenShift OVN-Kubernetes with F5 BIG-IP Routes

This document demonstrates how to use OVN-Kubernetes with F5 BIG-IP Routes to Ingress traffic without using an Overlay. Using OVN-Kubernetes with F5 BIG-IP Routes removes the complexity of creating VXLAN tunnels or using Calico. This document demonstrates Standalone BIG-IP working with OVN-Kubernetes. Diagram below demonstrates a OpenShift 4.11 Cluster with three Master and three worker nodes. The three applications; **tea,coffee and mocha** are deployed in the **cafe** namespace. As you can see from the diagram below the **cafe** namespace requires an annotation for the BIG-IP self-ip. 

![architecture](https://github.com/mdditt2000/openshift-4-9/blob/main/next-gen-routes/diagram/2022-06-08_11-09-57.png)

Demo on YouTube [video]()

### OpenShift Nodes

Capturing the OpenShift nodes is important as BIG-IP routes will need to be created for each OpenSift node to forward traffic

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



# oc describe node ocp-pm-trw88-worker-d6zsg |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/host-addresses: ["10.192.125.172"]
k8s.ovn.org/node-subnets: {"default":"10.129.2.0/23"}

# oc describe node ocp-pm-trw88-worker-k7lsd |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.174/24"}
k8s.ovn.org/node-subnets: {"default":"10.131.0.0/23"}

# oc describe node ocp-pm-trw88-worker-vdtmb |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/host-addresses: ["10.192.125.177"]
k8s.ovn.org/node-subnets: {"default":"10.128.2.0/23"}

Add static routes via tmsh for all node subnets

tmsh create /net route <node_subnet> gw <node_ip>

tmsh create /net route 10.129.2.0/23 gw 10.192.125.172
tmsh create /net route 10.131.0.0/23 gw 10.192.125.174
tmsh create /net route 10.128.2.0/23 gw 10.192.125.177

route 10.129.2.0/23 gw 10.192.125.172
route 10.131.0.0/23 gw 10.192.125.174
route 10.128.2.0/23 gw 10.192.125.177