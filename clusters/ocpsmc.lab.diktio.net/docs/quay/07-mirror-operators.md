# NEEDS EDITING

AA Mirroring an Operator catalog

 https://docs.openshift.com/container-platform/4.8/operators/admin/olm-restricted-networks.html#olm-mirror-catalog_olm-restricted-networks

#Not needed if ~/.docker/config.json is current

podman login registry.redhat.io

podman login lab-registry.ocpmgmt.lab.diktio.net



-------------------------------------
##WORKS
    ~/opm index prune \
        -f registry.redhat.io/redhat/redhat-operator-index:v4.8 \
        -p  ansible-automation-platform-operator,compliance-operator,container-security-operator,costmanagement-metrics-operator,kubernetes-nmstate-operator,kubevirt-hyperconverged,local-storage-operator,metering-ocp,mtv-operator,nfd,ocs-operator,openshift-gitops-operator,openshift-jenkins-operator,openshift-pipelines-operator-rh,performance-addon-operator,ptp-operator,quay-bridge-operator,quay-operator,rhacs-operator,rhmtv-operator,rhsso-operator,sandboxed-containers-operator,serverless-operator,service-registry-operator,service-telemetry-operator,servicemeshoperator,skupper-operator,smart-gateway-operator,sriov-network-operator,submariner,vertical-pod-autoscaler,web-terminal \
        -t lab-registry.ocpmgmt.lab.diktio.net/nsatsia/redhat-operator-index:v4.8

    podman push lab-registry.ocpmgmt.lab.diktio.net/nsatsia/redhat-operator-index:v4.8

    oc adm catalog mirror \
        lab-registry.ocpmgmt.lab.diktio.net:443/nsatsia/redhat-operator-index:v4.8 \
        lab-registry.ocpmgmt.lab.diktio.net:443/nsatsia \
        -a pull-secret-update.json \
        --index-filter-by-os='linux/amd64' 
--------------------------------------------------------
#MESSY AND NOT TESTED
oc adm catalog mirror \
    registry.redhat.io/redhat/redhat-operator-index:v4.8 \
    lab-registry.ocpmgmt.lab.diktio.net:443/nsatsia/rh-operators \
    -a ~/pull-secret-update.json \
    --insecure \
    --index-filter-by-os='linux/amd64' \
    --manifests-only

```
[nsatsia@fedora ~]$ oc adm catalog mirror     registry.redhat.io/redhat/redhat-operator-index:v4.8     lab-registry.ocpmgmt.lab.diktio.net:443/nsatsia/rh-operators     -a ~/pull-secret-update.json     --insecure     --index-filter-by-os='linux/amd64'     --manifests-only
src image has index label for database path: /database/index.db
using database path mapping: /database/index.db:/tmp/236377048
W0818 16:08:17.784009 1809646 manifest.go:442] Chose linux/amd64 manifest from the manifest list.
wrote database to /tmp/236377048
using database at: /tmp/236377048/index.db
.
.
wrote mirroring manifests to manifests-redhat-operator-index-1629266891
[nsatsia@fedora ~]$

[nsatsia@fedora ~]$ ls -al manifests-redhat-operator-index-1629266891/
total 1260
drwxrwxr-x.  2 nsatsia nsatsia    4096 Aug 18 16:08 .
drwx--x--x. 67 nsatsia nsatsia   12288 Aug 18 16:08 ..
-rwxrwxr-x.  1 nsatsia nsatsia     266 Aug 18 16:08 catalogSource.yaml
-rwxrwxr-x.  1 nsatsia nsatsia  129139 Aug 18 16:08 imageContentSourcePolicy.yaml
-rw-rw-r--.  1 nsatsia nsatsia 1137228 Aug 18 16:08 mapping.txt
[nsatsia@fedora ~]$ 
```
