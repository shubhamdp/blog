---
layout: post
current: post
navigation: True
class: post-template
subclass: 'post'
author: shubhamdp
title: "Openssl Commands"
date: "2025-01-13"
tags: [writing, openssl, cryptography, security]
cover: assets/images/cover/openssl-commands.png
---

Writing down some of the openssl commands that I use frequently.

NOTE: This is just a demonstration. Please do not use these commands in production without understanding them. Printing, leaking or sharing private keys can lead to a security breach.

### Table of Contents

- [Serialization formats](#serialization-formats)
- [Key Pair](#key-pair)
- [Certificate Signing Request (CSR)](#certificate-signing-request-csr)
- [Certificate](#certificate)
- [ASN.1 (Abstract Syntax Notation One) parsing](#asn1-abstract-syntax-notation-one-parsing)
- [Certificate Revocation List (CRL)](#certificate-revocation-list-crl)

### Serialization formats

PEM (Privacy Enhanced Mail) and DER (Distinguished Encoding Rules) are two widely used formats for serializing cryptographic objects.

DER is a binary format, and PEM is a base64 encoded DER format with header and footer lines.

### Key Pair

#### Generate an elliptic curve P-256 key and serialize it to a file in PEM format

```bash
openssl ecparam -genkey -name prime256v1 -out private-key.pem
```

#### Print the contents of the private key

```bash
openssl ec -in private-key.pem -text -noout
```

#### Generate a public key from the private key

```bash
openssl ec -in private-key.pem -pubout -out public-key.pem
```

#### Print the contents of the public key

```bash
openssl ec -in public-key.pem -pubin -text -noout
```

#### PEM to DER and DER to PEM conversion

```bash
openssl ec -in private-key.pem -outform DER -out private-key.der
openssl ec -in private-key.der -inform DER -out private-key.pem
```

### Certificate Signing Request (CSR)

#### Generate a CSR using the private key

```bash
openssl req -new -key private-key.pem -out csr.pem
```

#### Print the contents of the CSR

```bash
openssl req -in csr.pem -text -noout
```

#### PEM to DER and DER to PEM conversion

```bash
openssl req -in csr.pem -outform DER -out csr.der
openssl req -in csr.der -inform DER -out csr.pem
```

### Certificate

#### Generate a self-signed certificate using the private key

```bash
openssl req -new -x509 -key private-key.pem -out certificate.pem
```

#### Print the contents of the certificate

```bash
openssl x509 -in certificate.pem -text -noout
```

#### PEM to DER and DER to PEM conversion

```bash
openssl x509 -in certificate.pem -outform DER -out certificate.der
openssl x509 -in certificate.der -inform DER -out certificate.pem
```

### ASN.1 (Abstract Syntax Notation One) parsing

#### Parse the contents of the private key

```bash
openssl asn1parse -in private-key.pem
```

#### Parse the contents of the public key

```bash
openssl asn1parse -in public-key.pem
```

#### Parse the contents of the CSR

```bash
openssl asn1parse -in csr.pem
```

#### Parse the contents of the certificate

```bash
openssl asn1parse -in certificate.pem
```

### Certificate Revocation List (CRL)

TODO: This requires setting up the CA and issuing a certificate. Will cover in the separate post.