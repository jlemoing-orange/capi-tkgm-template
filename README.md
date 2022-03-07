# ClusterAPI template for TKG on vCloud Director

This template use ytt templating tool to simplify tkg cluster creation on VCD

## 1. Retrieve ClusterAPI Template files

These files will be use to create all the objects required to create a cluster

```
git clone https://github.com/jlemoing-orange/capi-tkgm-template

cp ./capi-tkgm-template/values-tmpl.yml values.yml
```

## 2. Fill the values.yml file

The values.yml file is used to fill automatically all the minimal required information inside all the ClusterAPI yaml files using [ytt](https://carvel.dev/ytt/) templating tool.


|Parameter|Value|example|
|:--------|:----|:------|
|name:| Cluster name to create | mycluster1
|namespace| Namespace where the cluster objects should be created in management cluster | default
|sshKey| sshKey used to connect to nodes, should be 2048bits & in OpenSSH format !! DO NOT FORGET TO ENCAPSULATE YOUR TEXT BETWEEN TWO "" ||
|template|VM template name to use (can be found in the vCloud Director catalog)| ubuntu-2004-kube-v1.21.2+vmware.1-tkg.1-7832907791984498322|
|version|Version of Kubernetes to install| v1.21.2+vmware.1|
|url|VCD endpoint with the format https://VCD_HOST. No trailing '/'|https://console1.cloudavenue.orange-business.com|
|org|organization name associated with the user logged in to deploy clusters|ORG1CUSTOMER1
|ovdc|VDC name where the cluster will be deployed|MYPRODVDC1
|ovdcNetwork|OVDC network to use for cluster deployment|PRODNETWORK01
|refreshToken|vCloud Director token to use|
|controlplane : count |Number of master nodes (must be an odd number)| 1
|controlplane : computePolicy|Compute policy to use for control plane nodes, let blank to use the default one| 
|controlplane : dnsVersion|Version of the core-dns component to install, should be link to the kubernetes version install https://github.com/vmware/cluster-api-provider-cloud-director/blob/0.5.0/docs/WORKLOAD_CLUSTER.md#script-to-get-kubernetes-etcd-coredns-versions-from-tkg-ova |  v1.8.0_vmware.5
|controlplane : etcdVersion|Version of the etcd component to install, should be link to the kubernetes version install https://github.com/vmware/cluster-api-provider-cloud-director/blob/0.5.0/docs/WORKLOAD_CLUSTER.md#script-to-get-kubernetes-etcd-coredns-versions-from-tkg-ova | 1 v3.4.13_vmware.15
|workers : count |Number of worker nodes| 3
|workers : computePolicy|Compute policy to use for worker nodes, let blank to use the default one| 

## 3. Apply the configuration
```
ytt  -f ./capi-tkgm-template -f values.yml| kubectl apply -f -
```

## 4. Check the deployment
1. Verify that the Cluster object is in phase provisioned
```
# kubectl get cluster

NAME          PHASE
clmngmono01   Provisioned
```

2. Wait that all masters are READY
```
# kubectl get kubeadmcontrolplane

NAME                  INITIALIZED   API SERVER AVAILABLE   VERSION            REPLICAS   READY   UPDATED   UNAVAILABLE
clmngmono01-masters                                        v1.21.2+vmware.1   1                  1         1
```


3. Wait that all worker nodes are READY and the Phase is Running
```
# kubectl get machinedeployment

NAME                   PHASE       REPLICAS   READY   UPDATED   UNAVAILABLE
clmngmono01-workers0   Running   3          3       3          
```
## 5. Retrieve the kubeconfig file for the new cluster

```
clusterctl get kubeconfig CLUSTERNAME > ~/.kube/kube-CLUSTERNAME
```

example : 
```
clusterctl get kubeconfig clmngmono01 > ~/.kube/kube-clmngmono01
```

Check if you are correctly connected to the good cluster with : 
```
kubectl get nodes --kubeconfig  ~/.kube/kube-clmngmono01
```