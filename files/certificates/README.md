# Creating Self-signer certificates 

- PKI certificates are everywhere in OpenShift
- To secure resources - like routes - it's essential to understand how certificates are working
- To use public keys, they need to be signed by a Certificate Authority 
- Self-signed certificates are an easy way to get started with your own certificates 
- These certificates can be used in openshift 

### Creating the CA 

```shell
mkdir openssl 
cd openssl
openssl genrsa -des3 -out myCA.key 2048
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 365 -out myCA.pem 
```

### Creating the certificate 

```shell
openssl genrsa -out tls.key 2048
openssl req -new -key tls.key -out tls.csr
```

### Self-signing the certificate

```shell
openssl x509 -req -in tls.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out tls.crt -days 1065 -sha256
```
