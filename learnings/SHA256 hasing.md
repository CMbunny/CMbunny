# Mbed TLS / SHA 256
## 1. Why `mbedtls/sha256.h`?
In embedded systems, you cannot rely on external heavy libraries. Mbed TLS is designed specifically for resource-constrained devices. It provides:<br>
- ***Hardware Acceleration:*** On the ESP32, Mbed TLS functions are often "hooked" into the chip’s dedicated SHA hardware accelerator. This means the CPU isn't doing the math; a specialized circuit is, making it incredibly fast and power-efficient.<br>
- ***Memory Safety:*** It is written to be strictly controlled, avoiding dynamic memory allocation during sensitive cryptographic operations, which prevents heap fragmentation and security vulnerabilities.
## 2. How it works: The "Context-Update-Finish" Pattern
Cryptographic hashing works in chunks. If you have a 1GB file, you cannot load it all into RAM to hash it. You hash it in small pieces.<br>
- `mbedtls_sha256_init`: Prepares the object.,br>
- `mbedtls_sha256_starts(&ctx, 0)`: Initializes the algorithm. The 0 argument tells the engine to use SHA-256 (Standard). If it were 1, it would use SHA-224 (a truncated version).<br>
- `mbedtls_sha256_update`: This is the "streaming" part. You can call this 10 times with different parts of a password or data.<br>
- `mbedtls_sha256_finish`: Finalizes the padding and produces the 32-byte (256-bit) raw binary digest.
## 3. Why Hex Encoding?
SHA-256 outputs binary data (32 bytes). Binary data often contains "non-printable" characters (bytes like `0x00` or `0xFF` which can break C-string functions like `printf` or `strcpy` because they look like null terminators).<br>
***Hex encoding*** converts each byte into two human-readable ASCII characters (`0-9`, `a-f`).<br>
- Binary (32 bytes) $\rightarrow$ Hex String (64 characters + null terminator = 65 bytes).
- This makes the hash safe to store in a database or NVS as a simple text string.
## 4. Code Example: Isolated SHA-256
This is the "pure" version of what you see in your source file:
```
#include "mbedtls/sha256.h"
#include <stdio.h>

void hash_string(const char *input) {
    uint8_t hash[32];                // 32 bytes = 256 bits
    mbedtls_sha256_context ctx;

    mbedtls_sha256_init(&ctx);
    mbedtls_sha256_starts(&ctx, 0);      // 0 = SHA-256
    mbedtls_sha256_update(&ctx, (const uint8_t *)input, strlen(input));
    mbedtls_sha256_finish(&ctx, hash);
    mbedtls_sha256_free(&ctx);

    // Convert to hex
    char hex_out[65];
    for(int i = 0; i < 32; i++) {
        sprintf(&hex_out[i*2], "%02x", hash[i]);
    }
    printf("Hash: %s\n", hex_out);
}
```
## 5.Critical Doubts & Explanations
**Q1:"Is my password now perfectly secure because I used SHA-256?"** <br>
**Answer:** No. SHA-256 is a "fast" hash. Modern GPUs can calculate billions of SHA-256 hashes per second. If someone steals your NVS data, they can run a "Brute Force" attack (guessing millions of passwords per second) very quickly.
- *Industry Standard:* For real-world passwords, you should use PBKDF2, Argon2, or bcrypt, which are "slow" hashes designed to be resistant to GPU brute-forcing. Your current implementation is fine for an HMI interface, but keep this in mind if you ever connect to the public internet.<br>
**Q2:"What happens if I call `mbedtls_sha256_starts` twice?"** <br>
**Answer:** It resets the internal state. You lose the previous progress. You must finish or free the context before starting a new hash. If you don't call free(), you will leak small amounts of memory, which will eventually crash your ESP32.<br>
**Q3: "Can I reverse the SHA-256 hex string to get the original password?"** <br>
**Answer:** No. That is the definition of a **One-Way Function**. You can prove a password is correct by hashing it and comparing the result to your stored hash, but you cannot "decrypt" the hash to retrieve the password.<br>
**Q4: "Why the 64 in s_jwt_secret[65]?"** <br>
**Answer:** Because 32 bytes of binary hash represented in Hex require 2 characters per byte ($32 \times 2 = 64$). Adding `+1` for the null terminator (`\0`) is mandatory in C to prevent buffer overflows.<br>

#### NOTE:
It is important to clarify a major security concept first: Hashing (like SHA-256) is NOT Encryption.
You cannot decode a SHA-256 hash. It is a "one-way street." You cannot take the hash and turn it back into the original data.<br>

**1. How do we know the data is "Correct" without decoding?** <br>
Since we can't "decode" the hash, we use a Verification Pattern:<br>
**i)The Store:** You calculate the hash of the password (e.g., "Bunny@123" becomes a3f...) and save that hash in your NVS.<br>
**ii)The Check:** When a user logs in, they type their password ("Bunny@123").<br>
**iii)The Re-Hash:** You hash the typed password immediately.<br>
**iv)The Compare:** You compare the new hash to the stored hash.
- If they match (a3f... == a3f...), the password was correct.
- If they don't, the password was wrong.<br>

**Does it have a CRC?** <br>
No. A CRC (Cyclic Redundancy Check) is for error detection (like a noisy cable). SHA-256 is for integrity and authentication. If even one bit of the password changes, the hash changes completely. This is called the "Avalanche Effect."<br>

**2. What changes if you use STM32?** <br>
The logic remains 100% identical because Mbed TLS is hardware-agnostic C code.<br>
- ***The Difference:*** On an ESP32, the mbedtls library is already optimized for the ESP32’s hardware. On an STM32, you will likely include the Mbed TLS library via the STM32CubeIDE (or import the source files).
- ***Performance:*** High-end STM32 chips (like the STM32H7 series) have "Cryptographic Acceleration" units just like the ESP32. You would need to configure the library to use those hardware-specific registers to keep your speeds up. <br>

**3. Can you use this on protocols other than Modbus?** <br>
Yes, absolutely. You can use SHA-256 with any protocol (MQTT, HTTP, raw TCP, UART).<br>
- ***Example:*** You could hash an entire configuration file, send the file + the hash to the device, and the device could re-hash the file to ensure it wasn't corrupted during the transfer (this is called a Checksum/Fingerprint).<br>

**4. What we didn't touch (The "Missing" Pieces)** <br>
***A. Salting*** <br>
Currently, the code hashes the password directly. If two users have the same password, they have the same hash. A hacker can spot this pattern.<br>
***The fix:*** A "Salt" is a random string added to the password before hashing.
- `Hash(Password + RandomSalt)`
- This makes the hash unique, even if the password is simple.<br>

***B. The JWT Signature "Secret"*** <br>
In the `auth_generate_jwt` function, you include `s_jwt_secret`. This is the "Key" to the kingdom. If someone learns this secret, they can forge their own JWTs and impersonate any user.<br>

**Critical Note:** The code generates this secret at the first boot and saves it in NVS. This is excellent security practice, as the secret is unique to that specific hardware device.

**Question:** What is the Salt Concept?<br>
**Answer:** A salt is a unique string added to the password to make the hash unpredictable. However, it is not added to the hex result. It is added to the raw password before the hashing process begins.<br>

**The correct flow:** <br>

1.)**Input:** User enters Password123 <br>
2.)**Combine:** You take a unique, random string (the Salt) and join it with the password: Password123 + a7b2... <br>
3.)**Hash:** You run the SHA-256 algorithm on the combined string.<br>
4.)**Result:** You get a hash that is unique to that specific user, even if two users choose the same password. <br>

**Question:** Why it's not added to the Hex string? <br>
If you add the salt to the resulting hex string (after the hash is finished), you are just appending data to the output. That doesn't make the underlying hash any harder to crack; it just makes the final string longer. By adding the salt before the hash, you force an attacker to crack each user's hash individually, one by one, rather than using a "Rainbow Table" (a pre-computed list of common password hashes) to crack everyone at once. <br>
|Action|Why?|
|--|--|
|Adding salt to password|Prevents "Rainbow Table" attacks (pre-computed lists).|
|Hashing the combination|Creates a unique fingerprint for that specific user.|
|Storing Salt + Hash|You must save the salt in NVS alongside the hash so you can re-create the same hash during login.|

### NOTES:
**Mbed TLS SHA-256** <br>
Think of SHA-256 as a "digital shredder" that turns any amount of data into a unique, fixed-size fingerprint. Once shredded, you can never put the data back together.
- **It’s a One-Way Street:** You can turn a password into a hash, but you cannot turn a hash back into a password. This is why we store hashes, not passwords.
- **The "Avalanche Effect":** If you change even one tiny character in the password (e.g., from admin to Admin), the resulting hash will look completely different.
- **Not Encryption:** People often confuse the two. Encryption is like a locked box you can open with a key; Hashing is like a fingerprint that proves you are who you say you are.
- **The Pattern (Context-Update-Finish):** <br>
i)***Init:*** Prepare the "shredder." <br>
ii)***Starts:*** Choose the specific mode (SHA-256).<br>
iii)***Update:*** Feed the password (or file) into the shredder in chunks.<br>
iv)***Finish:*** Collect the final 32-byte digest.<br>
v)***Free:*** Clean up the memory so your device doesn't crash.<br>
- **Why Hex Encoding?** The raw output of SHA-256 is "binary" (1s and 0s that look like junk to a computer). Hex encoding turns that junk into simple, readable text characters (0-9, a-f) so they are easy to save in a file or database.
- **Salting (The Security Booster):** * Never hash a "naked" password.<br>
i)Always add a random "Salt" string to the password before hashing.<br>
ii)This ensures that even if two people have the same password, their hashes will be different, making it much harder for hackers to crack the system.<br>
- **Verification:** To check if a password is correct, you don't "decode" the hash. You hash the typed password, and see if it matches the saved hash exactly. If they match, the password is correct.<br>

*This process is the gold standard for verifying identities in embedded systems because it is fast, memory-efficient, and—when used with a salt—extremely secure.*
