IDM-LAB ocpmgmt Quay registry

#Make sure DNS is correct, refer to "Cluster DNS entries"

rm -Rf ~/certificates/lab-registry-ocpmgmt
mkdir -p ~/certificates/lab-registry-ocpmgmt
cd ~/certificates/lab-registry-ocpmgmt
rm -f ~/certificates/lab-registry-ocpmgmt/openssl.cnf
#
#cp ~/openssl_base.cnf ~/certificates/lab-registry-ocpmgmt/openssl.cnf
#



cat > openssl.cnf <<EOF
[req]
default_bits = 2048
default_md = sha256
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
encrypt_key = no
#
[req_distinguished_name]
countryName = AU
stateOrProvinceName = ACT
localityName = Canberra
0.organizationName = LAB.DIKTIO.NET
organizationalUnitName = Home Lab Pty Ltd
emailAddress = admin@lab.diktio.net
commonName = lab-registry.ocpmgmt.lab.diktio.net
#
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
#
[alt_names]
DNS.1 = lab-registry.ocpmgmt.lab.diktio.net
EOF

openssl req -config openssl.cnf -new -newkey rsa:2048 -nodes -keyout lab-registry.key.pem  -out lab-registry.csr.pem

##Get certificate
kinit admin
ipa cert-request  --principal=HTTP/lab-registry.ocpmgmt.lab.diktio.net --certificate-out lab-registry.crt.pem lab-registry.csr.pem

##Check certificate
openssl x509 -in lab-registry.crt.pem -noout -subject -issuer -enddate -ocsp_uri
openssl x509 -text -noout -in lab-registry.crt.pem

##Files you'll need
    ~/certificates/lab-registry-ocpmgmtlab-registry.key.pem 
    ~/certificates/lab-registry-ocpmgmtlab-registry.crt.pem 
    /etc/ipa/ca.crt

