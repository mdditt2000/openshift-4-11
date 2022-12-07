# Advancing OpenShift Routes using ExternalDNS and WAF

This document demonstrates how F5 Controller Ingress Services (CIS) can advance OpenShift Routes. F5 CIS can expand the OpenShift Route API to use multiple Public IP addresses per Host. Without F5 CIS, OpenShift Route API can only manage one Public IP address. 

In this example we are using multiple hosts **cafeone** and **cafetwo** with three endpoints; **tea,coffee and mocha** as shown in the diagram below. Wide-IPs for Hosts **cafeone.example.com** and **cafetwo.example.com** are created on F5 GTM using ExternalDNS CRDs. All application will be protected using F5 WAF. All the Routes, ExternalDNS and WAF is managed from the OpenShift API.

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/next-gen-routes-2-11/diagram/2022-12-06_15-50-57.png)

Demo on YouTube [video]()

## Using CIS Next Generation OpenShift Route

### Step 1: Deploy CIS

Currently OpenShift Route API using HAproxy which can only support one **Public IP**, **Virtual Server** per BIG-IP. Routes uses HOST Header Load balancing to determine the backend application. CIS can resolve this issue by using a Global ConfigMap to specify global objects like IPs, Policies etc. This is similar to how Gateway API will work. The Global ConfigMap can be applied in different namespace than the Routes

Add the following parameters to the CIS deployment

* Global ConfigMap info is passed to CIS with argument --route-spec-configmap="namespace/configmap-name"
* Controller mode should be set to openshift to enable multiple VIP support(--controller-mode="openshift")
* Using AS3 API to post ExternalDNS configuration --cccl-gtm-agent=false
* Using OVN Kubernetes with static routes. No CNI configured

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
    "--as3-validation=true",
    "--log-as3-response=true",
    "--cccl-gtm-agent=false",
```

Deploy CIS in OpenShift

```
oc create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=<secret>
oc create -f bigip-ctlr-clusterrole.yaml
oc create -f f5-bigip-ctlr-deployment.yaml
```

CIS [repo](https://github.com/mdditt2000/openshift-4-11/tree/main/next-gen-routes-2-11/cis)

### Step 2: Deploy Global ConfigMap

Using Global ConfigMap

* Global ConfigMap provides control to the admin to create and maintain the resource configuration centrally like Public IPs, Policies, Certificates etc
* RBAC can be used to restrict modification of global ConfigMap by users with tenant level access

* namespace: cafeone, vserverAddr: 10.192.125.65
* namespace: cafetwo, vserverAddr: 10.192.125.66

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: global-cm
  namespace: default
  labels:
    f5nr: "true"
data:
  extendedSpec: |
    extendedRouteSpec:
    - namespace: cafeone
      vserverAddr: 10.192.125.65
      vserverName: cafeone
      allowOverride: true
      policyCR: default/policy-cafe
    - namespace: cafetwo
      vserverAddr: 10.192.125.66
      vserverName: cafetwo
      allowOverride: true
      policyCR: default/policy-cafe
```

Deploy global ConfigMap

```
oc create -f global-cm.yaml
```

### Step 3: Creating OpenShift Routes for cafe.example.com

User-case for the OpenShift Routes:

- Edge Termination
- Backend listening on PORT 8080
- Wide-IP configured to GTM
- WAF, logging policies applied to Virtual IP via PolicyCR

Create OpenShift Routes for **cafeone.example.com**

```
oc create -f route-tea-edge.yaml
oc create -f route-coffee-edge.yaml
oc create -f route-mocha-edge.yaml
```
Routes [repo](https://github.com/mdditt2000/openshift-4-11/tree/main/next-gen-routes-2-11/ocp-route/cafeone/route)

Create OpenShift Routes for **cafetwo.example.com**

```
oc create -f route-tea-edge.yaml
oc create -f route-coffee-edge.yaml
oc create -f route-mocha-edge.yaml
```
Routes [repo](https://github.com/mdditt2000/openshift-4-11/tree/main/next-gen-routes-2-11/ocp-route/cafetwo/route)

Validate OpenShift Routes for cafeone

![openshift](https://github.com/mdditt2000/openshift-4-9/blob/main/next-gen-routes/diagram/2022-06-07_15-35-21.png)

Validate OpenShift Virtual IP using the BIG-IP

![big-ip pools](https://github.com/mdditt2000/openshift-4-9/blob/main/next-gen-routes/diagram/2022-06-07_15-37-33.png)

Validate OpenShift Routes policies on the BIG-IP

![traffic](https://github.com/mdditt2000/openshift-4-9/blob/main/next-gen-routes/diagram/2022-06-07_15-38-08.png)

Validate OpenShift Routes policies by connecting to the Public IP

![traffic](https://github.com/mdditt2000/openshift-4-9/blob/main/next-gen-routes/diagram/2022-06-07_15-38-33.png)

