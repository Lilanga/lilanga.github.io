---
title: Creating Local Certificate Authority and approve Certificate Signing Requests
date: 2023-11-30 18:00:00 +0800
categories: [Security, Cybersecurity, Certificate Authority, Certificate Signing Request, Certificate, Public Key Infrastructure, ]
tags: [SSL, TLS, PKI, Security]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2023-11-30/cover.png
---

In this article, let's discuss creating trusted certificate authority for our devices and generating trusted certificates for HTTPS communication.

Let's use our local certificate authority to sign the certificate signing requests. Our certificate authority will not be recognized out of the box by devices or browsers, unlike authorities like Comodo, Let's Encrypt, and DigiCert. But we can make it trusted by telling our devices or browsers that this root certificate is legitimate and can be trusted for those operations. For that, we need to manually add the root certificate of our CA to the browser.

We will use the OpenSSL tool as our Public Key Infrastructure to generate the necessary certificates, keys, and signing requests.

First, we must create a root certificate for our certificate authority (CA). Root certificates represent our certificate authority and help to verify certificates signed by them using their non-disclosed private key. When we can trust a root certificate of a CA, we can verify and trust certificates signed by that authority.

## Generate the root Certificate Authority (CA) key

We will use the OpenSSL tool to generate an RSA (Rivest-Shamir-Adleman) certificate. RSA derives from the surnames of three mathematicians who introduced the algorithm. For RSA, the highest supported key length is 4096 bits. The higher the bits, the more complex it is to break the encryption. The disadvantage of higher bit lengths is that they are also slow in signing and verifications. Due to the security strength needed versus the expected performance of certificate validations, the recommendation is to use a 4096-bit length for root certificates (less frequent signings with the highest possible security for brute forcing) and 2048 bits for intermediate certificate authorities when using the RSA algorithm.

```bash
openssl genrsa -out rootCA.key 4096
```

Using the RSA algorithm, the above command will generate a random private key with a 4096 key length. This key will be our root certificate authority's private key to sign external certificate signing requests..

## Create the Root Certificate

Let's create a certificate for our authority using X.509 certificate standard. The X.509 standard is the widely accepted public key certificate standard in HTTPS communication. The certificate we generated here will be a self-signed root certificate representing the certificate authority.

```bash
openssl req -x509 -sha256 -new -nodes -key rootCA.key -days 365 -out rootCACert.crt
```

```bash
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:AU
State or Province Name (full name) [Some-State]:NSW
Locality Name (eg, city) []:Sydney
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bitsfactory
Organizational Unit Name (eg, section) []:CyberTech
Common Name (e.g. server FQDN or YOUR name) []:Bitsfactory.lilanga.me
Email Address []:info@bitsfactory.lilanga.me
```

Interactive prompts will present questions to collect information regarding the certificate. Here, we used the `sha256` algorithm to generate the certificate. Since we used the `-new` option, we will be asked a few follow-up questions to populate the certificate fields. We passed the generated root CA key file and asked not to apply DES encryption on the key file with the `-nodes` option. Our certificate validity was set for one year.

### Verify the Root Certificate content

Now we have a root certificate with us. Let's get the certificate information and verify it with the information we set in the earlier step.

```bash
openssl x509 -in .\rootCACert.crt -text -noout

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            61:14:3a:48:62:25:c3:9e:ca:fe:c3:49:ba:fd:c7:8a:ac:7d:f3:9b
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = AU, ST = NSW, L = Sydney, O = Bitsfactory, OU = CyberTech, CN = Bitsfactory.lilanga.me, emailAddress = info@bitsfactory.lilanga.me
        Validity
            Not Before: Nov 30 11:45:37 2023 GMT
            Not After : Nov 29 11:45:37 2024 GMT
        Subject: C = AU, ST = NSW, L = Sydney, O = Bitsfactory, OU = CyberTech, CN = Bitsfactory.lilanga.me, emailAddress = info@bitsfactory.lilanga.me
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                ### REST IS REMOVED FOR CLARITY ###
```

## Create Client certificates using Root Certificate Authority

We have a root certificate authority with a valid certificate. Now, our root certificate authority can sign client's certificate signing requests. Let's create a key pair to be used in a REST API and generate a certificate for the public key signed by our root certificate authority.

### Generate a private key for the application server

Here, we are creating a private key to be used with our application server and certifying the public key of that private key using our Certificate Authority. We will prepare a Certificate Signing Request (CSR) for that process and ask the authority to sign it.

Let's generate a private key with a 2048 length for TLS encryptions.

```bash
openssl genrsa -out server.key 2048
```

We use a 2048 key length here since this is a server key. This key length is perfect for security as well as performance when it comes to securing HTTPS-based server communications.

### Create Certificate Signing Request for the server certificate

When getting a trusted certificate from a certificate authority, an external party requests it from the authority with the necessary information to prove that they are the legitimate owners of the entity they represent, and the certificate authority needs to verify the legitimacy of the request.

We must submit our information to the certificate authority for validation as a Certificate Signing Request. Certificate Authority will check the information, and they will decide to sign the request if the provided information is valid. So, we need to create a certificate signing request with all the required information.

#### Create Certificate Config file for certificate metadata

Last time, when creating our certificate, we used the -new keyword and generated a certificate with the required information using the interactive prompt. This time, however, let's follow a different approach.

First, let's create `certConfig.cnf` file containing certificate information.

```cnf
[req]
default_bits = 2048
req_extensions = v3_req
distinguished_name = req_distinguished_name
prompt = no

[req_distinguished_name]
CN = www.bitsfactory.org
C = SG
L = Serangoon
O = Bitsfactory
OU = CyberSecurity

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth,serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = io.bitsfactory.org
DNS.2 = test.bitsfactory.org
DNS.3 = localhost
DNS.3 = 127.0.0.1
```

#### Generating Certificate Signing Request for the server public key

Let's generate a CSR file with `certConfig.cnf` configurations for the public key of the key pair we generated earlier.

```bash
openssl req -new -key server.key -sha256 -config certConfig.cnf -out server.csr
```

Now, we have a certificate signing request for our public key. Let's verify the content of the CSR.

```bash
openssl req -text -noout -verify -in server.csr
```

```bash
Certificate request self-signature verify OK
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = www.bitsfactory.org, C = SG, L = Serangoon, O = Bitsfactory, OU = CyberSecurity
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
```

Now we have a Certificate Signing Request with all the valid information.

### Create a `v3.ext` file for the certificate request

For trusted certificates, we need to specify additional extensions the certificate is used for. This file is often called `v3.ext`. These extension configurations may include details like subject alternative names (SANs), key usage constraints, and other attributes.

Lets create `v3.ext` file with the following content

```bash
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.bitsfactory.org
```

The next step is to submit CSR and V3.ext files to the Certificate authority for their validation. Since we are using our local certificate authority, let's proceed to sign our CSR.

### Signing Certificate Signing Request (CSR) and generating a certificate for the server

We need to forward our CSR with `v3.ext` to a certificate authority. They will validate the request and sign to generate the certificate if the given particulars are valid.

Since we are the Certificate Authority here, let's use our CA private key to sign the Certificate Signing Request.

```bash
openssl x509 -req -sha256 -in server.csr -CA rootCACert.crt -CAkey rootCA.key -days 90 -CAcreateserial -extfile v3.ext -out server.crt
```

### Validate Certificate

Let's validate the certificate we generated using our OpenSSL Public Key Infrastructure (PKI) tool.

```bash
openssl x509 -in server.crt -text -noout
```

```bash
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            3e:47:e6:4c:33:80:1b:2d:7d:38:1b:40:c5:42:c4:0f:c7:a4:a5:3c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = AU, ST = NSW, L = Sydney, O = Bitsfactory, OU = CyberTech, CN = Bitsfactory.lilanga.me, emailAddress = info@bitsfactory.lilanga.me
        Validity
            Not Before: Nov 30 12:21:12 2023 GMT
            Not After : Feb 28 12:21:12 2024 GMT
        Subject: CN = www.bitsfactory.org, C = SG, L = Serangoon, O = Bitsfactory, OU = CyberSecurity
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
```

Bravo! We have a valid certificate signed by our in-house (Well Trusted?) Root Certificate Authority.

## Adding certificate of Certificate Authority(CA) to trusted store

To make our certificate authority a trusted certificate authority, we need to add the Root certificate of the CA to the trusted certificate store of our devices. When we do that, certificates issued by that CA will be trusted by the device.

In MacOS, you can add the CA certificate to the login keychain and mark that certificate as `Always Trust`

Add the root certificate to the login certificate collection (To make it system-wide, you can add it to the system keychain)

![Add root certificate to the login certificate collection](/assets/img/posts/2023-11-30/keychain-access.png "Add root certificate to the login certificate collection")

Then double-click on the certificate and mark it as `Always Trust` for the required categories
![Certificate Trust Settings](/assets/img/posts/2023-11-30/certificate-trust-settings.png "Certificate Trust Settings")

Now, our device is trusting the Certificate Authority as a trusted entity. The device will trust any valid certificate issued by this Certificate Authority.

It's time to create a simple NodeJS service and use our newly created certificates for HTTPS communication.

## HTTPS enabled NodeJS API using the signed server certificate

Let's create a simple Golang API service and enable HTTPS communication using our signed certificates.

```go
package main

import (
 "fmt"
 "net/http"
)

func main() {
 http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, this is a secure Go web service!")
 })

 // Signed server certificate and key
 certFile := "./certs/server.crt"
 keyFile := "./certs/server.key"

 // Start the HTTPS server
 err := http.ListenAndServeTLS(":443", certFile, keyFile, nil)
 if err != nil {
  fmt.Println("Error starting server:", err)
 }
}
```

Following is the same implementation using `NodeJS`

```javascript
const https = require(`https`);
const fs = require(`fs`);

const options = {
  key: fs.readFileSync('../certs/server.key'),
  cert: fs.readFileSync('../certs/server.crt')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end(`Hello, this is a secure NodeJS web service!\n`);
}).listen(443);
```

Our certificate is issued for `www.bitsfactory.org`, which is the common name. So, this service needs to be served from the `www.bitsfactory.org` domain to be identified as trusted by browsers or clients.

![Client Certificate](/assets/img/posts/2023-11-30/client-certificate.png "Client Certificate")

Let's tweak our `hosts` file to redirect `www.bitsfactory.org` traffic to localhost. In MacOS, you can achieve this by editing the `/etc/hosts` file as follows.

```bash
sudo nvim /etc/hosts
```

and add the following line to the end of hosts file

```sh
127.0.0.1 www.bitsfactory.org
```

Viola. Everything is in place now. We can go to `www.bitsfactory.org` while local service is running in the background to access our secured web service with a trusted certificate.

![Web service screenshot](/assets/img/posts/2023-11-30/web-service-screenshot.png "web service screenshot")

## Summary

Here, we created Certificate Authority with a strong key and asked our devices/browsers to trust the authority certificate as a valid entity. Then, we generated a Certificate Signing Request (CSR) for our web service and asked our Certificate Authority to sign it. We used the certificate issued by the authority to create an HTTPS web service. Since we have a trusted certificate issued by a recognized CA for the device, and certificate validations are passing, our service is marked as secured by the browser's security validations.

You can use this approach to create trusted CA for our testing environments and generate certificates for each and every service without using self-signed certificates. (My recommendation goes to tools like CertBot, however).

I hope this will give you a better understanding of how `Chain of Trust` and certificate validations work together in secure communications.
