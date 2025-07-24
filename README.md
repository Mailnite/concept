# mailnite concept

Concept of the encrypted and protected emails for Internet

Mailnite provides for free two servers:
* SMTP server that encrypts traffic automatically (could be used as relay).
* POP/IMAP server that decrypts traffic automatically.

For commercial license Mailnite provides the Kubernetes solution:
* SMTP server that encrypts traffic automatically (enterprise scalable version).
* POP/IMAP server that decrypts traffic automatically (enterprise scalable version).
* Distributed DNS server for high load web-sites.
* Web UI Interface for the email.

For mobile users Mailnite provides applications:
* iPhone payable
* Android payable

## **Sample Workflow**

1.  **Alice generates her keypair**, puts the pubkey (base64 or JWT) in DNS under  
    `_mailpubkey.alice.domain.zone`.
    
2.  **Bob wants to email Alice securely**:
    
    -   Looks up DNS record to get pubkey.
        
    -   Uses plugin to encrypt content.
        
    -   Sends to `alice@domain.zone` or `alice+key@domain.zone`. If there is a key equal the username then it would be used first.
  
    -   Several users can share the same <key>. For example alice.zeta@domain.zone and alice@domain.zone.
        
3.  **Alice’s client/plugin** sees encrypted mail, uses her private key to decrypt and display.

* * *

## **DNS TXT Record Format for Next-Gen Email Public Key**

The <key> notation of the record is TXT DNS record safe and can not have system characters.
For example if you have an email <alice.zeta@domain.zone> and also <alice@domain.zone> it would be reasonable to
use common `key=alice`.

So the protected inbox emails could be
```
alice@domain.zone
alice.zeta+alice@domain.zone
```

Whereas the `key` field in TXT record should be equal to `alice`.

Another case if you have organization or prefer to keep user names hidden from DNS scans.
You can use for the <key> common identifier for the all users `enc` or any other common name.

So the protected inbox emails could be
```
alice+enc@domain.zone
alice.zeta+enc@domain.zone
```

Whereas the `key` field in TXT record should be equal to `key`.

Mailnite provides the POP/IMAP server component for free that decrypts the traffic automatically.


### **Record Name (RR Name)**

```
_mailpubkey.<key>.<domain>.
```

_Example:_
```
_mailpubkey.alice.example.com.
```

* * *

### **Record Value (TXT Content)**

Semicolon-delimited key-value pairs, with your requested fields:

| Field | Required | Example                                           | Description                                                                     |
| ----- | -------- | ------------------------------------------------- | ------------------------------------------------------------------------------- |
| `v`   | Yes      | `v=1`                                             | Version of the record format.                                                   |
| `pk`  | Yes      | `pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J` | Public key in URL-safe Base64 (no padding).                                     |
| `alg` | Yes      | `alg=secp256k1`                                   | Key algorithm (`secp256k1`, `ed25519`, etc).                                    |
| `usr` | Optional | `usr=alice`                                       | External user/account identifier, e.g., federated ID or email.                  |
| `id`  | Optional | `id=key-id-123`                                   | Opaque external key ID for mapping/accounting.                                  |
| `iss` | Optional | `iss=https://iss.mailnite.com`                    | **Provider** URL (e.g., for commercial service, feature set, or wallet).        |
| `enc` | Optional | `enc=aes256gcm`                                   | Preferred content encryption algorithm. Default=aes256gcm                       |

Example Records
Minimal:

```perl
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;"
```

With usr and iss:

```perl
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;usr=alice@company.com;iss=https://iss.mailnite.com;"
```

With id and iss:

```perl
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;id=42b519d0-0f6a-43ba-9ef8-8725f7c1bc98;iss=https://iss.mailnite.com;"
```

For the `secp256k1` key exhange algprithm we automatically select `aes256gcm` as default encryption algorithm.

* * *

### **Field Notes**

-   **pk**: Must be a Base64-URL-safe string (A–Z, a–z, 0–9, `-`, `_`), no padding.
    
-   **iss**: Allows clients and services to build cloud services with additional features or protocols (e.g., commercial, paid, or custom mail relay).
    
-   **usr/id**: Use as needed for integration with external systems.
    
-   **All fields are optional except `v`, `pk`, and `alg`.**

### **Parsing and Extensibility**

-   Parse on `;`, then split on `=`.
    
-   Ignore unknown fields for future compatibility.
    
-   For multi-valued fields (if needed), comma-separate values, or repeat keys.
    

* * *

### **Example (shown by `dig`)**

```shell
dig +short TXT _mailpubkey.alice.example.com "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;usr=alice@company.com;iss=https://iss.mailnite.com;"
```

* * *

### **Field Notes**

-   **pk**: Should be 44 chars (for 33 bytes, unpadded) or 43-44 chars depending on encoding. If using unpadded URL-safe base64:  
    e.g., 33 bytes (secp256k1 pubkey) → 44 base64 chars with padding (`=`), or 43 chars unpadded.
    
-   **user/id**: Use one as best fits your use case. Both can be supported for flexibility.
    
-   **sig**: Base64 encoded, typically DER format for ECDSA signatures.
    
-   **TXT limitations**: Each string is up to 255 bytes; use multiple strings if needed.
    

* * *

### **Field Notes**

-   **pk**: Should be 44 chars (for 33 bytes, unpadded) or 43-44 chars depending on encoding. If using unpadded URL-safe base64:  
    e.g., 33 bytes (secp256k1 pubkey) → 44 base64 chars with padding (`=`), or 43 chars unpadded.
    
-   **usr/id**: Use one as best fits your use case. Both can be supported for flexibility.
    
-   **sig**: Base64 encoded, typically DER format for ECDSA signatures for the provider `pv`.
    
-   **TXT limitations**: Each string is up to 255 bytes; use multiple strings if needed.

* * *

### **Client Processing**

1.  **Query:** `_mailpubkey.<user>.<domain>.` (TXT)
    
2.  **Parse** key-value pairs (split on `;`), ignore unknown fields for forward compatibility.
    
3.  **Extract** `pk`, decode from Base64 URL Safe version to bytes.
    
5.  **Verify** fields as needed (`exp`, `pv`, `sig`, etc).
    
6.  **Map** `usr` or `id` field for UI or account mapping.


* * *

### DNS limits

| Limitation                 | Value (Typical)      | Notes                               |
| -------------------------- | -------------------- | ----------------------------------- |
| DNS protocol (theoretical) | No explicit max      | Bounded by practical limits         |
| Cloudflare Free            | \~3,500 records/zone | (all record types, not just TXT)    |
| AWS Route 53               | 10,000 records/zone  | (all types)                         |
| Per TXT string             | 255 bytes            | Multiple strings per record allowed |
| Zone file size             | MBs (varies)         | Practical management limit          |


* * *

# Encrypted Email

### **Proposed Fields**

**Headers in output file:**

```makefile
To: recipient@example.com
Subject: [Enc] Confidential update
Content-Type: text/plain; charset=utf-8
```

**Body:**  
Base64-encoded ciphertext

## RLP Envelope Structure (as ordered list)

| Index | Field        | Required    | Description                                                 |
| ----- | ------------ | ----------- | ----------------------------------------------------------- |
| 0     | `version`    | ✅           | Envelope version (e.g. `1`)                                 |
| 1     | `alg`        | ✅           | Recipient ECIES key algorithm (e.g. `secp256k1`)            |
| 2     | `enc`        | ✅           | Symmetric encryption algorithm (e.g. `aes256gcm`)           |
| 3     | `eph_pk`     | ✅           | Ephemeral public key used for ECIES                         |
| 4     | `nonce`      | ✅           | Nonce for symmetric encryption                              |
| 5     | `usr`        | ✅           | Recipient user/account identifier (email)                   |
| 6     | `id`         | ✅           | Key ID or fingerprint of recipient public key               |
| 7     | `iss`        | ✅           | Issuer/provider URL (e.g. `https://iss.mailnite.com`)       |
| 8     | `ct`         | ✅           | Ciphertext (encrypted payload)                              |
| 9     | `sender_alg` | ⛔️ Optional | Sender's public key algorithm (e.g. `secp256k1`, `ed25519`) |
| 10    | `sender_pk`  | ⛔️ Optional | Sender’s public key used to verify signature                |
| 11    | `sender_sig` | ⛔️ Optional | Signature over fields `[0..8]` (to prove sender)            |


## 🔧 RLP Python Encoding Template (with padding for optional fields)

```python
import rlp

envelope = [
    1,                          # version
    b"secp256k1",               # alg
    b"aes256gcm",               # enc
    eph_pk_bytes,               # ephemeral public key
    nonce_bytes,                # nonce
    usr.encode(),               # usr (recipient identifier)
    key_id_bytes,               # id (recipient key fingerprint)
    iss.encode(),               # iss (issuer/provider)
    ciphertext_bytes,           # ct (encrypted payload)
    sender_alg.encode() if sender_alg else b"",
    sender_pk_bytes if sender_pk_bytes else b"",
    sender_sig_bytes if sender_sig_bytes else b"",
]

rlp_bytes = rlp.encode(envelope)
```




