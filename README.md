# Android Security Skill

A comprehensive security knowledge base for AI-assisted Android application development, based exclusively on official Android documentation from Android Open Source Project (AOSP) and Android Developers. Compatible with Claude Code, OpenAI Codex, Google Gemini, and other AI development tools.

## Overview

This skill provides authoritative Android security best practices, code examples, and guidelines to help developers build secure Android applications with AI assistance.

## Target Audience

This skill is designed for **third-party Android application developers** building apps for the Google Play Store and other distribution channels. The content focuses on:

- **App-level security** - Securing your application code, data, and user interactions
- **Android SDK APIs** - Using public Android framework APIs available to all developers
- **Best practices** - Following Google's recommended security patterns for production apps

**Not primarily intended for:**
- Platform/AOSP developers working on the Android operating system itself
- System app developers requiring platform signing certificates
- Low-level framework modifications or custom ROM development

Some advanced topics (clearly marked as such) cover system-level APIs for completeness, but the core focus is on practical security for third-party app development.

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
    └── ai.md                             # AI/LLM security risks
```

## Topics Covered
- Communication Security
- Data Storage
- Authentication & Credentials
- Permissions
- Cryptography
- Android Components
- AI Security Risks

## Quick Start

### For Claude Code

1. **Copy this skill directory** to one of these locations:
   - Personal: `~/.claude/skills/android-security/`
   - Project: `.claude/skills/android-security/`

2. **Use the skill** by asking Claude about Android security:
   - "How do I securely store API keys in Android?"
   - "What are the best practices for Android network security?"
   - "Help me implement biometric authentication"
   - "Review this code for security vulnerabilities"

3. **Invoke directly** with `/android-security` followed by your question

### For OpenAI (ChatGPT/Codex)

1. **Upload reference files** as context:
   - Use "Upload files" to add relevant reference documents from the `reference/` directory
   - Or upload all reference files at once for comprehensive coverage

2. **Provide system context** in your first message:
   ```
   You are an Android security expert. Use the uploaded Android security documentation
   to provide accurate, secure coding guidance based on official Android best practices.
   ```

3. **Ask security questions** directly:
   - "Based on the Android security docs, how should I store API keys?"
   - "Review my authentication implementation for security issues"
   - "What are the security best practices for Content Providers?"

### For Google Gemini

1. **Upload documentation** to your conversation:
   - Click the attachment icon and upload relevant files from the `reference/` directory
   - For comprehensive help, upload multiple reference files

2. **Set the context** in your initial prompt:
   ```
   I'm developing an Android application. Use the uploaded Android security
   documentation from AOSP and Android Developers to help me implement secure
   features following official best practices.
   ```

3. **Query about Android security**:
   - "According to the Android security guidelines, how should I handle sensitive data?"
   - "Check my code for security vulnerabilities using the uploaded guidelines"
   - "What's the secure way to implement WebView according to Android docs?"

### General Usage Tips

- Reference specific files for focused help: "Based on `data-storage.md`, how should I..."
- Request code reviews: "Review this code against Android security best practices"
- Ask for implementation guidance: "Help me implement [feature] securely"
- Verify approaches: "Is this implementation secure according to Android guidelines?"

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
