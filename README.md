# Quantum Safe Encrypted Communication Guide
Guide to sharing secret information in the age of spying and quantum computers breaking encryption. Still haven't quite figured out how to stop AI fuzzing attacks on cryptographic algorithms and agentic zero-days but I hope to God someone does. 

> [!NOTE]
> - This is for **SERIOUS COMMUNICATION**. Like sending sensitive messages/documents/files, that can jeopardize ones freedom/safety.
> - In simple words, if the information you are about to send got out and as a result you **GET KILLED** then this is the right method for the occasion.


# Step 0: Verify OpenSSL™ & Environment
Ensure your **Debian™ 13.7 *"Trixie"*** system has an **OpenSSL™ version (3.3+)** that natively supports the **NIST** post-quantum cryptographic standards.

> Since this is the version we will be working with you are encouraged to check out the [build guide](https://github.com/SadBIOS/PQC-Compliant-Inter-Node-Bridge#manual-installation) for `OpenSSL 4.0.3 29 Sep 2026 (Library: OpenSSL 4.0.3 29 Sep 2026)`

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
To derive a shared **256-bit** (`128-bit` effective Quantum Security taking Grover's Algorithm into consideration) secret without transmitting private keys, Alice and Bob use **ML-KEM-1024** (**NIST Security Level 5**, the highest standardized bit depth).

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
> sha512sum shared_secret.bin
> ```
> Both terminals must display the **EXACT SAME** SHA-256 hash string.


# Step 3: Encrypt and Decrypt Multi-Gigabyte Files or Text
Using the shared secret one can securely encrypt anything, from a short text snippet to a multi-gigabyte file. The binary shared secret is converted into a hex key string on the fly. Now, to perform **AES-256-CBC** encryption:

> [!NOTE]
> Create and share a **NIST** recommended standard **16-Byte** (`128-Bit` or 32 Hex Characters) initialization vector beforehand (unlike secret keys this can be shared safely over the open internet).
> ```bash
> openssl rand -hex 16
> ```

> [!WARNING]
> - I personally recommend another way to get to the same initialization vector. One can do this by taking the `hash` of a publicly known entity or other known entity and use that rather than transmitting a unique one over the `clearnet`. This can reduce the risk of `de-anonymization` via `linkage attacks` or other methods of `attribution`.
> - For example:
> > ```bash
> > string=f22d85a8e2918daa7179879da32b1ceddb99a772dd6a1e4b4c7328cfd83ae57a; echo ${string:0:32}
> > ```

## Encrypt a file (e.g., a moderately sized PNG file)

Let's assume the pre-shared initialization vector is `091d3d814eaf109650f2e7d182b43d2e`

```bash
openssl enc -aes-256-cbc -salt -in alice.png -out encrypted_alice_payload.enc -K $(hexdump -v -e '/1 "%02x"' shared_secret.bin) -iv 091d3d814eaf109650f2e7d182b43d2e
```


## Decrypt the payload on the receiving end

After Alice sends Bob the `encrypted_alice_payload.enc` over email/IM. Bob runs:

```bash
openssl enc -d -aes-256-cbc -in encrypted_alice_payload.enc -out decrpted_alice.png -K $(hexdump -v -e '/1 "%02x"' shared_secret.bin) -iv 091d3d814eaf109650f2e7d182b43d2e
```

> **Verification:** Run `sha512sum <file>` to verify the hashes match byte-for-byte.


# Step 4: Double Redundancy, HMAC Tamper Detection & Quantum-Safe Identity Signing (Authentication)

In order to guarantee that the encrypted payload has not been tampered with or modified in transit, Alice signs `encrypted_alice_payload.enc` using an **HMAC-SHA256** MAC built directly from the shared session secret. Additionally, she applies a dual-layer signing process to the encrypted payload by using her **ML-DSA-87** private identity key.

> [!WARNING]
> - This section is separate but the signing **MUST** be done on the **SENDERS SIDE** and subsequently verified.
> - This step is **STRICTLY NECESSARY** because we used `AES-256-CBC` which does not have inbuilt tamper protection (like `AES-256-GCM`).
> - **ONLY DECRYPT AFTER BOTH SIGNATURES MATCH** 

**Alice generates the HMAC signature for integrity using the shared secret:**

```bash
openssl dgst -sha256 -mac HMAC -macopt hexkey:$(hexdump -v -e '/1 "%02x"' shared_secret.bin) -binary -out alice_payload.hmac encrypted_alice_payload.enc

```

**Alice** signs the encrypted payload using her post-quantum identity key (`alice_signing_priv.pem`):

```bash
openssl pkeyutl -sign -inkey alice_signing_priv.pem -rawin -in encrypted_alice_payload.enc -out alice_payload.sig

```

**Alice** sends `encrypted_alice_payload.enc`, `alice_payload.hmac` and `alice_payload.sig` to Bob.

**Bob** verifies the `HMAC` integrity using his identical shared secret:

```bash
openssl dgst -sha256 -mac HMAC -macopt hexkey:$(hexdump -v -e '/1 "%02x"' shared_secret.bin) -binary -out expected_alice.hmac encrypted_alice_payload.enc

```

```bash
cmp -s alice_payload.hmac expected_alice.hmac && echo "HMAC Verified Successfully: Ciphertext is authentic" || echo "Tampering detected!"

```

**Bob verifies Alice's digital signature using Alice's public identity key (`alice_signing_pub.pem`):**

```bash
openssl pkeyutl -verify -pubin -inkey alice_signing_pub.pem -rawin -in encrypted_alice_payload.enc -sigfile alice_payload.sig

```

> **Verification:** The HMAC comparison outputs `HMAC Verified Successfully: Ciphertext is authentic` and the OpenSSL verification command outputs `Signature Verified Successfully`, proving with double redundancy that the file originated from Alice and has not been tampered with in transit.
