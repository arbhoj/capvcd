# Provisiong kubernetes clusters using (Cluster Api vCD controler) CAPVCD

This project contains simplified steps on building and managing kubernetes clusters on VMware vCD platform using [CAPVCD](https://github.com/vmware/cluster-api-provider-cloud-director).

Pre-reqs: 
- [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl)
- kubectl 

## Part 1: Setting up Cluster API (CAPI) enabled Management Kubernetes Cluster 

Deploy a CAPI bootstrap Management cluster
> Note: This can be easly built using the [dkp](https://d2iq.com/kubernetes-platform) cli
> ```
> dkp create bootstrap
>```
> 

Once the Management cluster is up and running, the next step is to deploy [CAPVCD](https://github.com/vmware/cluster-api-provider-cloud-director) components on top of it to enable provisioning to vCD.

First, clone this repository and move to the direcory it creates

```
git clone https://github.com/arbhoj/capvcd.git
cd capvcd
```

Then copy artifacts from the cloned repository to specific locations on the server connected to the CAPI bootstrap Management cluster

```
mkdir -p ~/infrastructure-vcd/v1.0.0/
cp -r infrastructure-vcd/v1.0.0 ~/infrastructure-vcd/.

cp infrastructure-vcd/clusterctl.yaml 

cp infrastructure-vcd/clusterctl.yaml ~/.cluster-api/clusterctl.yaml
```
> Note: If `~/.cluster-api/clusterctl.yaml` already exists in the environment for another infrastructure provider, then do not overwrite the file. Instead use the example in this repository to append the `providers` section of the existing file 

