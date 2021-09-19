# SKYLARK

Skylark is a RAN workload cluster.  Skylark is meant to be deployed by a Management cluster.

## Networks
Domain: skylark.cars.lab

| IP or CIDR     | USAGE        |
|----------------|:-------------|
| 10.50.0.0/24   | Cluster CIDR |
| 10.50.0.98     | apiVIP       |
| 10.50.0.99     | ingressVIP   |

## Nodes
| SERVER   | FQDN                       | LABEL        | NODE IP      | SERVICE TAG | BMC IP        | LOCATION  |
|----------|----------------------------|--------------|--------------|-------------|---------------|-----------|
| CU2      | cu2.skylark.cars.lab       | R740 XL      | 10.50.0.152  | B51TJ93     | 172.28.11.35  | LDC1      |
| MGMT1    | mgmt1.skylark.cars.lab     | R740 XL      | 172.16.0.101 | B53TJ93     | 172.28.11.36  | LDC1      |
| DU1      | du1.skylark.cars.lab       | SMCI X12     | 172.17.0.181 |             | 172.28.11.42  | FEC1      |

## Control Plane
OpenShift Control Plane for cluster, Baremetal option

| Supervisors              | MAC               | IP           | CONFIG                                   |
|--------------------------|-------------------|--------------|------------------------------------------|
| super1.skylark.cars.lab  | 40:A6:B7:51:9F:70 | 10.50.0.101  | 96 vCPU, 192G RAM, 2*480 SSD, 2x1.6 NVMe |
| super2.skylark.cars.lab  | 40:A6:B7:51:8A:60 | 10.50.0.102  | 96 vCPU, 192G RAM, 2*480 SSD, 2x1.6 NVMe |
| super3.skylark.cars.lab  | 40:A6:B7:51:51:C0 | 10.50.0.103  | 96 vCPU, 192G RAM, 2*480 SSD, 2x1.6 NVMe |

OpenShift Control Plane for cluster, Virtual Machine option

| Supervisors              | MAC               | IP           | CONFIG                                  |
|--------------------------|-------------------|--------------|-----------------------------------------|
| super1.skylark.cars.lab  | 52:52:00:11:33:11 | 172.17.0.121 | 8 vCPU, 16G RAM, 1*120GB Disk           |
| super2.skylark.cars.lab  | 52:52:00:11:33:22 | 172.17.0.122 | 8 vCPU, 16G RAM, 1*120GB Disk           |
| super3.skylark.cars.lab  | 52:52:00:11:33:33 | 172.17.0.123 | 8 vCPU, 16G RAM, 1*120GB Disk           |


## Cluster Deployment using Hive / Metal3 / OpenShift Infrastructure Operator

```bash
 cd ztp/
 ```

```bash
 oc kustomize .
 ```
 to examine what would be applied.

```bash
oc apply -k .
```

to apply manifests using Kustomize in this directory.

```bash
oc create -f 02-bmh-ran-skylark-cars-lab-RWN.yaml
```

## Verification During RAN Cluster Buildout
After feeding the ztp/02-bmh* CRD file to the management cluster, the workload cluster will start deploying.  To check:

``` bash
$ oc get agents.agent-install.openshift.io -o wide
NAME                                   CLUSTER                        APPROVED   ROLE     STAGE                  HOSTNAME
57c5d417-f114-48c2-8eb9-769bc0dd532e   cluster-ran-skylark-cars-lab   true       master   Done                   super3-vm   
73e2b7fe-5388-4502-910b-fc8796d61e55   cluster-ran-skylark-cars-lab   true       master   Waiting for bootkube   super2-vm   
81acb6aa-f73c-a5f7-a3c1-fa760c4fb43f   cluster-ran-skylark-cars-lab   true       worker   Waiting for ignition   cu2         
b1054569-9638-f04e-7c38-12ae57d754d4   cluster-ran-skylark-cars-lab   true       worker   Waiting for ignition   mgmt1       
cd4dc2da-7337-4f1f-a92f-0e2bcf5681f5   cluster-ran-skylark-cars-lab   true       master   Done                   super1-vm
```

### Check node booting and associating with OpenShift Infrastructure operator
This command can be used to get regular status from the cluster deployment:

```bash
oc get agents.agent-install.openshift.io -n assisted-installer  -o=jsonpath='{range .items[*]}{"\n"}{.spec.clusterDeploymentName.name}{"\n"}{.status.inventory.hostname}{"\n"}{range .status.conditions[*]}{.type}{"\t"}{.message}{"\n"}{end}'

cluster-ran-skylark-cars-lab
super3-vm
SpecSynced	The Spec has been successfully applied
Connected	The agent's connection to the installation service is unimpaired
RequirementsMet	The agent installation stopped
Validated	The agent's validations are passing
Installed	The installation has completed: Done

cluster-ran-skylark-cars-lab
super2-vm
SpecSynced	The Spec has been successfully applied
Connected	The agent's connection to the installation service is unimpaired
RequirementsMet	Installation already started and is in progress
Validated	The agent's validations are passing
Installed	The installation is in progress: Waiting for bootkube

cluster-ran-skylark-cars-lab
cu2
SpecSynced	The Spec has been successfully applied
Connected	The agent's connection to the installation service is unimpaired
RequirementsMet	Installation already started and is in progress
Validated	The agent's validations are passing
Installed	The installation is in progress: Waiting for ignition

cluster-ran-skylark-cars-lab
mgmt1
SpecSynced	The Spec has been successfully applied
Connected	The agent's connection to the installation service is unimpaired
RequirementsMet	Installation already started and is in progress
Validated	The agent's validations are passing
Installed	The installation is in progress: Waiting for ignition

cluster-ran-skylark-cars-lab
super1-vm
SpecSynced	The Spec has been successfully applied
Connected	The agent's connection to the installation service is unimpaired
RequirementsMet	The agent installation stopped
Validated	The agent's validations are passing
Installed	The installation has completed: Done
```

### Commands to get kubeconfig and kubeadmin password
Use the following two commands once the cluster is finished buiding to retrieve both the kubeconfig and kubeadmin credentials:

```bash
oc get secret -n assisted-installer cluster-ran-skylark-cars-lab-admin-kubeconfig -o json | jq -r '.data.kubeconfig' | base64 -d > ~/kubeconfig-skylark
oc get secret -n assisted-installer cluster-ran-skylark-cars-lab-admin-password -o json | jq -r '.data.password' | base64 -d > ~/kubeadmin-skylark
```

## Post deployment
The following manifests configure the cluster to operate as a RAN cluster.  You will see some warning messages and some "unable to recognize" and Error from server (Invalid) messages.  Run `oc apply -k` a few times.

```bash
export KUBECONFIG=~/kubeconfig-skylark
export TELCO_RANCLUSTER_PATH=~/carslab-public/rhocp-clusters/tesla.cars.lab
oc kustomize $TELCO_RANCLUSTER_PATH

oc apply -k $TELCO_RANCLUSTER_PATH
```
