# PKI-infrastructure
A)   Design and build a PKI infrastructure that includes Root CA, Signing CA, and TLS Certificate, as described here http://pki-tutorial.readthedocs.io/en/latest/simple/

B)   Use the TLS certificate to install a web server, e.g. tomcat, https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.Links to an external site.htmlLinks to an external site.

Document progress, including screenshots/recording of web browser showing you are securely connected to the site you have created, and submit Word document including github reference to code, etc..
## DJNS Homework 8: PKI Infrastructure Design and Implementation
Overview
Welcome to the DJNS group's repository for Homework 8! In this assignment, we will be designing and building a Public Key Infrastructure (PKI) that includes a Root Certificate Authority (CA), a Signing CA, and a TLS Certificate. The project follows the guidelines outlined in the PKI Tutorial.


# 1. Create Root CA
## 1.1 Create directories

```
mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/31fcba90-c322-45ca-aea9-92e55ae0f168)

The ca directory holds CA resources, the crl directory holds CRLs, and the certs directory holds user certificates.

## 1.2 Create database

```
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/32007164-d2b2-4340-89b1-a46b6a5b6a26)

The database files must exist before the openssl ca command can be used. The file contents are described in Appendix B: CA Database.

## 1.3 Create CA request
```
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/9922d530-2ca0-4269-bd1b-67bdb7e06b54)

With the openssl req -new command we create a private key and a certificate signing request (CSR) for the root CA. You will be asked for a passphrase to protect the private key. The openssl req command takes its configuration from the [req] section of the configuration file.

## 1.4 Create CA certificate
```
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/cbb1b788-b242-4ebc-a4ad-cbbcd5aa5da0)


With the openssl ca command we issue a root CA certificate based on the CSR. The root certificate is self-signed and serves as the starting point for all trust relationships in the PKI. The openssl ca command takes its configuration from the [ca] section of the configuration file.

# 2. Create Signing CA
## 2.1 Create directories

```
mkdir -p ca/signing-ca/private ca/signing-ca/db crl certs
chmod 700 ca/signing-ca/private
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/79b945d1-5bd1-4f46-9f54-cb2f3f6f1d2d)

The ca directory holds CA resources, the crl directory holds CRLs, and the certs directory holds user certificates. We will use this layout for all CAs in this tutorial.

## 2.2 Create database
```
cp /dev/null ca/signing-ca/db/signing-ca.db
cp /dev/null ca/signing-ca/db/signing-ca.db.attr
echo 01 > ca/signing-ca/db/signing-ca.crt.srl
echo 01 > ca/signing-ca/db/signing-ca.crl.srl
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/0edff96d-7b81-4809-ae96-6cae66fced8f)

The contents of these files are described in Appendix B: CA Database.

## 2.3 Create CA request
```
openssl req -new \
    -config etc/signing-ca.conf \
    -out ca/signing-ca.csr \
    -keyout ca/signing-ca/private/signing-ca.key
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/9f24842a-c959-4aaf-b7ef-1b593db82e5d)

With the openssl req -new command we create a private key and a CSR for the signing CA. You will be asked for a passphrase to protect the private key. The openssl req command takes its configuration from the [req] section of the configuration file.

## 2.4 Create CA certificate
```openssl ca \
    -config etc/root-ca.conf \
    -in ca/signing-ca.csr \
    -out ca/signing-ca.crt \
    -extensions signing_ca_ext
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/ee609305-3c45-4e3d-bb82-651c920184d7)

With the openssl ca command we issue a certificate based on the CSR. The command takes its configuration from the [ca] section of the configuration file. Note that it is the root CA that issues the signing CA certificate! Note also that we attach a different set of extensions.

# 3. Operate Signing CA
## 3.1 Create email request
```
openssl req -new \
    -config etc/email.conf \
    -out certs/fred.csr \
    -keyout certs/fred.key
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/49082814-b750-40ac-99be-2a3c179bfe3b)

With the openssl req -new command we create the private key and CSR for an email-protection certificate. We use a request configuration file specifically prepared for the task. When prompted enter these DN components: DC=org, DC=simple, O=Simple Inc, CN=Fred Flintstone, emailAddress=fred@simple.org. Leave other fields empty.

## 3.2 Create email certificate
```
openssl ca \
    -config etc/signing-ca.conf \
    -in certs/fred.csr \
    -out certs/fred.crt \
    -extensions email_ext
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/caf59f6d-66dc-447a-87ea-dcf7174f944e)

We use the signing CA to issue the email-protection certificate. The certificate type is defined by the extensions we attach. A copy of the certificate is saved in the certificate archive under the name ca/signing-ca/01.pem (01 being the certificate serial number in hex.)

## 3.3 Create TLS server request
```
SAN=DNS:www.simple.org \
openssl req -new \
    -config etc/server.conf \
    -out certs/simple.org.csr \
    -keyout certs/simple.org.key
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/e2bf37be-4622-4d1f-a0b9-f996cc7d7e1f)

Next we create the private key and CSR for a TLS-server certificate using another request configuration file. When prompted enter these DN components: DC=org, DC=simple, O=Simple Inc, CN=www.simple.org. Note that the subjectAltName must be specified as environment variable. Note also that server keys typically have no passphrase.

## 3.4 Create TLS server certificate
```
openssl ca \
    -config etc/signing-ca.conf \
    -in certs/simple.org.csr \
    -out certs/simple.org.crt \
    -extensions server_ext
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/9f9b2fc8-a789-4ead-97c5-401ff1d05a9e)

We use the signing CA to issue the server certificate. The certificate type is defined by the extensions we attach. A copy of the certificate is saved in the certificate archive under the name ca/signing-ca/02.pem.

## 3.5 Revoke certificate
```
openssl ca \
    -config etc/signing-ca.conf \
    -revoke ca/signing-ca/01.pem \
    -crl_reason superseded
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/b6382aef-1498-4b0a-b95a-dbdda6ad0823)

Certain events, like certificate replacement or loss of private key, require a certificate to be revoked before its scheduled expiration date. The openssl ca -revoke command marks a certificate as revoked in the CA database. It will from then on be included in CRLs issued by the CA. The above command revokes the certificate with serial number 01 (hex).

## 3.6 Create CRL
```
openssl ca -gencrl \
    -config etc/signing-ca.conf \
    -out crl/signing-ca.crl
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/f85f55f1-2b88-4524-a00b-2b8669962488)

The openssl ca -gencrl command creates a certificate revocation list (CRL). The CRL contains all revoked, not-yet-expired certificates from the CA database. A new CRL must be issued at regular intervals.


# 4. Output Formats
## 4.1 Create DER certificate
```openssl x509 \
    -in certs/fred.crt \
    -out certs/fred.cer \
    -outform der
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/3254388a-7703-4848-8b93-9c4af54c52d7)

All published certificates must be in DER format [RFC 2585#section-3]. Also see Appendix A: MIME Types.

## 4.2 Create DER CRL
```
openssl crl \
    -in crl/signing-ca.crl \
    -out crl/signing-ca.crl \
    -outform der
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/5c7b3522-f888-408f-93cc-2bb55d1a8f01)

All published CRLs must be in DER format [RFC 2585#section-3]. Also see Appendix A: MIME Types.


## 4.3 Create PKCS#7 bundle
```
openssl crl2pkcs7 -nocrl \
    -certfile ca/signing-ca.crt \
    -certfile ca/root-ca.crt \
    -out ca/signing-ca-chain.p7c \
    -outform der
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/4a52a249-4a2a-4082-b3e1-3a6d6fb55a30)

PKCS#7 is used to bundle two or more certificates. The format would also allow for CRLs but they are not used in practice.

## 4.4 Create PKCS#12 bundle
```
openssl pkcs12 -export \
    -name "Fred Flintstone" \
    -inkey certs/fred.key \
    -in certs/fred.crt \
    -out certs/fred.p12
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/5711de8a-9e52-46d3-bbae-f09709ee8cfe)

PKCS#12 is used to bundle a certificate and its private key. Additional certificates may be added, typically the certificates comprising the chain up to the Root CA.

## 4.5 Create PEM bundle
``` 
cat ca/signing-ca.crt ca/root-ca.crt > \
    ca/signing-ca-chain.pem

cat certs/fred.key certs/fred.crt > \
    certs/fred.pem
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/ab13a74d-1099-4cb9-a837-97c343f00b2b)

PEM bundles are created by concatenating other PEM-formatted files. The most common forms are “cert chain”, “key + cert”, and “key + cert chain”. PEM bundles are supported by OpenSSL and most software based on it (e.g. Apache mod_ssl and stunnel.)

# 5. View Results
## 5.1 View request
```
openssl req \
    -in certs/fred.csr \
    -noout \
    -text
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/58c6c687-d985-4c9d-b931-8a8a321cc70d)

The openssl req command can be used to display the contents of CSR files. The -noout and -text options select a human-readable output format.

## 5.2 View certificate
```
openssl x509 \
    -in certs/fred.crt \
    -noout \
    -text
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/044db8a1-6941-4523-85d4-91abaf292ab7)

The openssl x509 command can be used to display the contents of certificate files. The -noout and -text options have the same purpose as before.

## 5.3 View CRL
```openssl crl \
    -in crl/signing-ca.crl \
    -inform der \
    -noout \
    -text
```
The openssl crl command can be used to view the contents of CRL files. Note that we specify -inform der because we have already converted the CRL in step 4.2.

## 5.4 View PKCS#7 bundle
```openssl pkcs7 \
    -in ca/signing-ca-chain.p7c \
    -inform der \
    -noout \
    -text \
    -print_certs
```
The openssl pkcs7 command can be used to display the contents of PKCS#7 bundles.

## 5.5 View PKCS#12 bundle
```openssl pkcs12 \
    -in certs/fred.p12 \
    -nodes \
    -info
```
The openssl pkcs12 command can be used to display the contents of PKCS#12 bundles.

# We will use the generated TLS certificate above to configure Tomcat to work with SSL. For that , we need to generate a keystore which has the complete certificate chain beginning from the root and up until the server.

## Prepare the keystore file and add the root and signing certificate:

```
keytool -import -alias root-ca -keystore certs/simple.jks -trustcacerts -file ca/root-ca.crt
keytool -import -alias signing-ca -keystore certs/simple.jks -trustcacerts -file ca/signing-ca.crt
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/f9ae2136-6e45-4836-be25-0cae3fd55ffd)
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/de812a38-8f75-4e53-a4c3-df52ad6c0c7a)


## Generate the certificate-key pair for the server:
```
openssl pkcs12 -export -name "tomcat" -inkey certs/simple.org.key -in certs/simple.org.crt -out certs/simple.p12
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/cd6716d9-b032-43c1-82d3-7535e0972e80)


## Import the certificate-key pair to the keystore:

```
keytool -importkeystore -srckeystore certs/simple.p12 -destkeystore certs/simple.jks -srcstoretype pkcs12 -alias tomcat
```
![image](https://github.com/J-M0101/PKI-infrastructure/assets/115431730/467a6da2-e85c-4e91-ab58-76bb291ed39f)


## Once the keystore is ready, then in Tomcat, the SSL configuration needs to be made in “server.xml” which is located in the “conf” directory of the Tomcat installation.

```
<Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="443" maxThreads="200"
           maxParameterCount="1000"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="/Users/joshuamedina/Desktop/148Project/PKI-infrastructure/certs/simple.jks" keystorePass="pass123"
           clientAuth="false" sslProtocol="TLS"/>
```
