---
title: Public Key Infrastructure — Digital Certificates with Chain of trust
date: 2022-05-22 18:30:00 +0800
categories: [Cybersecurity, Cryptography]
tags: [SSL, PKI, TLS]
render_with_liquid: true
image:
  path: /assets/img/posts/2022-05-22/cover.png
---

In this article, let’s discuss how digital certificates are working. What are the tools and eco-system associated with public-key certificates? and how the data senders and receivers are able to establish trust using those tools.

Digital certificates’ main purpose is to prevent man-in-the-middle attacks (MITM). Data need to be encrypted for communication between two parties. Digital certificates can guarantee this encryption happens strictly between those two parties only.

## Man in the middle attacks

![man in the middle attacks](/assets/img/posts/2022-05-22/man-in-the-middle.png)
_man in the middle attacks_

Man-in-the-middle attacks happen when intruders position themselves between the communication and eavesdropping on the information. Intruders do not interrupt the transmission. But tunnel the data flow through him so that he can recover the data from encrypted channels.

Man-in-the-middle attacks can be used for extended frauds such as identity thefts, phishing attacks, session hijacks, etc. These attacks can steal any information transferred between two parties, such as credit card details, and financial and personal information.

Man-in-the-middle attackers use various techniques to reroute traffic to their malicious applications. For example, with DNS spoofing, attackers poison DNS server cache records to redirect traffic to themselves. To steal information from encrypted data transmission, attackers use HTTPS spoofing and SSL session hijacking related to weak certificate validations. Also, attackers use SSL Beast exploits with TLS v1.0 to access encrypted cookies and decrypt the traffic.

## Public Key Cryptography

Public key cryptography utilizes one-way asymmetric encryption algorithms for data encryption. This technique is proper, especially when data transmission happens over the wire. Public key cryptography is the widely accepted data in transit security mechanisms. This approach has two keys that are not identical in any way. Only the other key can reverse the process and retrieve the information if one key generates the encrypted data.

Two-way encryption ensures that anybody in the world can encrypt data using the key known to the general public. However, nobody can access the information after encrypted except the private key holder. On the other hand, anyone can decrypt information when data is encrypted using a private key. But it ensures that if the public key can decrypt it, only the private key holder can generate the encrypted data, ensuring the authenticity of the data.

When a user accesses a secured website (using HTTPS), the server sends his public key and asks the client to generate and share a secret with the server by encrypting it using his public key. When a secret is encrypted, the only party able to reads the data is the server that holds the private key.

This mechanism is safe only if the client can verify the ownership of public keys. Third parties will not be able to share their public key acting as the server when the client validates the ownership. So public key verification is an essential step to preventing man-in-the-middle attacks. Digital certificates help us verify this ownership validation problem. Certificate Authorities (CA) with Public Key Infrastructure (PKI) implement a mechanism to issue and validate digital certificates.

## Validate ownership for issuing Digital certificate

Clients who need digital certificates need to create a certificate signing request (CSR) and send it to a certificate authority for validation. CSR contains essential information such as domain, organization, country details, etc. Certificate authorities issue digital certificates to certify the ownership of public keys by validating client metadata. They use different techniques to validate ownership against the provided metadata before issuing the certificate.

### Validation standards

**Domain validation:** Domain-validated certificates are called DV certificates. These certificates can secure the communication between a web server hosted under a given domain and the browser. Authorities verify the ownership of the domain from the certificate request party. DV is the lowest level of ownership validation.

**Organization validation:** Organization-validated certificates are called OV certificates. OV certificates provide an additional layer of trust on top of DV certificates. Authorities validate the legality of registered businesses and ownership of the domain in addition to domain control validation.

**Extended validation:** Extended validated certificates are called EV certificates. They provide the highest level of trust and validity. In this validation mode, Certificate authorities validate the control over the domain, their rights to use the domain, legality of the business, its physical existence, and operational status. All those verified information is embedded with the EV certificates.

## Structure of a common digital certificate

Following are the common fields of X.509 certificates. Information is stated using ‘Abstract Syntax Notation One’ (ASN.1) notation language.

```asn.1
Version number: Version of the certificate
Serial Number: Unique identity of the certificate in the certificate authority's certificate pool
Signature Algorithm: Encryption algorithm and Hashing algorithm of the signature
Signature: Body of the certificate hash and encrypted using the certificate authority's private key.
Issuer Name: The certificate authority who validated information and signed.
Validity Period: Validity period of the certificate.
  Not Before: Earliest possible date and time certificate is valid.
  Not After: Date and time which the certificate is no longer valid.
Subject: Information about the entity that the certificate belongs to.
Subject public key information: Certified public key information.
   Public key algorithm: Algorithm used by the public key
   Subject Public key: Public key data
```

## Certificate verification— Breach detection

Clients need to verify certificates to ensure that public keys certified by the certificates are trustworthy. When verifying certificates, it is not sufficient to verify the signatures of the certificates. The following steps need to carry out to verify the trustworthiness of certificates properly.

- The signature is validated against the authority’s public key
- Validate the certificate chain
- The validity period of the certificate is checked
- Certificate revoke status is checked

Clients can carry out the first three steps with the information provided by certificates. Certificate revoke status checks need additional infrastructure and utilities.

When certificate authorities revoke certificates before their expiration, it is added to Certificate Revocation List (CRL).

Certificate Authorities work as chains. Root certificate authorities certify intermediate certificate authorities, and intermediate certificate authorities can be trusted with root trust and issue personal certificates. When certificates issued to those intermediate certificate authorities (CA) are revoked, relevant records are added to Authority Revocation List (ARL). All the certificates issued under that intermediate CA certificate are invalidated when authority is revoked.

Most clients use Online Certificate Status Protocol (OCSP) to check the validity of certificates. Lightweight Directory Access Protocol (LDAP) is supported for ARL and CRL created on LDAP servers.

## Public Key Infrastructure — PKI

> Public Key Infrastructures (PKI) consists of software, hardware, roles, and policies to form an eco-system for generating, managing, validating, and distributing digital certificates.
{: .prompt-tip }

Following are some of the main components of Public Key Infrastructures.

- Registration Authority
- Certificate Authority
- Validation Authority

When Certificate Signing Request (CSR) is received, the Registration Authority (RA) of the PKI does the authentication and identification of the information specified in the signing request. CSR Approval or rejection, Certificate revocation, and certificate renewal process are happening with the RA component of PKI. Certificate owners call Registration Authorities for their public key certificate requests.

Certificate Authority (CA) does the signing and provisioning of digital certificates.

Validation Authority (VA) is responsible for verifying the validity of digital certificates. Validation Authorities handle Certificate Revocation Lists and Authority Revocation lists. Clients are calling VA to verify certificates issued by their PKI.

OpenSSL is well known simple implementation of PKI specification.

### Chain of Trust

Certificate Authorities are composed in a hierarchical order starting from Root Certificate authorities. This is a centralized trust model created to bulletproof the security of root-level authorities and expand the capability of verification and distribution operations. If Certificate Authority is compromised, it invalidates all the issued certificates.

The hierarchy of certificates makes it easy to harden the root level security and invalidate subordinate authorities in the event of a compromise. Intermediate CA can quickly validate and revoke a certificate in the event of suspicious activity.

When it comes to validation, especially when it comes to EV validations, legal requirements and processes may differ from region to region. So it is easy to have regional intermediate certificate authorities responsible for the procedures. Clients have built-in trusted root certificate collection included by the client software vendor.

When a given certificate can be traced back to a trusted root certificate, that certificate is treated as a trusted certificate. Users with super administrator privileges can add or remove root certificates in most clients. Users need to be mindful to add well-trusted, well-known root certificates to prevent the system from becoming compromised.

### Web of Trust

> Web of Trust popularized with Pretty Good Privacy (PGP) encryption mechanism.
{: .prompt-tip }

Alternative to the PKI model’s hierarchical and centralized trust model, Web of Trust implements a decentralized trust model.

The concept behind the web of trust is that every peer has trusted parties directly known to him, which is labeled as a trusted party. If one peer can trust another peer, he can inherently trust his peer’s trusted parties. So there are direct trust and indirect trusts introduced by this trusted parties model.
