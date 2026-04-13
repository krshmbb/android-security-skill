# Cryptography Best Practices

Guidelines for implementing secure cryptography in Android applications.

---

## Core Principles

### Use Standard Implementations

**Fundamental Rules**
- Use Java Cryptography Architecture (JCA) security providers
- Use highest-level framework implementations available
- **NEVER implement custom cryptographic algorithms**
- Prefer `HttpsURLConnection` or `SSLSocket` for secure communications

**Why**
- Custom crypto implementations are error-prone
- Standard implementations are thoroughly tested
- Framework implementations receive security updates

---

## Encryption Algorithms

### AES (Advanced Encryption Standard)

**Key Sizes**
```kotlin
// Recommended - 256-bit for commercial applications
val keySize = 256

// Minimum acceptable - 128-bit
val minKeySize = 128

// Generate AES key
val keyGenerator = KeyGenerator.getInstance("AES")
keyGenerator.init(keySize, SecureRandom())
val secretKey = keyGenerator.generateKey()
```

**Best Practices**
- Use 256-bit keys for commercial applications
- 128-bit minimum for less sensitive data
- Never use keys smaller than 128-bit

### Elliptic Curve Cryptography

**Key Sizes**
```kotlin
// Recommended key sizes
val keySize = 256  // or 224

// Generate EC key pair
val keyPairGenerator = KeyPairGenerator.getInstance("EC")
keyPairGenerator.initialize(keySize, SecureRandom())
val keyPair = keyPairGenerator.generateKeyPair()
```

**Requirements**
- Use 224-bit or 256-bit public key sizes
- Provides strong security with smaller key sizes than RSA

---

## Block Cipher Modes

### Understanding Block Modes

**Available Modes**
- **CBC** (Cipher Block Chaining)
- **CTR** (Counter Mode)
- **GCM** (Galois/Counter Mode) - Recommended

**When to Use Each**
```kotlin
// GCM - Best choice (provides encryption + integrity)
val cipher = Cipher.getInstance("AES/GCM/NoPadding")

// CTR - When you need streaming encryption
val cipherCTR = Cipher.getInstance("AES/CTR/NoPadding")

// CBC - Legacy support
val cipherCBC = Cipher.getInstance("AES/CBC/PKCS5Padding")
```

### Initialization Vector (IV)

**Critical Requirements**

**For CTR Mode**
```kotlin
// NEVER reuse IV/Counter in CTR mode
// Use cryptographically random IV
val iv = ByteArray(16)
SecureRandom().nextBytes(iv)

val cipher = Cipher.getInstance("AES/CTR/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey, IvParameterSpec(iv))
```

**For CBC Mode**
```kotlin
// Use random IV for each encryption
val iv = ByteArray(16)
SecureRandom().nextBytes(iv)

val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey, IvParameterSpec(iv))
```

**For GCM Mode**
```kotlin
// GCM uses a nonce - must be unique for each encryption
val nonce = ByteArray(12)  // 12 bytes for GCM
SecureRandom().nextBytes(nonce)

val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey, GCMParameterSpec(128, nonce))
```

**Rules**
- Never reuse IV/nonce with same key
- Always use cryptographically random IV
- Store IV with ciphertext (IV doesn't need to be secret)

---

## Integrity Protection

### Why Integrity Matters

**Without Integrity**
- Attacker can modify encrypted data
- May lead to plaintext recovery attacks
- Can't detect tampering

### Implementation Options

**Option 1: GCM Mode (Recommended)**
```kotlin
// GCM provides both encryption and integrity
val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey, GCMParameterSpec(128, nonce))

// Encrypt data
val ciphertext = cipher.doFinal(plaintext)
// GCM tag is appended to ciphertext automatically
```

**Option 2: Encrypt-then-MAC**
```kotlin
// 1. Encrypt data
val cipher = Cipher.getInstance("AES/CTR/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey, IvParameterSpec(iv))
val ciphertext = cipher.doFinal(plaintext)

// 2. Calculate MAC
val mac = Mac.getInstance("HmacSHA256")
mac.init(macKey)
val tag = mac.doFinal(ciphertext)

// 3. Store: iv || ciphertext || tag
```

### HMAC Algorithms

**Supported Algorithms**
```kotlin
// Strongest - Use for new implementations
val mac = Mac.getInstance("HmacSHA512")

// Good - Widely supported
val mac256 = Mac.getInstance("HmacSHA256")

// Acceptable - Minimum recommendation
val mac1 = Mac.getInstance("HmacSHA1")
```

**Best Practice**
- Use HMAC-SHA256 or stronger
- Never use HMAC-MD5 or non-cryptographic hashes

---

## Key Generation

### Secure Random Number Generation

**Always Use SecureRandom**
```kotlin
// Good - Cryptographically secure
val secureRandom = SecureRandom()
val key = ByteArray(32)  // 256-bit key
secureRandom.nextBytes(key)

// Bad - NEVER use regular Random for crypto
// val random = Random()
// random.nextBytes(key)  // INSECURE!
```

**Key Derivation**
```kotlin
// Use proper key derivation function
val keySpec = PBEKeySpec(
    password.toCharArray(),
    salt,
    iterations = 100000,  // High iteration count
    keyLength = 256
)
val keyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256")
val key = keyFactory.generateSecret(keySpec)
```

### Important Rules

**Never Use Non-Random Keys**
- Don't derive keys from passwords without proper KDF
- Don't use hardcoded keys in source code
- Don't generate keys from predictable sources

**Weakens Cryptography**
```kotlin
// Bad - Predictable key
// val key = "my_secret_key".toByteArray()

// Bad - Weak derivation
// val key = password.hashCode().toString().toByteArray()

// Good - Proper random generation
val keyGenerator = KeyGenerator.getInstance("AES")
keyGenerator.init(256, SecureRandom())
val key = keyGenerator.generateKey()
```

---

## Android Keystore System

### Overview

**Purpose**
- Secure long-term key storage and retrieval
- Hardware-backed encryption (when available)
- Keys never leave secure hardware
- User authentication required for key use

### Basic Usage

**Generate Key in Keystore**
```kotlin
val keyGenerator = KeyGenerator.getInstance(
    KeyProperties.KEY_ALGORITHM_AES,
    "AndroidKeyStore"
)

keyGenerator.init(
    KeyGenParameterSpec.Builder(
        "my_key_alias",
        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
    )
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .setKeySize(256)
    .build()
)

val secretKey = keyGenerator.generateKey()
```

**Retrieve Key from Keystore**
```kotlin
val keyStore = KeyStore.getInstance("AndroidKeyStore")
keyStore.load(null)

val secretKey = keyStore.getKey("my_key_alias", null) as SecretKey
```

### User Authentication Requirement

**Require Biometric/PIN for Key Use**
```kotlin
keyGenerator.init(
    KeyGenParameterSpec.Builder(
        "authenticated_key",
        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
    )
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .setUserAuthenticationRequired(true)
    .setUserAuthenticationValidityDurationSeconds(30)
    .build()
)
```

**Benefits**
- Keys protected by device lock screen
- Automatic key invalidation on lock screen change
- Hardware-backed security

---

## API Key Management

### Storage

**Never Hardcode Keys**
```kotlin
// Bad - Keys in source code
// const val API_KEY = "sk_live_abc123..."

// Bad - Keys in strings.xml
// <string name="api_key">sk_live_abc123...</string>

// Good - Use Android Keystore
// Store encrypted in Keystore
```

**Use Build Configuration**
```gradle
// build.gradle
android {
    defaultConfig {
        // Load from environment or local.properties
        buildConfigField "String", "API_KEY",
            "\"${project.findProperty("API_KEY")}\""
    }
}

// local.properties (gitignored)
// API_KEY=your_key_here
```

**Use Secrets Gradle Plugin**
```gradle
// build.gradle (project level)
plugins {
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin'
}

// Reads from local.properties automatically
// Keys never committed to source control
```

### Encryption for API Keys

**Encrypt with Tink**
```kotlin
// Using Tink library for API key encryption
val aead = AndroidKeysetManager.Builder()
    .withSharedPref(context, "api_keys", "master_key")
    .withKeyTemplate(KeyTemplates.get("AES256_GCM"))
    .withMasterKeyUri("android-keystore://master_key")
    .build()
    .keysetHandle
    .getPrimitive(Aead::class.java)

// Encrypt API key
val encrypted = aead.encrypt(
    apiKey.toByteArray(),
    null  // No additional data
)

// Decrypt when needed
val decrypted = aead.decrypt(encrypted, null)
val apiKey = String(decrypted)
```

### Environment-Specific Keys

**Separate Keys by Build Type**
```gradle
android {
    buildTypes {
        debug {
            buildConfigField "String", "API_KEY",
                "\"${project.findProperty("DEV_API_KEY")}\""
        }
        release {
            buildConfigField "String", "API_KEY",
                "\"${project.findProperty("PROD_API_KEY")}\""
        }
    }
}
```

---

## Access Control for API Keys

### Application Restrictions

**Limit by Package Name**
- Configure API keys to work only with your app's package name
- Restrict by app signing certificate
- Prevents key theft and reuse

**Platform-Specific Restrictions**
- Google Maps API: Restrict by package name and SHA-1
- Firebase: Automatic package name restrictions
- Custom APIs: Implement server-side package verification

### IP Restrictions

**When Possible**
- Implement IP allowlists for API calls
- More applicable to server-side APIs
- Limited usefulness for mobile clients (dynamic IPs)

### Monitoring

**Track API Usage**
```kotlin
// Log API calls for suspicious activity
fun logApiCall(endpoint: String, success: Boolean) {
    analytics.logEvent("api_call") {
        param("endpoint", endpoint)
        param("success", success)
        param("timestamp", System.currentTimeMillis())
    }
}
```

**Monitor for Abuse**
- Unusual usage patterns
- Requests from unexpected locations
- High volume from single device

---

## Key Rotation

### Why Rotate Keys

**Security Best Practices**
- Limit exposure if key is compromised
- Compliance requirements (ISO 27001: 90 days to 6 months)
- Reduce cryptanalysis opportunities

### Implementation

**Support Multiple Keys**
```kotlin
class KeyManager {
    private val currentKeyVersion = 2

    fun encrypt(data: ByteArray): EncryptedData {
        val key = getKey(currentKeyVersion)
        val ciphertext = performEncryption(data, key)
        return EncryptedData(ciphertext, currentKeyVersion)
    }

    fun decrypt(encryptedData: EncryptedData): ByteArray {
        val key = getKey(encryptedData.keyVersion)
        return performDecryption(encryptedData.ciphertext, key)
    }

    private fun getKey(version: Int): SecretKey {
        return when (version) {
            1 -> loadKey("key_v1")
            2 -> loadKey("key_v2")
            else -> throw IllegalArgumentException("Unknown key version")
        }
    }
}

data class EncryptedData(
    val ciphertext: ByteArray,
    val keyVersion: Int
)
```

**Re-encryption Strategy**
```kotlin
// Gradually re-encrypt data with new key
fun rotateData(oldData: EncryptedData): EncryptedData {
    // Decrypt with old key
    val plaintext = decrypt(oldData)

    // Encrypt with new key
    return encrypt(plaintext)
}
```

---

## Certificate Pinning

### Purpose

**Prevent MITM Attacks**
- Validate server certificates
- Ensure connection to legitimate server
- Additional layer beyond standard TLS

### Implementation

**Network Security Configuration**
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-01-01">
            <pin digest="SHA-256">base64-encoded-pin-1</pin>
            <pin digest="SHA-256">base64-encoded-pin-2</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

**Generate Certificate Pin**
```bash
# Get certificate from server
openssl s_client -connect api.example.com:443 | \
    openssl x509 -pubkey -noout | \
    openssl pkey -pubin -outform der | \
    openssl dgst -sha256 -binary | \
    base64
```

**Best Practices**
- Pin backup certificates
- Set expiration dates
- Monitor certificate rotation
- Have update mechanism for pin changes

---

## SSL/TLS Best Practices

### Always Use HTTPS

**Configuration**
```xml
<!-- Network Security Config -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

### Custom Trust Managers

**When to Use**
- Custom Certificate Authorities
- Self-signed certificates (development only)
- Additional certificate validation

**Implementation**
```kotlin
// Custom trust manager for additional validation
class CustomTrustManager : X509TrustManager {
    private val systemTrustManager = // System trust manager

    override fun checkServerTrusted(
        chain: Array<X509Certificate>,
        authType: String
    ) {
        // First validate with system trust manager
        systemTrustManager.checkServerTrusted(chain, authType)

        // Additional custom validation
        validateAdditionalConstraints(chain)
    }

    // Implement other required methods
}
```

---

## Cryptography Checklist

Before releasing your app:

- [ ] JCA security providers used (no custom crypto)
- [ ] AES with 256-bit keys (or 128-bit minimum)
- [ ] GCM mode used for encryption + integrity
- [ ] Cryptographically random IVs/nonces
- [ ] HMAC-SHA256 or stronger for integrity
- [ ] SecureRandom used for key generation
- [ ] Android Keystore used for long-term keys
- [ ] No hardcoded encryption keys
- [ ] API keys not in source control
- [ ] Secrets Gradle Plugin configured
- [ ] Environment-specific API keys used
- [ ] Certificate pinning implemented
- [ ] HTTPS used for all network communication
- [ ] Key rotation strategy implemented
- [ ] Keys require user authentication (when appropriate)

---

*Based on official Android documentation from developer.android.com and source.android.com*
