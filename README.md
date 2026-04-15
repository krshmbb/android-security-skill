# Android Security Skill

A comprehensive Claude Code skill for Android application security based exclusively on official Android documentation from Android Open Source Project (AOSP) and Android Developers.

## Overview

This skill provides authoritative Android security best practices, code examples, and guidelines to help developers build secure Android applications.

## Skill Structure

```
android-security-skill/
├── SKILL.md                              # Main entry point with overview
├── README.md                             # This file
└── reference/
    ├── communication-security.md         # Network and IPC security
    ├── data-storage.md                   # Secure data storage
    ├── authentication.md                 # Auth and credentials
    ├── permissions.md                    # Permission best practices
    ├── cryptography.md                   # Crypto implementations
    ├── components.md                     # Android components security
    ├── ai.md                             # AI/LLM security risks
    └── common-vulnerabilities.md         # Common security risks    
```

## Topics Covered
- Communication Security
- Data Storage
- Authentication & Credentials
- Permissions
- Cryptography
- Android Components
- AI Security Risks
- Common Vulnerabilities

## Quick Start

1. **Copy this skill directory** to one of these locations:
   - Personal: `~/.claude/skills/android-security/`
   - Project: `.claude/skills/android-security/`

2. **Use the skill** by asking Claude about Android security:
   - "How do I securely store API keys in Android?"
   - "What are the best practices for Android network security?"
   - "Help me implement biometric authentication"
   - "Review this code for security vulnerabilities"

3. **Invoke directly** with `/android-security` followed by your question

## Key Features

✅ **Based on Official Documentation**
- Android Open Source Project (AOSP)
- Android Developers (developer.android.com)
- Current as of April 2026

✅ **Comprehensive Coverage**
- App security best practices
- System security guidelines
- Code examples in Kotlin
- Security checklists

✅ **Progressive Disclosure**
- Quick reference in SKILL.md
- Detailed guidance in reference files
- Load only what you need

✅ **Practical Code Examples**
- Real-world implementations
- Security patterns
- Anti-patterns to avoid

## Official Documentation Sources

### Android Developers
- [Security Best Practices](https://developer.android.com/privacy-and-security/security-best-practices)
- [Security Checklist](https://developer.android.com/privacy-and-security/security-tips)
- [Android Keystore](https://developer.android.com/privacy-and-security/keystore)
- [Network Security Config](https://developer.android.com/privacy-and-security/security-config)
- [Cryptography](https://developer.android.com/privacy-and-security/cryptography)
- [AI Risks and Mitigations](https://developer.android.com/privacy-and-security/risks/ai-risks/risks-mitigations)

### Android Open Source Project
- [Security Best Practices](https://source.android.com/docs/security/best-practices)
- [App Security](https://source.android.com/docs/security/best-practices/app)
- [System Security](https://source.android.com/docs/security/best-practices/system)
- [Security Overview](https://source.android.com/docs/security)
- [Security Bulletins](https://source.android.com/docs/security/bulletin)

## Version

**Last Updated:** April 2026

**Based on:**
- Android Security Best Practices (Updated March 30, 2026)
- Android Security Checklist (Updated March 6, 2026)
- Android AI Risks and Mitigations (Updated December 16, 2025)
- Android Open Source Project Security Documentation (2026)

## License

MIT License - see [LICENSE](LICENSE) file for details.

This skill is based on official Android documentation, which is licensed under the Apache License 2.0.

## Feedback

If you find outdated information or need additional Android security topics covered, please ensure requests are based on official Android documentation.
