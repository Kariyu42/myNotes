#security #cryptography #data #privacy
### Definition:

> Cryptographic failures, previously known as Sensitive Data Exposure, covers the protection of data in transit and at rest. This includes passwords, credit card numbers, health records, personal information and other sensitive information.

It is especially important for organizations covered by standards like PCI Data Security Standards (PCI DSS) or data privacy regulations like the EU general Data Protection Regulation (GDPR).

### Mitigating Cryptographic Failures

- Identify sensitive data and apply appropriate security controls.
- Don't store sensitive data unless you have to -> discard sensitive data, use tokenization or truncation.
- Encrypt all sensitive data at rest using strong encryption algorthms, protocols and keys.
- Encrypt data in transit using secure protocols like TLS and HTTPS HSTS.
- Disable caching for sensitive data.
- Store password using strong, salted hashing functions like Argon2, scrypt, and bcrypt.

Read more on:
[[Data Encryption]]

Next up: [[A03 2021 Injection]]