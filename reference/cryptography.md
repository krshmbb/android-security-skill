Last updated: April 2026

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

### Provider Specification Rules

**Android Keystore System**
- **MUST** specify the provider when using Android Keystore
- Example: `KeyGenerator.getInstance("AES", "AndroidKeyStore")`

**All Other Situations**
- **DO NOT** specify a provider to avoid compatibility problems
- Let the system select the best provider
- Example: `KeyGenerator.getInstance("AES")`

---

## Recommended Algorithms

Use these algorithms for cryptographic operations:

| Class | Recommended Algorithm |
|-------|----------------------|
| **Cipher** | AES in CBC or GCM mode with 256-bit keys (e.g., `AES/GCM/NoPadding`) |
| **MessageDigest** | SHA-2 family (e.g., `SHA-256`) |
| **Mac** | SHA-2 family HMAC (e.g., `HmacSHA256`) |
| **Signature** | SHA-2 family with ECDSA (e.g., `SHA256withECDSA`) |

---

## Common Cryptographic Operations

### Encrypt a Message

```kotlin
val plaintext: ByteArray = ...
val keygen = KeyGenerator.getInstance("AES")
keygen.init(256)
val key: SecretKey = keygen.generateKey()
val cipher = Cipher.getInstance("AES/CBC/PKCS5PADDING")
cipher.init(Cipher.ENCRYPT_MODE, key)
val ciphertext: ByteArray = cipher.doFinal(plaintext)
val iv: ByteArray = cipher.iv
```

**Important**: Store the IV with the ciphertext for decryption.

### Generate a Message Digest

```kotlin
val message: ByteArray = ...
val md = MessageDigest.getInstance("SHA-256")
val digest: ByteArray = md.digest(message)
```

### Generate a Digital Signature

```kotlin
val message: ByteArray = ...
val key: PrivateKey = ...
val s = Signature.getInstance("SHA256withECDSA")
    .apply {
        initSign(key)
        update(message)
    }
val signature: ByteArray = s.sign()
```

### Verify a Digital Signature

```kotlin
val message: ByteArray = ...
val signature: ByteArray = ...
val key: PublicKey = ...
val s = Signature.getInstance("SHA256withECDSA")
    .apply {
        initVerify(key)
        update(message)
    }
val valid: Boolean = s.verify(signature)
```

---

## Deprecated Functionality to Avoid

### Do Not Use

**Bouncy Castle Algorithms**
- Deprecated when explicitly requested
- Let the system choose the provider instead

**Crypto Provider**
- Removed as of Android 9 (API 28)
- Do not rely on this provider

**security-crypto Jetpack Library**
- All APIs deprecated in version 1.1.0+
- Migrate to direct Android Keystore usage

**PBE without Initialization Vector**
- Always pass explicit IV for password-based encryption
- Never rely on default IV generation

---

## Android Keystore System

### Overview

**Purpose**
- Secure long-term key storage and retrieval
- Hardware-backed encryption (when available)
- Keys never leave secure hardware
- Optional user authentication for key use (configure when needed)

**When to Require User Authentication**

Require user authentication (biometric/PIN/password) for keys that protect:
- Financial transactions or payments
- Sensitive personal data (health records, private messages)
- Access to critical account operations
- Data that requires proof of user presence

Do NOT require authentication for:
- Background operations that run without user interaction
- App-to-server communication keys
- Non-sensitive data encryption
- Keys that need automatic access on app startup

### When to Use: Keystore vs Keychain

**Android Keystore Provider**

Use when you need **app-specific credentials** that only your app should access:
- Credentials are private to your app
- No user selection UI required
- Simpler implementation for single-app scenarios
- Best for most app-specific encryption needs

**KeyChain API**

Use when you need **system-wide credentials** shared across apps:
- Multiple apps can access the same credentials with user consent
- Users select which credentials to share via system UI
- Required for enterprise scenarios (VPN, Wi-Fi authentication)
- Best for certificates and credentials that should be reusable

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

**When to Use**

Enable user authentication for keys protecting:
- Payment or financial data
- Sensitive user information
- Operations requiring explicit user consent
- Data that should only be accessible when user is present

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
    .setUserAuthenticationValidityDurationSeconds(30)  // Key valid for 30 seconds after auth
    .build()
)
```

**Authentication Validity Duration**
- **Short duration (5-30 seconds)**: For highly sensitive operations like payments
- **Longer duration (300+ seconds)**: For repeated operations in same session
- **Per-operation**: Set to `-1` to require authentication for each key use

**Benefits**
- Keys protected by device lock screen
- Automatic key invalidation on lock screen change
- Hardware-backed security
- Ensures user presence for sensitive operations

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

## Additional Resources

- [Android Cryptography](https://developer.android.com/privacy-and-security/cryptography)
- [Android Keystore System](https://developer.android.com/privacy-and-security/keystore)
- [API Key Management](https://developer.android.com/privacy-and-security/security-tips#api-keys)
