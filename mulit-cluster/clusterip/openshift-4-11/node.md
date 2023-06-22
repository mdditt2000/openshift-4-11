### Get Node for OpenShift 4-11

```
[root@ocp-installer cis]# oc get nodes
NAME                        STATUS   ROLES    AGE    VERSION
ocp-pm-trw88-master-0       Ready    master   254d   v1.24.0+3882f8f
ocp-pm-trw88-master-1       Ready    master   254d   v1.24.0+3882f8f
ocp-pm-trw88-master-2       Ready    master   254d   v1.24.0+3882f8f
ocp-pm-trw88-worker-d6zsg   Ready    worker   254d   v1.24.0+3882f8f
ocp-pm-trw88-worker-k7lsd   Ready    worker   254d   v1.24.0+3882f8f
ocp-pm-trw88-worker-vdtmb   Ready    worker   254d   v1.24.0+3882f8f
```

#### Workers
```
# oc describe node ocp-pm-trw88-worker-d6zsg |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.172/24"}
k8s.ovn.org/node-subnets: {"default":"10.129.2.0/23"}
```

```
# oc describe node ocp-pm-trw88-worker-k7lsd |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.174/24"}
k8s.ovn.org/node-subnets: {"default":"10.131.0.0/23"}
```

```
# oc describe node ocp-pm-trw88-worker-vdtmb |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.177/24"}
k8s.ovn.org/node-subnets: {"default":"10.128.2.0/23"}
```
---

### Get Node for OpenShift 4-13

[root@ocp-installer cis]# oc get nodes
NAME                           STATUS   ROLES                  AGE   VERSION
ocp-tpm-7rx67-master-0         Ready    control-plane,master   9d    v1.26.3+b404935
ocp-tpm-7rx67-master-1         Ready    control-plane,master   9d    v1.26.3+b404935
ocp-tpm-7rx67-master-2         Ready    control-plane,master   9d    v1.26.3+b404935
ocp-tpm-7rx67-worker-0-7n466   Ready    worker                 9d    v1.26.3+b404935
ocp-tpm-7rx67-worker-0-ttwdj   Ready    worker                 9d    v1.26.3+b404935
ocp-tpm-7rx67-worker-0-zrwmq   Ready    worker                 9d    v1.26.3+b404935

#### Workers
```
# oc describe node cp-tpm-7rx67-worker-0-7n466 |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.187/24"}
k8s.ovn.org/node-subnets: {"default":["10.131.0.0/23"]} --------------------------------- Overlap with OpenShift 4-11
```

```
# oc describe node ocp-tpm-7rx67-worker-0-ttwdj |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.189/24"}
k8s.ovn.org/node-subnets: {"default":["10.129.2.0/23"]} --------------------------------- Overlap with OpenShift 4-11
```

```
# oc describe node ocp-tpm-7rx67-worker-0-zrwmq |grep "node-subnets\|node-primary-ifaddr"
k8s.ovn.org/node-primary-ifaddr: {"ipv4":"10.192.125.188/24"}
k8s.ovn.org/node-subnets: {"default":["10.128.2.0/23"]} --------------------------------- Overlap with OpenShift 4-11
```