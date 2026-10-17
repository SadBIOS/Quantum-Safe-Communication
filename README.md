# Quantum Safe Encrypted Communication Guide
Guide to sharing secret information in the age of spying and quantum computers breaking encryption. Still haven't quite figured out how to stop AI fuzzing attacks on cryptographic algorithms and agentic zero-days but I hope to God someone does. 

> [!NOTE]
> - This is for serious communication like sending sensitive messages and documents that can jeopardize ones social status/freedom/safety.
> - In simple words if the information you are about to send puts you 6 feet under then this is the right method for the occasion.

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
