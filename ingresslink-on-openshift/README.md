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

Now that the operator is installed you can create an **two instance of CIS**. This will also deploy CIS in OpenShift

![diagram](https://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-07-32.png)

Note that currently some fields may not be represented in form so its best to use the "YAML View" for full control of object creation. Select the "YAML View"

![diagram](hhttps://github.com/mdditt2000/openshift-4-11/blob/main/ingresslink-on-openshift/diagram/2022-10-24_14-24-00.png)

### Step 5

Enter requirement objects in the YAML View. Please add the recommended setting below:

* Remove **agent as3** as this is default
* Change repo image to **f5networks/cntr-ingress-svcs**. By default OpenShift will pull the image from Docker. 
* Change the user to **registry.connect.redhat.com** so OpenShift will be pull the published image from the RedHat Ecosystem Catalog [repo](https://catalog.redhat.com/software/containers/f5networks/cntr-ingress-svcs/5ec7ad05ecb5246c0903f4cf)


```
apiVersion: cis.f5.com/v1
kind: F5BigIpCtlr
metadata:
  name: f5-server
  namespace: openshift-operators
spec:
  args:
    log_as3_response: true
    manage_routes: true
    log_level: DEBUG
    route_vserver_addr: 10.192.75.109
    bigip_partition: OpenShift
    openshift_sdn_name: /Common/openshift_vxlan
    bigip_url: 10.192.75.60
    insecure: true
    pool-member-type: cluster
  bigip_login_secret: bigip-login
  image:
    pullPolicy: Always
    repo: f5networks/cntr-ingress-svcs
    user: registry.connect.redhat.com
  namespace: kube-system
  rbac:
    create: true
  resources: {}
  serviceAccount:
    create: true
  version: latest
```

Select the Create tab

![diagram](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/operator/diagrams/2021-06-14_14-38-24.png)

### Step 6

Validate CIS deployment. Select Workloads/Deployments 

![diagram](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/operator/diagrams/2021-06-14_14-42-54.png)

Select the **f5-bigip-ctlr-operator** to see more details on the CIS deployment. Also validate the CIS deployment image

![diagram](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/operator/diagrams/2021-06-14_14-45-08.png)

     
### Create CIS CRD schema

    kubectl create -f customresourcedefinition.yaml

cis-crd-schema [repo](https://github.com/F5Networks/k8s-bigip-ctlr/blob/master/docs/config_examples/customResourceDefinitions/customresourcedefinitions.yml)

### Update the bigip address, partition and other details(image, imagePullSecrets, etc) in CIS deployment manifest

* Add the following statements to the CIS deployment for CRD support for namespace nginx-ingress

    - "--custom-resource-mode=true"
    - "--namespace=nginx-ingress"

* To deploy the CIS controller in cluster mode update CIS deployment arguments as follows for kubernetes. Additionally, if you are deploying the CIS in Cluster Mode you need to have following prerequisites. For more information, see [Deployment Options](https://clouddocs.f5.com/containers/latest/userguide/config-options.html#config-options)
    
* You must have a fully active/licensed BIG-IP. SDN must be licensed. For more information, see [BIG-IP VE license support for SDN services](https://support.f5.com/csp/article/K26501111).
* VXLAN tunnel should be configured from Kubernetes Cluster to BIG-IP. For more information see, [Creating VXLAN Tunnels](https://clouddocs.f5.com/containers/latest/userguide/cis-helm.html#creating-vxlan-tunnels)

    - "--pool-member-type=cluster"
    - "--flannel-name=fl-vxlan"

* Add the **IPAM parameter** in the CIS deployment to provide **LoadBalancer Service type** support

    - "--ipam=true"

```
kubectl create -f f5-cis-deployment.yaml
```

cis-deployment [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/cis/cis-deployment/f5-cis-deployment.yaml)

### Configure BIG-IP as a node in the Kubernetes cluster. This is required for OVN Kubernetes using ClusterIP

    kubectl create -f f5-bigip-node.yaml

bigip-node [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/cis/cis-deployment/f5-bigip-node.yaml)

**Step 2**

### F5 IPAM Deploy Configuration Options

The **LoadBalancer Service type** provisions an external IP address. CIS uses IPAM integration to provisions this external IP address on BIG-IP. F5 IPAM controller is required for this solution. Add the following configuration parameter to the IPAM manifest:

* --orchestration=kubernetes - The orchestration parameter holds the orchestration environment i.e. Kubernetes
* --ip-range='{"Test":"10.192.75.113-10.192.75.116"}' - ipamlabel that matches the service. Ranch of public IPs that CIS will configure BIG-IP
* --log-level=debug - recommend info after testing

```
args:
  - --orchestration=kubernetes
  - --ip-range='{"Test":"10.192.75.113-10.192.75.116"}'
  - --log-level=DEBUG
```

Modify the persistent volume manifest file that meets your kubernetes deployment 

```
- ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /tmp/cis_ipam
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-1-19-node1.example.com
```

### Deploy RBAC, Schema, F5 IPAM Controller and Persistent Volumes deployment files

```
kubectl create -f f5-ipam-ctlr-rbac.yaml
kubectl create -f f5-ipam-schema.yaml
kubectl create -f f5-ipam-persitentvolume.yaml
kubectl create -f f5-ipam-deployment.yaml
```
ipam-deployment [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/simplifying-ingress/ipam-deployment)

**Step 3**

### Nginx-Controller Installation

This diagram demonstrates the how CIS and IPAM work together for the **LoadBalancer Service type**

![CRD](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/diagram/2021-10-18_14-30-40.png)

Create NGINX IC custom resource definitions for VirtualServer and VirtualServerRoute, TransportServer and Policy resources:

    kubectl apply -f k8s.nginx.org_virtualservers.yaml
    kubectl apply -f k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f k8s.nginx.org_transportservers.yaml
    kubectl apply -f k8s.nginx.org_policies.yaml

crd-schema [repo](https://github.com/nginxinc/kubernetes-ingress/tree/v1.10.0/deployments/common/crds)

Create a namespace and a service account for the Ingress controller:
   
    kubectl apply -f nginx-config/ns-and-sa.yaml
   
Create a cluster role and cluster role binding for the service account:
   
    kubectl apply -f nginx-config/rbac.yaml
   
Create a secret with a TLS certificate and a key for the default server in NGINX:

    kubectl apply -f nginx-config/default-server-secret.yaml
    
Create a config map for customizing NGINX configuration:

    kubectl apply -f nginx-config/nginx-config.yaml
    
Create an IngressClass resource (for Kubernetes >= 1.18):  
    
    kubectl apply -f nginx-config/ingress-class.yaml

Add the annotation to the nginx-service for **LoadBalancer Service type** to obtain the public IP address from IPAM.

```
  annotations:
    cis.f5.com/ipamLabel: Test
```

    kubectl apply -f nginx-config/nginx-ingress.yaml
  
Create a service for the Ingress Controller pods for ports 80 and 443 as follows:

    kubectl apply -f nginx-config/nginx-service.yaml

nginx-config [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/simplifying-ingress/nginx-config)

Validate that the EXTERNAL-IP is populated 

```
❯ kubectl get service -n nginx-ingress
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
nginx-ingress-service   LoadBalancer   10.97.193.45   10.192.75.113   80:30004/TCP,443:31530/TCP   2d22h
```

**Step 3**

### Deploy the Cafe Application

Create the coffee and the tea deployments and services:

    kubectl create -f cafe.yaml

### Configure Load Balancing for the Cafe Application

Create a secret with an SSL certificate and a key:

    kubectl create -f cafe-secret.yaml

Create an Ingress resource:

    kubectl create -f cafe-ingress.yaml

ingress-example [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/simplifying-ingress/ingress-example)

**Step 4**

### Test the Application

Validate the external IP address on BIG-IP

![CRD](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/diagram/2021-10-18_14-50-41.png)

Connext to Cafe App for Coffee and Tea

* cafe.example.com/coffee

![CRD](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/diagram/2021-10-18_14-53-03.png)

* cafe.example.com/tea

![CRD](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/simplifying-ingress/diagram/2021-10-18_14-53-34.png)