# QUAY registry

The objective here is to create a registry with a short and meaningful FQDN.

## Obtain the crt and key files for the certificate

```
[nsatsia@fedora ~]$ ls -a1 lab-registry.*
lab-registry.crt.pem
lab-registry.key.pem
[nsatsia@fedora ~]$
```

## Create the config.yaml file for the registry. 
The only variable modified form a default registry deployment is the "SERVER_HOSTNAME" which **must** match the FQDN of the certificate.

### config.yaml
```    
cat << EOF > ~/config.yaml
ALLOW_PULLS_WITHOUT_STRICT_LOGGING: false
AUTHENTICATION_TYPE: Database
DEFAULT_TAG_EXPIRATION: 2w
ENTERPRISE_LOGO_URL: /static/img/RH_Logo_Quay_Black_UX-horizontal.svg
FEATURE_BUILD_SUPPORT: false
FEATURE_DIRECT_LOGIN: true
FEATURE_MAILING: false
REGISTRY_TITLE: Red Hat Quay
REGISTRY_TITLE_SHORT: Red Hat Quay
TAG_EXPIRATION_OPTIONS:
- 2w
TEAM_RESYNC_STALE_TIME: 60m
TESTING: false
SERVER_HOSTNAME: lab-registry.ocpmgmt.lab.diktio.net
EOF
```

### Bundle for registry including the certificate
```
oc create secret generic \
    --from-file config.yaml=config.yaml \
    --from-file ssl.cert=lab-registry.crt.pem \
    --from-file ssl.key=lab-registry.key.pem \
    -n quay-enterprise \
    lab-quay-config-bundle
```

### Registry CR
```
cat << EOF > lab-registry.yaml
apiVersion: quay.redhat.com/v1
kind: QuayRegistry
metadata:
  name: lab
  namespace: quay-enterprise
spec:
  components:
    - kind: clair
      managed: true
    - kind: postgres
      managed: true
    - kind: objectstorage
      managed: true
    - kind: redis
      managed: true
    - kind: horizontalpodautoscaler
      managed: true
    - kind: route
      managed: true
    - kind: mirror
      managed: true
    - kind: monitoring
      managed: false
  configBundleSecret: lab-quay-config-bundle
EOF

oc create -f lab-registry.yaml
```


