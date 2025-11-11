<!---
{
  "id": "d58d972d-7730-4b1c-8ec2-a8288ef0ed05",
  "teaches": "Understanding Cryptographic Signatures",
  "depends_on": ["e8add8e9-7a67-4b50-af89-6c1ce6558e0d"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-13",
  "keywords": ["cryptography", "digital signatures", "shell", "hashing", "security"]
}
--->

# Understanding Cryptographic Signatures

> In this exercise you will learn how to conceptually understand and practically apply cryptographic signatures. Furthermore we will explore how message authenticity can be verified and compromised using simple shell commands.

## Introduction

Cryptographic signatures are a fundamental component of digital security systems, enabling message authenticity, integrity, and non-repudiation. At their core, signatures allow a sender to attach a unique cryptographic code to a message, created using their private key, which can later be verified by anyone with access to the sender's public key. This mechanism underpins secure email, software distribution, blockchain systems, and more.

The process involves hashing the original message into a fixed-length digest using a secure hash algorithm (e.g., SHA-256). This digest is then encrypted with the sender's private key, producing the digital signature. Upon receiving the message and the signature, the recipient hashes the message independently and decrypts the signature using the sender's public key. If the two digests match, the signature is valid, confirming that the message has not been altered and originates from the claimed sender.

This exercise sheet explores a basic shell-based simulation of these steps without needing external libraries or software packages beyond common UNIX tools like `openssl`, `diff`, and file system commands. You'll also simulate an attack scenario to observe what happens when integrity checks fail.

### Further Readings and Other Sources

* [RFC 4880 - OpenPGP Message Format](https://datatracker.ietf.org/doc/html/rfc4880)
* [Introduction to Cryptography by Christof Paar from RUB](https://doi.org/10.1007/978-3-662-49893-5)
* [Digital Signatures Explained on YouTube](https://www.youtube.com/watch?v=Aq3a-_O2NcI)
* [OpenSSL Documentation](https://www.openssl.org/docs/)

## Tasks

### 1. Generate a Naive Keypair

Open a terminal and run:

```sh
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -pubout -in private_key.pem -out public_key.pem
```

This creates a 2048-bit RSA keypair: a private key (`private_key.pem`) and a public key (`public_key.pem`). The private key is used to sign messages; the public key is used to verify them.

### 2. Write and Hash a Message

Create a text file:

```sh
echo "This is a secure message." > message.txt
```

Hash the file:

```sh
openssl dgst -sha256 message.txt > hash.txt
```

This step computes the SHA-256 digest of the file `message.txt`. The output hash is a unique fingerprint of the file's contents and is saved to `hash.txt`. This hash acts as a compact representation of the entire message, ensuring that even the slightest change in the file content will produce a different hash value.

### 3. Sign the Hash

Sign the hash using the private key:

```sh
openssl rsautl -sign -inkey private_key.pem -in hash.txt -out signature.bin
```

In this step, the content of `hash.txt` (the SHA-256 digest) is encrypted using the RSA private key. This encrypted digest, stored in `signature.bin`, constitutes the digital signature. Because only the private key holder can generate this specific output, the signature proves both the origin and the integrity of the message.

### 4. Simulate Transmission

Create a new directory to simulate a remote location:

```sh
mkdir remote && cp message.txt signature.bin public_key.pem remote/
cd remote
```

### 5. Verify the Message

Recompute the hash:

```sh
openssl dgst -sha256 message.txt > new_hash.txt
```

This ensures the recipient computes the SHA-256 digest independently, using the received `message.txt` file.

Decrypt the signature:

```sh
openssl rsautl -verify -inkey public_key.pem -pubin -in signature.bin -out decrypted_hash.txt
```

This decrypts the signature using the sender’s public key. If the signature is authentic, the decrypted output should match the hash computed by the recipient.

Compare the hashes:

```sh
diff new_hash.txt decrypted_hash.txt
```

If there is no output, the two files are identical, confirming the message is authentic and unmodified.

### 6. Simulate an Attack

Before proceeding, understand the expected outcome: a digital signature is only valid for the exact content it was generated for. Any change in the content should cause a mismatch in verification.

Now, simulate an attack by altering the message:

```sh
echo "Hacked!" >> message.txt
```

This small modification represents an attacker tampering with the file after it was signed.

Rehash and compare:

```sh
openssl dgst -sha256 message.txt > attacked_hash.txt
openssl rsautl -verify -inkey public_key.pem -pubin -in signature.bin -out decrypted_hash.txt

diff attacked_hash.txt decrypted_hash.txt
```

Now `diff` should show discrepancies, indicating a failed verification. This step simulates a common security breach: an adversary modifies the content of a message without access to the private key. Because the signature was generated on the original content, it will no longer match the hash of the tampered message. Thus, the mismatch reveals that the message was altered after signing, and any attempt to forge a valid signature would require the private key. This demonstrates how cryptographic signatures safeguard integrity and authenticity by ensuring that even a minor change in content is detectable.

## Questions

1. Why does hashing the message before signing improve performance and security?
2. What would happen if we signed the entire message directly instead of its hash?
3. How would this workflow differ using elliptic curve cryptography (ECC)?
4. What risks exist if the private key is compromised?

## Advice

This exercise demonstrates the basic principles of digital signatures in a tangible, reproducible way. Use it to build an intuition for how cryptographic verification works behind the scenes. Always treat your private keys with utmost care—once leaked, the integrity of all your signed messages is void. If you're interested in more advanced topics like certificate authorities or ECC-based signing, consider checking out our follow-up sheet on [Advanced Public Key Infrastructure](#UUID-PLACEHOLDER-ADVANCED-PKI).

For editing and script exploration, stick to a reliable terminal editor like `vim`. Mastery of such tools complements your understanding of cryptographic principles with real-world command-line proficiency.
