---
name: android-security
description: Comprehensive Android app and system security best practices from official Android documentation. Use when implementing security features, reviewing security issues, handling sensitive data, or ensuring compliance with Android security guidelines.
---

# Android Security Best Practices

Official guidelines from Android Open Source Project (AOSP) and Android Developers documentation for building secure Android applications.

## Core Security Principles

1. **Minimize attack surface** - Request only necessary permissions, prefer intents over permissions
2. **Encrypt in transit** - Always use TLS/HTTPS for network communication
3. **Secure at rest** - Use internal storage for sensitive data, validate external storage
4. **Trust verification** - Implement signature permissions, custom trust managers, app choosers
5. **Keep updated** - Maintain current dependencies, libraries, and security providers
6. **User control** - Implement app choosers and biometric authentication for sensitive actions

---

## Security Topics

### [Communication Security](reference/communication-security.md)
Secure network communication, IPC, inter-app communication, and WebView security.

**Key Topics:**
- Network Security Configuration (HTTPS/TLS)
- Inter-app communication (Intents, Content Providers)
- WebView security best practices
- IPC mechanisms (Binder, Messenger, Services, Broadcast Receivers)

### [Data Storage](reference/data-storage.md)
Secure data storage strategies for sensitive information.

**Key Topics:**
- Internal storage (most secure)
- External storage validation
- SharedPreferences security
- Cache file management
- Input validation

### [Authentication & Credentials](reference/authentication.md)
User authentication, biometrics, and credential management.

**Key Topics:**
- Credential Manager integration
- Passkeys and biometric authentication
- Account Manager usage
- Credential exposure minimization

### [Permissions](reference/permissions.md)
Permission best practices and minimization strategies.

**Key Topics:**
- Minimal permission requests
- Using intents instead of permissions
- Signature-based permissions
- Permission-protected data sharing

### [Cryptography](reference/cryptography.md)
Cryptographic best practices and key management.

**Key Topics:**
- JCA security providers
- AES and Elliptic Curve algorithms
- Key generation and storage (Android Keystore)
- API key management
- Certificate pinning

### [Android Components Security](reference/components.md)
Security for Services, BroadcastReceivers, ContentProviders, and Activities.

**Key Topics:**
- Component export control
- Service security
- BroadcastReceiver permissions
- ContentProvider security
- Dynamic code loading risks

### [Common Vulnerabilities](reference/common-vulnerabilities.md)
Security risks to avoid and testing strategies.

**Key Topics:**
- Configuration issues (debuggable, exported)
- Network vulnerabilities
- Data security issues
- Code injection risks
- WebView vulnerabilities

---

## Quick Security Checklist

Before releasing your app:

- [ ] All network communication uses HTTPS with Network Security Configuration
- [ ] Sensitive data stored in internal storage only
- [ ] Permissions minimized to only necessary ones
- [ ] `android:exported` explicitly set for all components
- [ ] Content Providers properly secured with permissions
- [ ] Input validation implemented for all external data
- [ ] SQL injection prevented with parameterized queries
- [ ] Cryptography uses standard JCA providers (no custom implementations)
- [ ] API keys not committed to source control
- [ ] WebView JavaScript disabled or restricted to trusted content
- [ ] Dynamic code loading avoided or secured
- [ ] Biometric/credential authentication for sensitive operations
- [ ] All dependencies and libraries up to date
- [ ] `android:debuggable="false"` in production
- [ ] Vulnerability scanning completed
- [ ] Play Integrity API implemented

---

## Additional Resources

### App Integrity & Verification
- Play Integrity API - Verify app authenticity and device integrity
- Advanced Protection Mode - Enhanced security for sensitive apps
- Certificate Transparency - Compliance with Google's CT policy
- Developer Verification - Required starting September 2026 for select regions

### System Security (Device Manufacturers)
- SELinux - Defines and enforces Android's security model
- Root Processes - Minimize code running as root
- Security Patch Level (SPL) - Monthly update requirements

### Other Security Tools
- Safe Browsing - Avoid known threat URLs
- reCAPTCHA - Challenge malicious traffic
- Hardware-Backed Keys - TEE integration
- Direct Boot - Actions before device unlock

---

## Official Documentation Links

**Android Developers (developer.android.com)**
- [Security Best Practices](https://developer.android.com/privacy-and-security/security-best-practices)
- [Security Checklist](https://developer.android.com/privacy-and-security/security-tips)
- [Android Keystore System](https://developer.android.com/privacy-and-security/keystore)
- [Network Security Configuration](https://developer.android.com/privacy-and-security/security-config)
- [Cryptography Guide](https://developer.android.com/privacy-and-security/cryptography)

**Android Open Source Project (source.android.com)**
- [Security Best Practices](https://source.android.com/docs/security/best-practices)
- [App Security Best Practices](https://source.android.com/docs/security/best-practices/app)
- [System Security Best Practices](https://source.android.com/docs/security/best-practices/system)
- [Android Security Overview](https://source.android.com/docs/security)
- [Monthly Security Bulletins](https://source.android.com/docs/security/bulletin)

---

*This skill is based exclusively on official Android documentation from Android Open Source Project (AOSP) and Android Developers (developer.android.com), current as of April 2026.*
