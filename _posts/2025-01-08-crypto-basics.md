---
layout: post
current: post
navigation: True
class: post-template
subclass: 'post'
author: shubhamdp
title: "Cryptography Basics"
date: "2024-01-08"
tags: [writing]
cover: assets/images/cover/basics-of-cryptography.png
---

I'm writing down the lessons that I've learned along the way in my professional journey.

I'll talk about the basics of the cryptography in this post.

We'll take a look at Certificate Signing Request (CSR), X.509 Certificate, and Private Key.

We'll also take a look at simple operations that can be performed using these entities.

We will run some commands to generate these entities and perform operations using them.
We will use OpenSSL for this purpose.

Parallely, we will see how to use the Python's cryptography library to perform the same operations.

I think we should also take a look at PKI (Public Key Infrastructure).

We will also take a look at the Certificate Revocation List (CRL). (Probably in a separate post)

We would also talk about the certificate chain and how to validate it. (Probably in a separate post)

> Note: All the Python code references can be found in github.com/shubhamdp/... (TODO: add link)

### PKI (Public Key Infrastructure)

PKI is a system for creating, distributing, managing, and revoking digital certificates.

Very basic components of PKI are:
- Certificate Authority (CA)
- List of issues certificates
- List of revoked certificates (CRL)

CA is a trusted entity that issues digital certificates. CA can be the root of trust or the intermediate CA.
A CA can delegate the authority to issue certificates to another CA. This CA is called the intermediate CA.

### Key Pair

A key pair consists of a public key and a private key.
The public key is shared with others, and the private key is kept secret.
Private key is used to sign the data, and the public key is used to verify the signature.

Private key is the most important part and shall be kept secret at all times. If the private key is compromised,
the attacker can impersonate the owner of the private key. The public key can be shared with anyone.

Private key shall be stored securely. If affordable, it shall be stored in a hardware security module (HSM).

- OpenSSL command generate a private key

```bash
openssl ecparam -genkey -name prime256v1 -out key.pem
```

You can dump the key details using the following command:

> NOTE: We are dumping key only for the sake of demonstration. DO NOT DUMP PRIVATE KEYS.

```bash
openssl ec -in key.pem -text -noout
```

### Certificate Signing Request (CSR)

A CSR is a message sent from an applicant to a CA to apply for a digital certificate.
Most important information in a CSR is the public key. It also contains information about the applicant,
like the requested subject name, extensions, etc.

- OpenSSL command to generate a CSR
```bash
openssl req -new -key key.pem -out csr.pem
```

To check the CSR contents, you can use the following command:
```bash
openssl req -in csr.pem -text -noout
```

### X.509 Certificate

An X.509 certificate is a digital certificate that uses the X.509 standard.

Usually, a certificate is issued by a CA to a user. The certificate contains the public key of the user,
some more details like the subject name, issuer name, validity period, few extensions, and a signature.

The signature is created using the CA's private key. The signature is created using the CA's private key.
And it can be verified using the CA's public key. One can find the CA's public key in the CA's certificate.

#### Self-Signed Certificate (Root CA)

Lets generate a self-signed certificate for a CA.

```bash
openssl req -new -x509 -noenc -newkey ec:<(openssl ecparam -name prime256v1) -keyout ca_key.pem -out ca_cert.pem -days 365
```

To verify the certificate contents, we can use the following command:
```bash
openssl x509 -in ca_cert.pem -text -noout
```

But how do you know that the certificate is self-signed?

- Issuer name is same as the subject name.
    ```bash
    openssl x509 -noout -in ca_cert.pem -issuer -subject
    ```

    <details>
    <summary>Example Output</summary>

    ```
    issuer=CN=I'm G-root
    subject=CN=I'm G-root
    ```

    </details>

- Subject key identifier (SKID) is the same as the authority key identifier (AKID).
    ```bash
    openssl x509 -in ca_cert.pem -noout -ext subjectKeyIdentifier
    openssl x509 -in ca_cert.pem -noout -ext authorityKeyIdentifier
    ```

    <details>
    <summary>Example Output</summary>

    ```
    X509v3 Subject Key Identifier:
        77:55:8F:AE:FB:C6:D7:92:C4:58:2B:67:AA:00:38:25:D8:70:62:1B
    X509v3 Authority Key Identifier:
        keyid:77:55:8F:AE:FB:C6:D7:92:C4:58:2B:67:AA:00:38:25:D8:70:62:1B
    ```

    </details>

- If we can verify the signature using the CA's public key.
    ```bash
    openssl verify -CAfile ca_cert.pem ca_cert.pem
    ```

    <details>
    <summary>Example Output</summary>

    ```
    ca_cert.pem: OK
    ```

    </details>


#### Issuing a Certificate (Intermediate CA or Leaf Certificate)

A general flow of issuing a certificate is:

1. User generates the key-pair. (This is top secret and SHALL be stored securely)
2. User generates the CSR using the public key.
3. User sends the CSR to the CA.
4. CA verifies the CSR and issues the certificate.

Lets generate a leaf certificate for a user.

We have the CA (ca_cert.pem), the user's keypair (key.pem), and users CSR (csr.pem).

We send the CSR to the CA, and the CA issues the certificate.

```bash
openssl x509 -req -in csr.pem -CA ca_cert.pem -CAkey ca_key.pem -CAcreateserial -out cert.pem -days 365
```

To verify the certificate contents, we can use the following command:
```bash
openssl x509 -in cert.pem -text -noout
```

But how do you know that the certificate is issued by the particular CA?

- Issuer name is same as the CA's subject name.
    ```bash
    openssl x509 -noout -in cert.pem -issuer
    ```

    <details>
    <summary>Example Output</summary>

    ```
    issuer=CN=I'm G-root
    ```

    </details>

- Authority key identifier (AKID) is same as the CA's SKID.
    ```bash
    openssl x509 -in cert.pem -noout -ext authorityKeyIdentifier
    ```

    <details>
    <summary>Example Output</summary>

    ```
    X509v3 Authority Key Identifier:
        keyid:77:55:8F:AE:FB:C6:D7:92:C4:58:2B:67:AA:00:38:25:D8:70:62:1B
    ```

    </details>

- If we can verify the signature using the CA's public key.
    ```bash
    openssl verify -CAfile ca_cert.pem cert.pem
    ```

    <details>
    <summary>Example Output</summary>

    ```
    cert.pem: OK
    ```

    </details>