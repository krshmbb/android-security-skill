---
name: android-security-skill

description: Comprehensive Android app and system security best practices from official Android documentation. Use when implementing security features, reviewing security issues, handling sensitive data, or ensuring compliance with Android security guidelines.
---

## Disclaimer

This AI skill is provided "as is", without warranty of any kind, express or implied.

The generated code may contain errors, security vulnerabilities, or unintended behavior. 
You are solely responsible for reviewing, testing, and validating any output before use.

The author assumes no liability for any damages arising from the use of this skill.

## Intended Use

This skill is intended as an assistive tool for developers. 
It is not guaranteed to produce secure, production-ready, or legally compliant code.

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

### [AI Security Risks](reference/ai.md)
Security considerations for apps using generative AI and Large Language Models (LLMs).

**Key Topics:**
- Prompt injection
- Sensitive information disclosure
- Excessive agency

### [Common Vulnerabilities](reference/common-vulnerabilities.md)
Security risks to avoid and testing strategies.

**Key Topics:**
- Configuration issues (debuggable, exported)
- Network vulnerabilities
- Data security issues
- Code injection risks
- WebView vulnerabilities

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
