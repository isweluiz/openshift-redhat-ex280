# Creating Self-signer certificates 

- PKI certificates are everywhere in OpenShift
- To secure resources - like routes - it's essential to understand how certificates are working
- To use public keys, they need to be signed by a Certificate Authority 
- Self-signed certificates are an easy way to get started with your own certificates 
- These certificates can be used in openshift 

### OpenSSL List of commands 

```bash 
luiz---------------->>>$openssl list -commands
asn1parse         ca                ciphers           cmp               
cms               crl               crl2pkcs7         dgst              
dhparam           dsa               dsaparam          ec                
ecparam           enc               engine            errstr            
fipsinstall       gendsa            genpkey           genrsa            
help              info              kdf               list              
mac               nseq              ocsp              passwd            
pkcs12            pkcs7             pkcs8             pkey              
pkeyparam         pkeyutl           prime             rand              
rehash            req               rsa               rsautl            
s_client          s_server          s_time            sess_id           
smime             speed             spkac             srp               
storeutl          ts                verify            version           
x509              
```

### 1. **Creating the Certificate Authority (CA)**

The first part is about creating a Certificate Authority (CA), which will be used to sign the certificate.

- **Step 1: Create a directory for OpenSSL files**  
  ```shell
  mkdir openssl 
  cd openssl
  ```
  - Creates a folder named `openssl` and changes the current directory into it. This will keep all the generated files organized.

- **Step 2: Generate the private key for the CA**  
  ```shell
  openssl genrsa -des3 -out myCA.key 2048
  ```
  - This command generates a 2048-bit RSA private key encrypted with DES3 and saves it to `myCA.key`.
  - The `-des3` flag encrypts the key using a password. You’ll be prompted to enter a password when creating and using this key.

- **Step 3: Generate the CA certificate**  
  ```shell
  openssl req -x509 -new -nodes -key myCA.key -sha256 -days 365 -out myCA.pem
  ```
  - This command generates an X.509 certificate, which is a standard format for public key certificates.
  - `-key myCA.key`: Uses the previously generated private key to sign the CA certificate.
  - `-days 365`: The certificate is valid for 365 days.
  - `-out myCA.pem`: The output is saved to a file called `myCA.pem`, which is your CA certificate.
  - You’ll be prompted to fill in certificate details (such as country, organization, etc.).

### 2. **Creating the Certificate**

Next, you generate the certificate that will be signed by the CA.

- **Step 1: Generate a private key for the certificate**  
  ```shell
  openssl genrsa -out tls.key 2048
  ```
  - This command generates a 2048-bit RSA private key for the certificate, stored in `tls.key`. This key will be used to create the certificate signing request (CSR).

- **Step 2: Generate the certificate signing request (CSR)**  
  ```shell
  openssl req -new -key tls.key -out tls.csr
  ```
  - This command generates a CSR, which is a request for a certificate. It includes information about the entity requesting the certificate (like its domain, organization, etc.) and the public key.
  - The `-key tls.key`: Uses the private key generated in the previous step to create the CSR.
  - `-out tls.csr`: The output is saved in the file `tls.csr`.

  - You will be prompted to provide information such as the domain name, organization, and location. This data will be embedded in the certificate.

### 3. **Self-signing the Certificate**

Finally, the CA signs the CSR to create a certificate.

- **Step 1: Sign the CSR with the CA to generate a certificate**  
  ```shell
  openssl x509 -req -in tls.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out tls.crt -days 1065 -sha256
  ```
  - This command creates a self-signed certificate using the CSR.
  - `-in tls.csr`: This is the CSR generated earlier.
  - `-CA myCA.pem`: Specifies the CA certificate (from step 3 of CA creation).
  - `-CAkey myCA.key`: Specifies the CA’s private key to sign the certificate.
  - `-CAcreateserial`: Automatically creates a serial number for the certificate (saved as `myCA.srl`).
  - `-out tls.crt`: Saves the final signed certificate in the file `tls.crt`.
  - `-days 1065`: Specifies that the certificate is valid for 1065 days (approximately 3 years).
  
This final certificate `tls.crt` can now be used to secure services like web servers, enabling TLS/SSL encryption.

#### Inspecting the certificate
```bash
luiz---------------->>>$openssl x509 -in tls.crt -noout -text

Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            2a:08:ef:49:d8:9e:54:f9:fd:06:16:89:94:ae:91:76:ce:8c:df:30
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = Ra, L = RA, O = redhat, CN = cert.apps.ocp4.example.com
        Validity
            Not Before: Oct  8 23:06:04 2024 GMT
            Not After : Sep  8 23:06:04 2027 GMT
        Subject: C = AU, ST = Some-State, O = Internet Widgits Pty Ltd, CN = certificate.ocp4.example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d0:37:11:81:63:11:01:b8:9f:fd:d1:89:ab:1c:
                    8d:99:62:6c:09:ce:9a:7b:60:a7:ae:d2:26:d9:19:
                    03:a5:4c:2f:47:b8:04:91:b3:75:a9:03:c8:6e:f3:
                    2d:57:48:51:d0:82:f1:48:59:a8:37:c5:45:9f:4b:
                    af:81:cd:de:e2:64:e8:1c:6a:fb:07:ed:f0:ca:99:
                    37:a6:13:23:b5:84:76:96:c4:b7:53:35:0b:a9:01:
                    43:b7:2c:44:fe:db:82:11:ac:81:47:aa:4c:1d:3c:
                    97:f3:ce:31:40:5c:42:21:6c:00:77:09:b3:de:46:
                    0c:44:89:cb:e5:15:f2:22:0f:58:23:c6:e2:15:e2:
                    ef:df:44:d8:29:c7:7e:c8:6f:ae:a4:15:06:5b:f0:
                    7c:1a:20:7a:45:dc:a3:df:f9:5c:b6:5a:a4:ab:5b:
                    a4:8e:bf:3f:3b:a9:d2:ff:cd:76:9b:b0:4c:f0:0c:
                    b2:4e:6b:f9:21:9e:49:1a:1b:fb:8e:11:41:c9:eb:
                    32:9a:da:ee:a6:2a:48:55:29:d2:6f:7e:b4:a7:17:
                    0f:c8:1c:85:51:fa:d7:cc:33:e5:a7:2b:9c:d1:c5:
                    7b:bb:12:e7:16:e6:cf:2f:4a:57:a5:3c:f9:25:b3:
                    67:f7:ae:2d:49:78:1c:d8:99:57:3e:eb:3f:84:2e:
                    ca:63
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        19:04:be:59:7b:91:e0:d4:16:a0:17:e2:9c:59:46:47:67:66:
        ac:ca:f1:a4:5d:ea:48:19:aa:3d:2f:f0:c4:75:52:3f:d5:15:
        92:c0:96:61:59:32:3d:75:fe:7d:06:18:7b:bc:85:03:63:91:
        20:39:57:42:b4:52:e4:bb:18:eb:83:1c:2d:66:44:f3:6d:fb:
        f1:03:2e:d0:fc:ca:f2:f2:81:11:a1:1c:04:34:c5:2d:dc:1a:
        b8:93:35:2c:47:ca:d3:31:58:06:02:62:d4:33:f8:21:8c:84:
        49:0e:ce:9a:ae:03:a9:8d:e6:4d:49:83:20:32:27:30:6e:f1:
        d3:e9:ac:59:47:37:e8:f3:6e:c8:82:c4:72:b3:22:46:98:79:
        d7:14:2e:2d:bc:db:d4:d6:5d:9a:d0:d6:b5:0e:10:92:e7:77:
        5c:62:d8:29:2e:1c:27:fa:d3:93:44:49:59:54:0c:f6:87:dd:
        40:58:3d:bf:68:e2:c4:75:38:6c:34:23:ec:62:fd:5b:02:40:
        c2:f5:55:c2:54:4b:eb:c6:85:e6:2e:51:57:66:9a:4a:f6:f0:
        b0:0d:68:64:dd:85:6d:80:e3:fd:52:5e:cf:34:e4:40:e9:8c:
        4f:02:11:08:02:73:3e:9e:34:c8:e4:5c:8b:8c:46:f5:18:98:
        30:3c:33:8d

```


### OpenSSL Video Series

- [Official Documentation](https://wiki.openssl.org/index.php/Command_Line_Utilities)

[![YouTube Video](https://img.youtube.com/vi/O1OaJmrRHrw/0.jpg)](https://www.youtube.com/watch?v=O1OaJmrRHrw&list=PLgBMtP0_D_afzNG7Zs2jr8FSoyeU4yqhi)
