# Quantum Safe Encrypted Communication Guide
Guide to sharing secret information in the age of spying and quantum computers breaking encryption. Still haven't quite figured out how to stop AI fuzzing attacks on cryptographic algorithms and agentic zero-days but I hope to God someone does. 

> [!NOTE]
> - This is for **SERIOUS COMMUNICATION**. Like sending sensitive messages/documents/files, that can jeopardize ones freedom/safety.
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

Finally, **Alice** sends `alice_signing_pub.pem` to **Bob** via email, WhatsApp or any messaging app. **Bob** does the same with his public key. Now they both have each others signing keys.

> **Verification:** Check that both key files exist and contain PEM headers by running `ls -l alice_signing_*.pem`. Also, open the received public key file in a text editor or terminal (cat alice_signing_pub.pem) to ensure it contains the `-----BEGIN PUBLIC KEY-----` header without truncation.


# Step 2: Establish a Shared Session Key (ML-KEM-1024)
To derive a shared 256-bit secret without transmitting private keys, Alice and Bob use **ML-KEM-1024** (**NIST Security Level 5**, the highest standardized bit depth).

**Alice** generates an ephemeral KEM keypair and sends her public key file (`alice_kem_pub.pem`) to Bob:
```bash
openssl genpkey -algorithm ML-KEM-1024 -out alice_kem_priv.pem
```

```bash
openssl pkey -in alice_kem_priv.pem -pubout -out alice_kem_pub.pem
```

**Bob** receives `alice_kem_pub.pem`. Bob runs the encapsulation command, which outputs a ciphertext file (`kem_ciphertext.bin`) to send back to Alice, and writes the shared secret to `shared_secret.bin`:
```bash
openssl pkeyutl -inkey alice_kem_pub.pem -pubin -encap -secret shared_secret.bin -out kem_ciphertext.bin
```

**Bob** sends kem_ciphertext.bin back to Alice via email/IM.
Alice receives `kem_ciphertext.bin` and decapsulates it using her private KEM key to recover the exact same shared secret:
```bash
openssl pkeyutl -inkey alice_kem_priv.pem -decap -secret shared_secret.bin -in kem_ciphertext.bin
```

> **Verification:** Both Alice and Bob verify their derived shared secrets match perfectly by running:
> ```bash
> sha256sum shared_secret.bin
> ```
> Both terminals must display the **EXACT SAME** SHA-256 hash string.
