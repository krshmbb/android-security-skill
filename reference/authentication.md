# Authentication & Credentials

Guidelines for user authentication, biometrics, and credential management in Android applications.

---

## Credential Manager

### Overview

**Benefits**
- Unified authentication interface
- Integrates with passkeys, passwords, and federated sign-in
- Reduces user friction
- Modern authentication standard

**When to Use**
- User authentication
- Credential storage and retrieval
- Cross-device credential sync

### Integration

**Basic Setup**
```kotlin
// Integrate with Credential Manager for unified authentication
// Support passkeys as primary authentication method
// Add biometric authentication for sensitive operations
// Use Android's autofill framework
```

---

## Passkeys

### Overview

**Characteristics**
- Phishing-resistant authentication
- Cryptographic credentials
- Synced across devices
- No passwords to remember

**Best Practices**
- Implement passkeys as primary authentication method
- Provide fallback authentication options
- Support cross-device passkey usage

### Implementation

**Recommended Approach**
- Use Credential Manager API for passkey integration
- Follow WebAuthn standards
- Support platform authenticators
- Enable biometric unlock for passkeys

---

## Biometric Authentication

### When to Use

**Sensitive Operations**
- Accessing financial information
- Viewing personal health data
- Making payments
- Changing security settings
- Deleting important data

**Implementation Considerations**
- Add biometric authentication as additional security layer
- Support fingerprint and facial recognition
- Provide fallback to device credentials (PIN/password/pattern)

### Best Practices

**User Experience**
- Request biometric authentication before sensitive operations
- Provide clear explanation of why authentication is needed
- Support device credential fallback
- Handle authentication failures gracefully

**Security**
- Don't rely solely on biometrics for critical security decisions
- Combine with other authentication factors when appropriate
- Re-authenticate after timeout periods

---

## Credential Storage

### Never Store Credentials on Device

**What NOT to Store**
- User passwords
- Raw authentication credentials
- Unencrypted tokens

**What to Store**
- Short-lived, service-specific authorization tokens
- Encrypted refresh tokens (in Keystore)
- Session identifiers (with expiration)

### Secure Token Management

**Token Characteristics**
```kotlin
// Use short-lived tokens
val TOKEN_EXPIRATION = 3600 // 1 hour in seconds

// Limit permission scope
val REQUIRED_SCOPES = setOf("read:user", "write:data")

// Use secure storage
// Store in Android Keystore, not SharedPreferences
```

**Token Storage**
```kotlin
// Good - Use Keystore for sensitive tokens
// Store encrypted in Android Keystore

// Bad - Never store in plain text
// val sharedPref = getSharedPreferences("tokens", MODE_PRIVATE)
// sharedPref.edit().putString("token", authToken).apply()
```

---

## Authentication Rate Limiting

### Prevent Brute-Force Attacks

**Implementation Strategies**
```kotlin
class AuthenticationManager {
    private var failedAttempts = 0
    private var lockoutUntil: Long = 0

    fun authenticate(credentials: Credentials): AuthResult {
        // Check if locked out
        if (System.currentTimeMillis() < lockoutUntil) {
            return AuthResult.LockedOut(lockoutUntil)
        }

        // Attempt authentication
        val result = performAuthentication(credentials)

        if (result.success) {
            failedAttempts = 0
        } else {
            failedAttempts++
            if (failedAttempts >= MAX_ATTEMPTS) {
                lockoutUntil = System.currentTimeMillis() + LOCKOUT_DURATION
            }
        }

        return result
    }

    companion object {
        const val MAX_ATTEMPTS = 5
        const val LOCKOUT_DURATION = 30_000L // 30 seconds
    }
}
```

**Best Practices**
- Limit authentication attempts
- Implement exponential backoff
- Consider temporary lockouts
- Log suspicious authentication patterns

---

## Account Manager

### When to Use

**Multi-App Service Access**
- Apps need to access shared account
- Multiple apps from same developer
- Centralized authentication

### Security Practices

**Don't Store Passwords**
```kotlin
// Use AccountManager for account management
// Don't store passwords on device
// Use tokens instead
```

**Verify Calling App**
```kotlin
// Use checkSignatures() to verify calling app
fun verifyCallingApp(callerUid: Int): Boolean {
    val pm = packageManager
    val callerPackages = pm.getPackagesForUid(callerUid)

    callerPackages?.forEach { packageName ->
        if (pm.checkSignatures(
            context.packageName,
            packageName
        ) == PackageManager.SIGNATURE_MATCH) {
            return true
        }
    }
    return false
}
```

**Use CREATOR Before Passing Credentials**
- Validate account authenticity
- Verify account ownership
- Check account type

---

## Keystore for Credentials

### Single-App Credentials

**When to Use**
- Credentials used only by your app
- Long-term key storage
- Encryption keys

**Implementation**
```kotlin
// Use Android Keystore for secure credential storage
val keyStore = KeyStore.getInstance("AndroidKeyStore")
keyStore.load(null)

// Generate key
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
    .setUserAuthenticationRequired(true)
    .setUserAuthenticationValidityDurationSeconds(30)
    .build()
)
val key = keyGenerator.generateKey()
```

---

## Federated Sign-In

### Overview

**Benefits**
- Reduces password fatigue
- Delegates authentication to trusted providers
- Improves security
- Simplifies user experience

**Supported Providers**
- Sign-in with Google
- Facebook Login
- Apple Sign-In
- Other OAuth 2.0 providers

### Best Practices

**Security**
- Validate ID tokens from providers
- Verify token signatures
- Check token expiration
- Validate audience (client ID)

**User Experience**
- Provide multiple sign-in options
- Support account linking
- Handle sign-in errors gracefully
- Allow account disconnection

---

## Autofill Framework

### Benefits

**User Experience**
- Reduces user friction
- Improves form completion speed
- Secure credential entry

**Security**
- Integrates with Credential Manager
- Supports password managers
- Encrypted credential storage

### Implementation

**Support Autofill**
```kotlin
// Mark views for autofill
emailEditText.setAutofillHints(View.AUTOFILL_HINT_EMAIL_ADDRESS)
passwordEditText.setAutofillHints(View.AUTOFILL_HINT_PASSWORD)

// Set input types
emailEditText.inputType = InputType.TYPE_TEXT_VARIATION_EMAIL_ADDRESS
passwordEditText.inputType = InputType.TYPE_TEXT_VARIATION_PASSWORD
```

---

## Credential Minimization

### Principles

**Avoid Unnecessary Credential Requests**
- Only request authentication when needed
- Use cached credentials when appropriate
- Implement remember-me functionality securely

**Limit Credential Exposure**
```kotlin
// Good - Request credentials only when needed
fun accessSensitiveData() {
    if (!isAuthenticatedRecently()) {
        requestAuthentication()
    } else {
        // Proceed with authenticated action
    }
}

// Track authentication timestamp
private var lastAuthenticationTime: Long = 0
private fun isAuthenticatedRecently(): Boolean {
    val now = System.currentTimeMillis()
    return (now - lastAuthenticationTime) < AUTH_VALIDITY_DURATION
}
```

---

## Session Management

### Secure Session Handling

**Session Expiration**
```kotlin
class SessionManager {
    private var sessionToken: String? = null
    private var sessionExpiration: Long = 0

    fun createSession(token: String, expirationSeconds: Int) {
        sessionToken = token
        sessionExpiration = System.currentTimeMillis() +
                           (expirationSeconds * 1000)
    }

    fun isSessionValid(): Boolean {
        return sessionToken != null &&
               System.currentTimeMillis() < sessionExpiration
    }

    fun invalidateSession() {
        sessionToken = null
        sessionExpiration = 0
    }
}
```

**Best Practices**
- Set reasonable session timeouts
- Invalidate sessions on logout
- Refresh tokens before expiration
- Clear session data on app exit

---

## Multi-Factor Authentication (MFA)

### When to Use

**High-Security Scenarios**
- Financial transactions
- Healthcare data access
- Administrative actions
- Password changes
- Account recovery

### Implementation Options

**First Factor**
- Passkeys
- Password
- Biometric

**Second Factor**
- SMS OTP (less secure, but widely supported)
- Authenticator app TOTP
- Hardware security keys
- Biometric verification

**Best Practices**
- Support multiple MFA methods
- Provide backup authentication methods
- Store recovery codes securely
- Allow users to manage MFA settings

---

## Authentication Testing

### Security Testing

**Verify Authentication**
- Test credential validation
- Verify rate limiting
- Test session expiration
- Validate token handling
- Check biometric integration

**Common Vulnerabilities to Test**
- Brute force attacks
- Session fixation
- Credential stuffing
- Token theft
- Replay attacks

---

## Authentication Checklist

Before releasing your app:

- [ ] Credential Manager integrated
- [ ] Passkeys implemented as primary authentication
- [ ] Biometric authentication for sensitive operations
- [ ] No passwords stored on device
- [ ] Short-lived tokens used
- [ ] Rate limiting implemented
- [ ] Session management secure
- [ ] Autofill framework supported
- [ ] Federated sign-in options provided
- [ ] MFA available for high-security scenarios
- [ ] Keystore used for credential storage
- [ ] Authentication flows tested

---

*Based on official Android documentation from developer.android.com and source.android.com*
