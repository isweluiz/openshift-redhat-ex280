# Creating Self-signer certificates 

- PKI certificates are everywhere in OpenShift
- To secure resources - like routes - it's essential to understand how certificates are working
- To use public keys, they need to be signed by a Certificate Authority 
- Self-signed certificates are an easy way to get started with your own certificates 
- These certificates can be used in openshift 

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

### OpenSSL Video Series

- [Official Documentation](https://wiki.openssl.org/index.php/Command_Line_Utilities)

[![YouTube Video](https://img.youtube.com/vi/O1OaJmrRHrw/0.jpg)](https://www.youtube.com/watch?v=O1OaJmrRHrw&list=PLgBMtP0_D_afzNG7Zs2jr8FSoyeU4yqhi)
