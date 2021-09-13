# Mirror OCP images 
In this example we will be mirroring OCP 4.8.5

These instructions were based on the following reference:
    https://docs.openshift.com/container-platform/4.8/installing/installing-mirroring-installation-images.html

## Create pull secret

Obtain your pull-secret from Red Hat (link below) and place it in place in ~/pull-secret.json
 
    https://console.redhat.com/openshift/install/pull-secret



Generate a base64-encoded user name and password for Quay the Quay registry created and add it to ~/pull-secret.json

```bash
host_fqdn=lab-registry.ocpmgmt.lab.diktio.net
b64auth=$(echo -n 'nsatsia:Redhat01' | base64 -w0)
AUTHSTRING="{\"$host_fqdn\": {\"auth\": \"$b64auth\",\"email\": \"nsatsia@redhat.com\"}}"; echo $AUTHSTRING

jq -c ".auths += $AUTHSTRING" < ~/pull-secret.json > ~/pull-secret-update.json

mkdir -p ~/.docker
cp ~/pull-secret-update.json ~/.docker/config.json
```

## Mirroring the OpenShift Container Platform image repository

Set environment variables to reflect the selected version of OCP and the Quay local repository FQDN.

```bash
export OCP_RELEASE=4.8.5
export RELEASE_IMAGE=$(curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_RELEASE/release.txt | grep 'Pull From: quay.io' | awk -F ' ' '{print $3}'); echo $RELEASE_IMAGE
export UPSTREAM_REPO=${RELEASE_IMAGE}
export LOCAL_REGISTRY='lab-registry.ocpmgmt.lab.diktio.net'
export LOCAL_REPOSITORY='nsatsia/ocp485'
```

Begin the mirroring as per the oc command below, approximately 11GB of data will be downloaded.

```bash
oc adm release mirror \
  -a ~/pull-secret-update.json \
  --from=$UPSTREAM_REPO \
  --to-release-image=$LOCAL_REGISTRY/$LOCAL_REPOSITORY:${OCP_RELEASE} \
  --to=$LOCAL_REGISTRY/$LOCAL_REPOSITORY
```

### Mirror the required ocpmetal images as follows:

```
for image in \
  "assisted-installer-agent:latest" \
  "assisted-installer-controller:latest" \
  "assisted-installer:latest" \
  "assisted-service:latest"  \
  "assisted-service:latest" \
  "ocp-metal-ui:latest" \
  "postgresql-12-centos7" \
  "s3server" 
  do oc image mirror quay.io/ocpmetal/$image lab-registry.ocpmgmt.lab.diktio.net/nsatsia/$image
done
```

oc image mirror  quay.io/openshift/origin-cli:latest lab-registry.ocpmgmt.lab.diktio.net/nsatsia/origin-cli:latest

oc -a pull-secret-update.json image mirror quay.io/openshift-release-dev/ocp-release:4.8.5-x86_64 lab-registry.ocpmgmt.lab.diktio.net/nsatsia/ocp-release:4.8.5-x86_64 --insecure=true

oc -a pull-secret-update.json image mirror quay.io/openshift-release-dev/ocp-release@sha256:90a2381b1083aa91f1b635727d7852eb359405cf009ce11fc8dc726f0e00d43a lab-registry.ocpmgmt.lab.diktio.net/nsatsia/ocp485 --insecure=true

