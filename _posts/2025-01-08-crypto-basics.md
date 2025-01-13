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
cover: ...
---

I'm writing down the lessons that I've learned along the way in my professional journey.

I'll talk about the basics of the cryptography in this post.

We'll take a look at Certificate Signing Request (CSR), X.509 Certificate, and Private Key.

We'll also take a look at simple operations that can be performed using these entities.

We will run some commands to generate these entities and perform operations using them.
We will use OpenSSL for this purpose.

Parallely, we will see how to use the Python's cryptography library to perform the same operations.

I think we should also take a look at PKI (Public Key Infrastructure) and how it is used in the real world.

We will also take a look at the Certificate Revocation List (CRL) and how it is used in the real world.

We would also talk about the certificate chain and how to validate it.

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

<details>
<summary> OpenSSL command and Python code to generate a private key </summary>

```bash
openssl ecparam -genkey -name prime256v1 -out key.pem
```
---
```python
from cryptography.hazmat.primitives.asymmetric import ec
from cryptography.hazmat.primitives import serialization

private_key = ec.generate_private_key(ec.SECP256R1())
pem = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.TraditionalOpenSSL,
    encryption_algorithm=serialization.NoEncryption()
)
```

</details>

### Certificate Signing Request (CSR)

A CSR is a message sent from an applicant to a CA to apply for a digital certificate.
Most important information in a CSR is the public key. It also contains information about the applicant,
like the requested subject name, extensions, etc.

Below is the command to generate a CSR using OpenSSL:

```bash
openssl req -new -newkey ec -pkeyopt ec_paramgen_curve:secp256r1 -keyout key.pem -out csr.pem
```