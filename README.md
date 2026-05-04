# Android Security Skill

A comprehensive security knowledge base for AI-assisted Android application development, based exclusively on official Android documentation from Android Open Source Project (AOSP) and Android Developers. Compatible with Claude Code, Android Studio, and other AI development tools.

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

### For Android Studio

**Note:** Only skills located within the project's codebase are supported.

1. **Download or clone this skill** to your local machine

2. **Import into your Android Studio project** by copying the skill directory to your project root in one of these locations:

   ```
   your-project/
   ├── .skills/
   │   └── android-security-skill/
   │       └── SKILL.md
   ```

   Or:

   ```
   your-project/
   ├── .agent/skills/
   │   └── android-security-skill/
   │       └── SKILL.md
   ```

3. **Use the skill:**

   **Automatic Activation**
   The agent automatically activates relevant skills based on your task. Simply prompt the agent with a related request:
   - "How do I securely store API keys in Android?"
   - "Review this code for security vulnerabilities"
   - "Help me implement biometric authentication"

   **Manual Invocation**
   Type `@android-security-skill` directly in the chat window to invoke the skill manually.

### For Claude Code

1. **Copy this skill directory** to one of these locations:
   - Personal: `~/.claude/skills/android-security-skill/`
   - Project: `.claude/skills/android-security-skill/`

2. **Use the skill** by asking Claude about Android security:
   - "How do I securely store API keys in Android?"
   - "What are the best practices for Android network security?"
   - "Help me implement biometric authentication"
   - "Review this code for security vulnerabilities"

3. **Invoke directly** with `/android-security-skill` followed by your question

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
