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
## 5. Critical Doubts & Explanations
**Q1: "Is my password now perfectly secure because I used SHA-256?"** <br>
**Answer:** No. SHA-256 is a "fast" hash. Modern GPUs can calculate billions of SHA-256 hashes per second. If someone steals your NVS data, they can run a "Brute Force" attack (guessing millions of passwords per second) very quickly.<br>
- *Industry Standard:* For real-world passwords, you should use PBKDF2, Argon2, or bcrypt, which are "slow" hashes designed to be resistant to GPU brute-forcing. Your current implementation is fine for an HMI interface, but keep this in mind if you ever connect to the public internet.<br>
**Q2: "What happens if I call `mbedtls_sha256_starts` twice?"** <br>
**Answer:** It resets the internal state. You lose the previous progress. You must finish or free the context before starting a new hash. If you don't call free(), you will leak small amounts of memory, which will eventually crash your ESP32.<br>
**Q3: "Can I reverse the SHA-256 hex string to get the original password?"** <br>
**Answer:** No. That is the definition of a **One-Way Function**. You can prove a password is correct by hashing it and comparing the result to your stored hash, but you cannot "decrypt" the hash to retrieve the password.<br>
**Q4: "Why the 64 in s_jwt_secret[65]?"** <br>
**Answer:** Because 32 bytes of binary hash represented in Hex require 2 characters per byte ($32 \times 2 = 64$). Adding `+1` for the null terminator (`\0`) is mandatory in C to prevent buffer overflows.<br>
