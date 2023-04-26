# F5 BIG-IP Integration with OpenShift using F5 CIS to Configure Network

**Note**: This demo and document is only for gathering feedback and making possible changes for CIS 2.13. This is a pre-release and not for production use

This document expands on [OpenShift OVN-Kubernetes using F5 BIG-IP HA with NO Tunnels](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/ovn-kubernetes-ha#readme) where CIS dynamically configures BIG-IP with network routes of the OpenShift Cluster. This feature simplifies BIG-IP administrators work when OpenShift nodes are adding or removed. 

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/next-gen-routes-2-13/diagram/2023-04-05_10-31-46.png)

Demo on YouTube [video]()

## Using CIS Next Generation OpenShift Route

### Step 1: Deploy CIS

Add the following additional parameters to the CIS deployment

* Create static routes on BIG-IP  --static-routing-mode=true
* Using OVN-Kubernetes --orchestration-cni=ovn-k8s

```
args: [
  # See the k8s-bigip-ctlr documentation for information about
  # all config options
  # https://clouddocs.f5.com/containers/latest/
    "--bigip-username=$(BIGIP_USERNAME)",
    "--bigip-password=$(BIGIP_PASSWORD)",
    "--bigip-url=10.192.125.60",
    "--bigip-partition=OpenShift",
    "--namespace=default",
    "--namespace=cafeone",
    "--namespace=cafetwo",
    "--pool-member-type=cluster",
    "--log-level=DEBUG",
    "--insecure=true",
    "--route-spec-configmap=default/global-cm",
    "--controller-mode=openshift",
    "--static-routing-mode=true",
    "--orchestration-cni=ovn-k8s",
    "--as3-validation=true",
    "--log-as3-response=true",
```

Deploy CIS in OpenShift

```
oc create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
oc create -f bigip-ctlr-clusterrole.yaml
oc create -f f5-bigip-ctlr-deployment.yaml
```

CIS [repo](https://github.com/mdditt2000/openshift-4-11/blob/main/next-gen-routes-2-13/cis/f5-bigip-ctlr-deployment.yaml)

### Step 2: Validate Network Routes Created on BIG-IP

CIS creates the network routes under the Application Tenant on BIG-IP. CIS will also log the creation of the routes in the logs

```
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 f5_cccl.service.manager DEBUG] Getting route tasks...
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] desired set is {'k8s-route-10.192.125.169', 'k8s-route-10.192.125.170', 'k8s-route-10.192.125.174', 'k8s-route-10.192.125.171', 'k8s-route-10.192.125.172', 'k8s-route-10.192.125.177'}
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] existing set is {'k8s-route-10.192.125.169', 'k8s-route-10.192.125.170', 'k8s-route-10.192.125.171', 'k8s-route-10.192.125.174', 'k8s-route-10.192.125.172', 'k8s-route-10.192.125.177'}
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 f5_cccl.service.manager DEBUG] Getting arp tasks...
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] desired set is set()
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] existing set is set()
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 f5_cccl.service.manager DEBUG] Getting tunnel tasks...
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] desired set is set()
2023/04/05 17:11:40 [DEBUG] [2023-04-05 17:11:40,829 root DEBUG] existing set is set()
```

No ARP, or VXLAN tunnels required

![routes](https://github.com/mdditt2000/openshift-4-11/blob/main/next-gen-routes-2-13/diagram/2023-04-05_10-45-48.png)