# JSON WEB TOKEN
A JSON Web Token (JWT) is essentially a "digital ID card" that is compact, self-contained, and—most importantly—tamper-proof. It allows you to prove your identity to a server without having to send your password every time.

## 1. The Anatomy of a JWT
A JWT is just a single string divided into three parts by dots (.):
Header.Payload.Signature
- **Header:** Metadata. It tells the server what type of token it is (JWT) and which algorithm was used to sign it (e.g., HS256).
- **Payload:** The "Data." This is where you put the user’s username, their role (Admin/Client), and the expiration time (exp).
- **Signature:** The "Seal." This is the cryptographic proof that the token was created by your server and that no one has modified the data inside.

## 2. Base64URL Encoding: Why not just Base64?
You already have the alphabet in your code, but here is the ***why***:
Standard Base64 uses characters like +, /, and = which are unsafe in URLs and HTTP headers (they often get misinterpreted by web servers).<br>
**Base64URL** is a variation that:
- Replaces `+` with `-`
- Replaces `/` with `_`
- Removes the `=` padding.
  
This ensures the token can be pasted directly into a URL or an Authorization header without breaking anything.

## 3. HMAC-SHA256 vs. Your Current Approach
You are currently using a method of signing that is effectively a custom implementation.
- **Your Current Approach:** You are manually hashing `Header + "." + Payload + "." + Secret.` This is technically a "MAC" (Message Authentication Code), but it's a "home-rolled" version.
- **HMAC-SHA256:** This is the industry standard. It stands for Hash-based Message Authentication Code.<br>
i) It is not just hashing. It uses a specific, mathematically vetted process to combine your secret key and the data.<br>
ii)**Why switch?** Because standard libraries (like Mbed TLS) have highly optimized, secure implementations of HMAC. Using the standard ensures you aren't accidentally creating a vulnerability by how you concatenate your strings (e.g., avoiding "Length Extension Attacks").

## 4. Token Expiry Validation
Since JWTs are "stateless" (the server doesn't keep a list of them), the server must check if the token is still valid every single time.

***The Workflow:*** <br>
1.)**Read the `exp` claim:** Inside the payload, you have a timestamp (e.g., `1719430000`).<br>
2.)**Compare:** if (current_time > exp_timestamp) { REJECT_TOKEN; } <br>
3.)**Clock Skew:** Always allow a small margin (30–60 seconds) for server time differences. If your server is at 10:00:05 and the token expired at 10:00:00, it's safer to let it pass than to lock out a legitimate user.

## 5. Common Doubt Questions
**Question:** Can a hacker read my payload?<br>
**Answer:** YES. Base64URL encoding is not encryption. It is just a format. Anyone who intercepts the token can decode the middle part (Payload) and read the username or role. Never put sensitive data (like passwords or private keys) in the payload.<br>
**Question:** If I store my secret key in the code, is it safe? <br>
**Answer:** No. If someone gets your firmware, they get your secret. In an ESP32, the best practice is to generate a unique random secret at first boot and store it in NVS (as you are already doing) so that each individual device has its own unique key.<br>
**Question:** How do I "log out" a user if the token is valid until it expires?<br>
**Answer:** You can't, at least not easily. Because JWTs are self-contained, the server doesn't "know" you've logged out.
- ***The Professional Fix:*** Use a short-lived access token (e.g., 15 minutes) and a long-lived refresh token (stored securely in a database). When the access token expires, the client uses the refresh token to get a new one. To "log out," you simply delete the refresh token from your database.

### Comparison: Manual Hashing vs. HMAC-SHA256
|Feature|Manual Concatenation|HMAC-SHA256 (Standard)|
|--|--|--|
|**Logic**|`Hash(Header + "." + Payload + "." + Secret)`|`Hash( (Secret XOR opad) + Hash( (Secret XOR ipad) + Message ) )`|
|**Security**|Susceptible to "Length Extension Attacks"|Mathematically immune to most known hash attacks|
|**Standardization**|Home-rolled/Proprietary|RFC 2104 compliant (Industry Standard)|
|**Performance**|Slightly faster (fewer operations)|Slightly slower (due to double-hashing)|
|**Complexity**|Easy to implement|Requires a standardized library|

**Why the difference matters:** <br>
1.)**Length Extension Attacks:** In your manual approach, if an attacker knows the secret length, they can potentially append data to your signature and calculate a valid hash without knowing the secret. HMAC prevents this by using a specific "inner" and "outer" hash structure (the XOR logic mentioned in the table), which completely obscures the original secret.<br>
2.)**Compatibility:** If you ever want to move your HMI dashboard from your ESP32 to a web browser or a mobile app, those platforms will have built-in libraries for HMAC-SHA256. They will not have a library for your "Manual Concatenation" method, meaning you would have to write custom code for every single platform.<br>
### NOTES
- **Header:** `{"alg":"HS256","typ":"JWT"}` (Encoded)
- **Payload:** `{"sub":"admin","exp":1719430000}` (Encoded)
- **Signature:** `HMAC_SHA256(encoded_header + "." + encoded_payload, secret)`
- **Integrity:** The signature ensures the data hasn't been changed.
- **Security:** Never store secrets in the payload, and always use HMAC-SHA256 over custom concatenation.
- **The Secret is Key:** In both cases, the Secret is the glue. Never share it.
- **Don't Roll Your Own:** In cryptography, "rolling your own" (inventing your own method of signing data) is considered a dangerous practice. Always use the HMAC standard when you are ready to move to a production-grade system.
- **The "Two-Step" Hash:** HMAC-SHA256 is basically the process of hashing the message, then hashing the secret, and combining them in a way that makes it impossible to "strip" the secret back out or extend the message.
