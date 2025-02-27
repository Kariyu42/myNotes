### Definition

> An injection vulnerability in a web application allows attackers to send hostile data to an interpreter, causing that data to be compiled and executed on the server. A common form of injection is [[SQL Injection]].

### Preventing Injection Attacks:

- Use a safe API which avoids the use of the interpreter entirely.
- Use positive or "whitelist" server-side input validation.
- Escape special characters.
- Use LIMIT and other SQL controls within queries to prevent mass disclosure of records in case of SQL injection.