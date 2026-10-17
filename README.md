# Quantum Safe Encrypted Communication Guide
Guide to sharing secret information in the age of spying and quantum computers breaking encryption. Still haven't quite figured out how to stop AI fuzzing attacks on cryptographic algorithms and agentic zero-days but I hope to God someone does. 

> [!NOTE]
> - This is for serious communication like sending sensitive messages and documents that can jeopardize ones social status/freedom/safety.
> - In simple words, if the information you are about to send got out and as a result you **GET KILLED** then this is the right method for the occasion.


# Step 0: Verify OpenSSL™ & Environment
Ensure your **Debian™ 13.7 *"Trixie"*** system has an **OpenSSL™ version (3.3+)** that natively supports the **NIST** post-quantum cryptographic standards.

> This case the version is `OpenSSL 4.0.3 29 Sep 2026 (Library: OpenSSL 4.0.3 29 Sep 2026)`

Run the following command in your terminal:

```bash
openssl version
```

```bash
openssl list -kem-algorithms | grep ML-KEM-1024
```

```bash
openssl list -signature-algorithms | grep ML-DSA-87
```

> **Verification:** Confirm that the OpenSSL version is 3.3 or higher and that both `ML-KEM-1024` and `ML-DSA-87` are listed.


# Step 1: Generate Quantum-Safe Identity Keys (ML-DSA-87)
Alice and Bob each generate their long-term identity keys to sign messages and prove their identity.
Run this on Alice's machine:
## Generate Alice's ML-DSA-87 signing private key

```bash
openssl genpkey -algorithm ML-DSA-87 -out alice_signing_priv.pem
```

## Extract Alice's public verification key

```bash
openssl pkey -in alice_signing_priv.pem -pubout -out alice_signing_pub.pem
```

**Bob** does the exact same on his machine to generate `bob_signing_priv.pem` and `bob_signing_pub.pem`.

> **Verification:** Check that both key files exist and contain PEM headers by running `ls -l alice_signing_*.pem`
