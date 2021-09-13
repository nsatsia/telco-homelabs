# Telco Managment Cluster

The Telco management cluster must be OCP 4.8.x and be deployed using one of the following methods:

- Assisted Installer / Assisted Service Operator mode
- Baremetal IPI mode

These method are the only methods that deploy and configure the `cluster-baremetal-operator` in the cluster which is a requirement for some of the automation flows.

**NOTE:** Due to `BZ-1953979` the minimum version for mgmt cluster is `4.8.0-0.nightly-2021-05-31-085539`

##

One the cluster is deployed using one of the valid deployment methods for Telco Management Clusters

```bash
export KUBECONFIG="~/path/to/kubeconfig-telco-mgmt"
export TELCO_MGMT_PATH="~/path/to/this/path"
oc apply -k $TELCO_MGMT_PATH

export KUBECONFIG="~/ocpsmc"
export TELCO_MGMT_PATH="~/Documents/vscode/telco-homelabs/clusters/ocpsmc.lab.diktio.net/"
oc apply -k $TELCO_MGMT_PATH

```

- Sample run from a directory containing a copy of this repository.

```bash
# "oc" should have access to the management cluster 
$ oc whoami --show-server
https://api.mgmt.telco.shift.zone:6443

# label all nodes for LSO
$ oc label node -l kubernetes.io/os=linux ran.openshift.io/lso=''
node/m0 labeled
node/m1 labeled
node/m2 labeled

# apply role to 
$ oc apply -k .
namespace/assisted-installer configured
namespace/open-cluster-management created
namespace/openshift-local-storage configured
namespace/openshift-serverless configured
namespace/telco-gitops configured
serviceaccount/cli-job-sa configured
clusterrole.rbac.authorization.k8s.io/cli-job-sa-role configured
clusterrole.rbac.authorization.k8s.io/telco-gitops-custom-role configured
clusterrolebinding.rbac.authorization.k8s.io/cli-job-sa-rolebinding configured
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-group created
...<snip>...
error: unable to recognize ".": no matches for kind "MultiClusterHub" in version "operator.open-cluster-management.io/v1"
```

- Ignore the "no matches for kind '...<a_kind_here>...' " errors. The reason for these is that the Operators are installing but the deployment has not compelted by the time it tries to configure the CRs in the first run.
  - Once the Operators are installed rerun the `oc apply -k .`
- Disconnection for the ingressVIP and apiVIP might be experienced as the configuration is modifying the corresponding operator configuration. There is a rolling update that is also kickstarted by any MCO configuration update applied to nodes. After few minutes the base configuration should be completed
- (optional) Set a default storageclass

    ```bash
    # set lso-filesystemclass as default
    oc patch storageclass lso-filesystemclass \
    -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

- Create secret from pull-secret for Assisted Installer Operator

    ```bash
    oc create secret generic assisted-deployment-pull-secret -n assisted-installer \
    --from-file=.dockerconfigjson=pull-secret.json --type=kubernetes.io/dockerconfigjson
    ```

- Patch metal3 so it can see all the `bmh` resources in all namespaces:
  
    ```bash
    oc patch provisioning provisioning-configuration --type merge -p '{"spec":{"watchAllNamespaces": true}}'
    ```
- To obtain the password for `telco-gitops` ArgoCD `admin`
  
    ```bash
    oc get secret -n telco-gitops telco-gitops-cluster -o go-template='{{index .data "admin.password"}}' | base64 -d
    ```

## Define a cluster for ArgoCD

After a management cluster is deployed, before using ArgoCD for day-2 GitOps configurations, the cluster credentials must be defined in ArgoCD.

```bash
# Linux argocd CLI 
curl -L -o argocd https://github.com/argoproj/argo-cd/releases/download/v2.0.5/argocd-linux-amd64

# MacOS argocd CLI
curl -L -o argocd https://github.com/argoproj/argo-cd/releases/download/v2.0.5/argocd-darwin-amd64
```

- Login into the ArgoCD instance in the management cluster using ArgoCD CLI. You will be prompted for the ArgoCD `admin` password.

    ```bash
    # User admin, password from above
    argocd login telco-gitops-server-telco-gitops.apps.ocpmgmt.lab.diktio.net
    ---argocd login https://api.mgmt.telco.shift.zone:6443 --name admin---
    ```

- List clusters
  
    ```bash
    $ argocd cluster list
    SERVER                                  NAME        VERSION  STATUS      MESSAGE
    https://kubernetes.default.svc          in-cluster  1.21+    Successful
    ```

- Add target cluster (assumes target cluster already deployed)

    ```bash
    $ argocd cluster add --kubeconfig ~/kubeconfig-telco-core admin --name telco-core
    INFO[0000] ServiceAccount "argocd-manager" created in namespace "kube-system"
    INFO[0000] ClusterRole "argocd-manager-role" created
    INFO[0000] ClusterRoleBinding "argocd-manager-role-binding" created
    Cluster 'https://api.core.telco.shift.zone:6443' added
    ```

- Validate cluster has been defined

    ```bash
    argocd cluster list
    SERVER                                  NAME        VERSION  STATUS      MESSAGE
    https://api.core.telco.shift.zone:6443  telco-core  1.20     Successful
    https://kubernetes.default.svc          in-cluster  1.21+    Successful
    ```
