Broken access control means that attackers gain access to user accounts and act as users or admins, and that regular users can gain unintended privileged functions. Strong access mechanisms ensure that each role has clear and isolated privileges.

### Mitigating Broken Access Control

- Deny access by default, except for public resources.
- Build strong access control mechanisms and reuse them across the application.
- Disable server directory listing and do not store sensitive data in root.
- Rate limit API and controller access.
- Validate JWT tokens after logout.

*Related reading*:
[[Access Control List (ACL)]]

Next up : [[A02 2021 Cryptographic Failures]]