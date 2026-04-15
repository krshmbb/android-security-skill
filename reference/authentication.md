Last updated: April 2026

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

**Recommendations**
- Integrate with Credential Manager for unified authentication
- Support passkeys as primary authentication method
- Add biometric authentication for sensitive operations
- Use Android's autofill framework

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

### Never Store Passwords or Raw Credentials on Device

**What NOT to Store**
- User passwords
- Raw authentication credentials
- Unencrypted tokens

**What to Store**
- Short-lived, service-specific authorization tokens
- Encrypted refresh tokens (with encryption key in Keystore)
- Session identifiers (with expiration)

---

## Account Manager

### When to Use

**Multi-App Service Access**
- Apps need to access shared account
- Multiple apps (typically from same developer) use the same account type
- Centralized authentication managed by an authenticator

### Security Practices

**Don't Store Passwords**
- Don't store passwords on device
- Use AccountManager for account management
- Use short-lived tokens instead of credentials

**Avoid passing credentials between components**
- Do not pass passwords or tokens via Intents or Bundles
- If necessary, use secure Parcelable implementation

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

---

## Credential Minimization

### Principles

**Avoid Unnecessary Credential Requests**
- Only request authentication when needed
- Use cached credentials when appropriate
- Implement remember-me functionality securely

---

## Official Documentation

- [Authentication](https://developer.android.com/privacy-and-security/security-tips#authentication)
- [Credential requests](https://developer.android.com/privacy-and-security/security-tips#credential-requests)
