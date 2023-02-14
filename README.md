# Provisiong kubernetes clusters to [VMware vCD](https://www.vmware.com/products/cloud-director.html) using (Cluster Api vCD controler) CAPVCD

This project contains simplified steps on building and managing kubernetes clusters on [VMware vCD](https://www.vmware.com/products/cloud-director.html) platform using [CAPVCD](https://github.com/vmware/cluster-api-provider-cloud-director).

Pre-reqs: 
- [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl)
- kubectl
- [dkp](https://d2iq.com/kubernetes-platform) cli
- Routable IPs to serve as VIPs for control plane and k8s services of type LoadBalancer for ingress.  
- vApp Template already created in vCD with a CAPI enabled image associated with it. 
> Note: 
>- The name of the vApp Template should be postfixed with the kubernetes version that it supports. e.g.: `dkp-2.4.0-ubuntu-k8s-v1.24.6`   
>- Use a base OS image and DKP [konvoy image builder(kib)](https://github.com/mesosphere/konvoy-image-builder) to build this image in vSphere and then create a vApp in the associated vCD using this image.

## Part 1: Setting up Cluster API (CAPI) enabled Management Kubernetes Cluster 

Deploy a CAPI bootstrap Management cluster
> Note: This can be easly built using the [dkp](https://d2iq.com/kubernetes-platform) cli
> ```bash
> dkp create bootstrap
>```
> 

Once the Management cluster is up and running, the next step is to deploy [CAPVCD](https://github.com/vmware/cluster-api-provider-cloud-director) components on top of it to enable provisioning to vCD.

First, clone this repository and move to the direcory it creates

```bash
git clone https://github.com/arbhoj/capvcd.git
cd capvcd
```

Then copy artifacts from the cloned repository to specific locations on the server connected to the CAPI bootstrap Management cluster

```bash
mkdir -p ~/infrastructure-vcd/v1.0.0/
cp -r infrastructure-vcd/v1.0.0 ~/infrastructure-vcd/.

cp infrastructure-vcd/clusterctl.yaml 

cp infrastructure-vcd/clusterctl.yaml ~/.cluster-api/clusterctl.yaml
```

> Note: If `~/.cluster-api/clusterctl.yaml` already exists in the environment for another infrastructure provider, then do not overwrite the file. Instead use the example in this repository to append the `providers` section of the existing file 

Change the `providers.url` path for `vcd` provider in `~/.cluster-api/clusterctl.yaml` to point to the absolute path of the `~/infrastructure-vcd/v1.0.0/infrastructure-components.yaml` file. 

>Note: Absolute path is mandatory. Relative path doesn't work.


Now use the `clusterctl` command to deploy the CAPVCD components to the Management cluster. This will deploy the controller and the required CRDs

> Ensure kubectl is pointing to the CAPI Management cluster 

```bash
clusterctl generate provider -i vcd:v1.0.0 | kubectl apply -f -
```

Ensure the capvcd comes up successfully
```bash
kubectl get pods -n capvcd-system -w
```

Now that the core components are up, deploy the ClusterResourceSets to provide CNI, CSI & CPI/CCM capabilities

```bash
kubectl apply -f common-crs
```

With these simple steps we are now ready to start provisioning Kubernetes clusters to [VMware vCD](https://www.vmware.com/products/cloud-director.html)

## Part 2: Deploying kubernetes workload clusters

To deploy clusters simply initiatlize some environments specifying the details of the cluster using the provided cluster.init script as reference and run clusterctl to generate the artifacts


Declare cluster name
```bash
export CLUSTER_NAME=my-first-vcd-cluster
cp cluster.init ${CLUSTER_NAME}.init
```

Then make chages as required 
```bash
# vi ${CLUSTER_NAME}.init
chmod +x ${CLUSTER_NAME}.init
```
Load the environment variables
```bash
. ./${CLUSTER_NAME}.init
```

Generate the cluster specs
```bash
clusterctl generate cluster -i vcd:v1.0.0 ${CLUSTER_NAME} > ${CLUSTER_NAME}.yaml
```

Review the generated manifest and make changes if requered
```
# vi ${CLUSTER_NAME}.yaml
```bash
Apply the manifests to the Management cluster to kick off a cluster build
```
kubectl apply -f ${CLUSTER_NAME}.yaml

```bash

Watch the progress
```bash
dkp describe cluster -c ${CLUSTER_NAME}
```

Once the cluster has been successfully deployed, get the kubeconfig file of the cluster and view the health of the cluster 

```bash
export WORKLOAD_KUBECONFIG=${CLUSTER_NAME}.conf

kubectl  --kubeconfig=${WORKLOAD_KUBECONFIG} get nodes

kubectl  --kubeconfig=${WORKLOAD_KUBECONFIG} get pods -A

```

## Cleanup

To delete the cluster just provisioned simply use the following DKP cli

```
dkp delete cluster -c ${CLUSTER_NAME} --delete-kubernetes-resources=false
```