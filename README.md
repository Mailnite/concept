# concept
Concept of the encrypted and protected emails for Internet


## **DNS TXT Record Format for Next-Gen Email Public Key**

### **Record Name (RR Name)**

php-template

CopyEdit

`_mailpubkey.<user>.<domain>.`

_Example:_  
`_mailpubkey.alice.example.com.`


### **Record Value (TXT Content)**

Semicolon-delimited key-value pairs, with your requested fields:

| Field | Required | Example                                           | Description                                                                     |
| ----- | -------- | ------------------------------------------------- | ------------------------------------------------------------------------------- |
| `v`   | Yes      | `v=1`                                             | Version of the record format.                                                   |
| `pk`  | Yes      | `pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J` | Public key in URL-safe Base64 (no padding).                                     |
| `alg` | Yes      | `alg=secp256k1`                                   | Key algorithm (`secp256k1`, `ed25519`, etc).                                    |
| `exp` | Optional | `exp=2026-01-01`                                  | Expiry date for key rotation (ISO 8601, UTC).                                   |
| `usr` | Optional | `usr=alice@company.com`                           | External user identifier, e.g., federated ID or email.                          |
| `id`  | Optional | `id=external-id-123`                              | Opaque external/user ID for mapping/accounting.                                 |
| `pv`  | Optional | `pv=mailnite`                                     | **Provider** identifier (e.g., for commercial service, feature set, or wallet). |
| `sig` | Optional | `sig=MEUCIQDv4uHKsThvX...`                        | Signature over the record (Base64/DER, optional).                               |
| `enc` | Optional | `enc=aes256gcm`                                   | Preferred content encryption algorithm.                                         |

Example Records
With usr and pv:

```perl
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;exp=2026-01-01;usr=alice@company.com;pv=mailnite;"
```

With id and pv:

```perl
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;exp=2026-01-01;id=42b519d0-0f6a-43ba-9ef8-8725f7c1bc98;pv=mailnite;"
```

With signature and encryption:
```
_mailpubkey.alice.example.com. IN TXT "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;exp=2026-01-01;usr=alice@company.com;pv=mailnite;enc=aes256gcm;sig=MEUCIQDv4uHKsThvX...;"
```

### **Field Notes**

-   **pk**: Must be a Base64-URL-safe string (A–Z, a–z, 0–9, `-`, `_`), no padding.
    
-   **pv**: Allows clients and services to recognize if additional features or protocols are supported for the user (e.g., commercial, paid, or custom mail relay).
    
-   **usr/id**: Use as needed for integration with external systems.
    
-   **All fields are optional except `v`, `pk`, and `alg`.**

### **Parsing and Extensibility**

-   Parse on `;`, then split on `=`.
    
-   Ignore unknown fields for future compatibility.
    
-   For multi-valued fields (if needed), comma-separate values, or repeat keys.
    

* * *

### **Example (shown by `dig`)**

```shell
dig +short TXT _mailpubkey.alice.example.com "v=1;pk=BzowchszhjIBqUCDj4wrsysO6BKOJkJsG0-Lbox7u27J;alg=secp256k1;exp=2026-01-01;usr=alice@company.com;pv=mailnite;"
```

* * *



