# Easy F5/NGINX Integrating using OpenShift 4.11

The purpose of this document is to demonstrate how easy it is to integration **F5 BIG-IP and NGINX technologies using OpenShift 4.11**. This guide simplifies the solution by providing examples and step by step guidance using **Operators and OVN-Kubernetes**

F5 BIG-IP and NGINX provides a solutions called **IngressLink** that use both **F5 BIG-IP Container Ingress Services (CIS)** and **NGINX Ingress Controller** deployed in OpenShift 4.11 using the Operators from OpenShift OperatorHub. It’s an elegant control plane solution that offers a unified method of working with both technologies from a single interface—offering the best of **F5 BIG-IP and NGINX** and **fostering better collaboration across NetOps and DevOps teams**. The diagram below demonstrates the architecture

![architecture](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-38-38.png)

Demo on YouTube [video]()

On this page you’ll find:

* Links to the GitHub repositories for all the requisite software
* Documentation for the solution(s)

## Configure F5 BIG-IP HA
This document demonstrates **High Availability (HA) F5 BIG-IP's working with OVN-Kubernetes**. Using OVN-Kubernetes with F5 BIG-IP and static routes removes the complexity of creating VXLAN tunnels or using Calico

**Step 1:**

### Install the two CIS Controllers using the Operators

### Prerequisites

Create BIG-IP login credentials for use with Operator Helm charts

    oc create secret generic bigip-login  -n kube-system --from-literal=username=admin  --from-literal=password=<secret>

### Step 1

Locate the F5 Container Ingress Services Operator in OpenShift OperatorHub as shown in the diagram below

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-14-36.png)

**Note:** Each F5 BIG-IP requires a unique instance of CIS

### Step 2

Select the Install tab as shown in the diagram

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-50-31.png)

### Step 3

Install the Operator using the defaults

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-53-03.png)

Operator will take a few minutes to install

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-55-06.png)

Once installed select the View Operator tab

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_13-56-49.png)

### Step 4

Create **two instance of CIS**. This will also deploy CIS in OpenShift

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-07-32.png)

Note that currently some fields may not be represented in form so its best to use the "YAML View" for full control of object creation. Select the "YAML View"

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-24-00.png)

### Step 5

Enter requirement objects in the YAML View. Please add the recommended setting below:

* Remove **agent as3** as this is default
* Change repo image to **f5networks/cntr-ingress-svcs**. By default OpenShift will pull the image from Docker. 
* Change the user to **registry.connect.redhat.com** to published image from the RedHat Ecosystem Catalog [repo](https://catalog.redhat.com/software/containers/f5networks/cntr-ingress-svcs/5ec7ad05ecb5246c0903f4cf)
* Change **custom-resource-mode: true** for **IngressLink CRD**

### f5-server-01

```
apiVersion: cis.f5.com/v1
kind: F5BigIpCtlr
metadata:
  name: f5-server-01
  namespace: openshift-operators
spec:
  ingressClass:
    create: false
    ingressClassName: f5
    defaultController: false
  resources: {}
  rbac:
    create: true
  version: latest
  serviceAccount:
    create: true
  image:
    pullPolicy: Always
    repo: f5networks/cntr-ingress-svcs
    user: registry.connect.redhat.com
  namespace: kube-system
  args:
    log-as3-response: true
    custom-resource-mode: true
    log-level: DEBUG
    bigip-partition: OpenShift
    bigip-url: 10.192.75.60
    insecure: true
    pool-member-type: cluster
    namespace: nginx-ingress
  bigip_login_secret: bigip-login
```

Select the Create tab

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-36-33.png)

Create the second instance for **f5-server-02**

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-37-23.png)

### f5-server-02

```
apiVersion: cis.f5.com/v1
kind: F5BigIpCtlr
metadata:
  name: f5-server-02
  namespace: openshift-operators
spec:
  ingressClass:
    create: false
    ingressClassName: f5
    defaultController: false
  resources: {}
  rbac:
    create: true
  version: latest
  serviceAccount:
    create: true
  image:
    pullPolicy: Always
    repo: f5networks/cntr-ingress-svcs
    user: registry.connect.redhat.com
  namespace: kube-system
  args:
    log-as3-response: true
    custom-resource-mode: true
    log-level: DEBUG
    bigip-partition: OpenShift
    bigip-url: 10.192.75.61
    insecure: true
    pool-member-type: cluster
    namespace: nginx-ingress
  bigip_login_secret: bigip-login
```

Verify that **f5-server-01** and **f5-server-02** are created

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-38-43.png)

### Step 6

Validate CIS deployment. Select Workloads/Deployments 

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-38-43.png)

Select the **f5-bigip-ctlr-operator** to see more details on the CIS deployment. Also validate the CIS deployment image

![diagram](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/operator/diagrams/2021-06-14_14-45-08.png)

Failed to create two CIS instances 