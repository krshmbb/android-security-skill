Last updated: April 2026

# Communication Security

Guidelines for secure network communication, IPC, and WebView security.

---

## Network Security

### Always Use TLS/HTTPS

**Requirements**
- Use HTTPS for all server communication with trusted Certificate Authorities
- Prefer `HttpsURLConnection` for secure web traffic
- Use `SSLSocket` for encrypted socket-level communication
- Don't use localhost network ports for sensitive IPC
- Use Android IPC mechanisms (Service, Intent) instead

### Network Security Configuration

**Configuration File**
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
    <debug-overrides>
        <!-- Debug build only overrides -->
    </debug-overrides>
</network-security-config>
```

**Manifest Declaration**
```xml
<application android:networkSecurityConfig="@xml/network_security_config">
```

**Best Practices**
- Disable cleartext traffic: `cleartextTrafficPermitted="false"` (disabled by default in Android 9+)
- Use `<debug-overrides>` only during development
- Implement certificate pinning for additional security

### IP Networking

**Security Requirements**
- Validate input from HTTP and insecure protocols
- Don't trust data downloaded over insecure protocols
- Use SSL/HTTPS for all API communication
- Validate and sanitize user input

### Telephony Networking

**Best Practices**
- Prefer secure, encrypted channels (e.g. FCM, IP networking) instead of SMS for data messages
- SMS is neither encrypted nor strongly authenticated
- Don't rely on unauthenticated SMS for sensitive commands
- Be aware of SMS spoofing and interception risks

---

## Intents

### Prefer Explicit Intents

**Best Practices**
- Use explicit intents, or call `setPackage()`, when you know the target app or component
- Use implicit intents with a chooser when the user should select the app
- Avoid placing sensitive data in implicit intents unless the receiver is constrained
- **NEVER use implicit intents for Services** (security hazard)
- Perform input validation in every exported intent receiver

**Explicit Intent Example**
```kotlin
val intent = Intent(this, TargetActivity::class.java)
startActivity(intent)
```

**Constrained Cross-App Intent**
```kotlin
val intent = Intent("com.example.partnerapp.SECURE_ACTION").apply {
    setPackage("com.example.partnerapp")
}
startActivity(intent)
```

**Implicit Intent with Chooser**
```kotlin
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, shareText)
}

val possibleActivities = packageManager.queryIntentActivities(
    intent,
    PackageManager.MATCH_ALL
)

if (possibleActivities.size > 1) {
    startActivity(Intent.createChooser(intent, "Share with"))
} else if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
}
```

---

## PendingIntent

- Lets your app delegate a predefined action to another app or the system
- The action executes with your app's permissions
- Grants temporary access to your app's capabilities
- Requires careful security handling

### Immutability Requirement (API 31+)

**Always Use FLAG_IMMUTABLE by default**
```kotlin
// Android 12 (API 31) and above require explicit mutability flag
val pendingIntent = PendingIntent.getActivity(
    context,
    REQUEST_CODE,
    intent,
    PendingIntent.FLAG_IMMUTABLE // Required for API 31+
)
```

### Mutable PendingIntents (Use with Caution)

**When FLAG_MUTABLE is Required**

Only use `PendingIntent.FLAG_MUTABLE` when there is a specific, documented platform requirement for mutation, such as:
- Inline reply actions in notifications
- `RemoteInput`
- Certain `Notification.CarExtender` cases
- Location-based intents that need updates
- Media controls with dynamic actions
- Other framework-driven cases where the system must modify the intent contents after creation

```kotlin
// Only use FLAG_MUTABLE when absolutely necessary
// Example: Notification actions that need to be updated
val pendingIntent = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    PendingIntent.getBroadcast(
        context,
        REQUEST_CODE,
        intent,
        PendingIntent.FLAG_MUTABLE or PendingIntent.FLAG_UPDATE_CURRENT
    )
} else {
    PendingIntent.getBroadcast(
        context,
        REQUEST_CODE,
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT
    )
}
```

### Security Best Practices

**Use Explicit Intents**
```kotlin
// GOOD: Explicit intent
val intent = Intent(context, SecureActivity::class.java).apply {
    putExtra("data", validatedData)
}
val pendingIntent = PendingIntent.getActivity(
    context,
    REQUEST_CODE,
    intent,
    PendingIntent.FLAG_IMMUTABLE
)

// BAD: Implicit intent (can be hijacked)
val badIntent = Intent("com.example.myapp.ACTION") // Avoid this
```

**Specify Component Name**
```kotlin
// Even better: Explicitly set component
val intent = Intent().apply {
    component = ComponentName(context, SecureActivity::class.java)
    putExtra("data", validatedData)
}
```

**Fill-in Intents Attack Prevention**
```kotlin
// If using FLAG_MUTABLE, limit what can be filled in
val baseIntent = Intent(context, SecureActivity::class.java)
val pendingIntent = PendingIntent.getActivity(
    context,
    REQUEST_CODE,
    baseIntent,
    PendingIntent.FLAG_MUTABLE
)

// When using the PendingIntent, restrict fill-in
try {
    pendingIntent.send(
        context,
        0,
        null, // Don't allow intent modifications
        null,
        null
    )
} catch (e: PendingIntent.CanceledException) {
    Log.e("PendingIntent", "PendingIntent was cancelled", e)
}
```

### Common Vulnerabilities

**Vulnerability: Implicit Intent Hijacking**
```kotlin
// VULNERABLE: Other apps can intercept this
val vulnerableIntent = Intent("CUSTOM_ACTION")
val vulnerablePendingIntent = PendingIntent.getBroadcast(
    context,
    0,
    vulnerableIntent,
    PendingIntent.FLAG_IMMUTABLE
)
```

**Fix: Use Explicit Intents**
```kotlin
// SECURE: Explicitly target your component
val secureIntent = Intent(context, SecureReceiver::class.java)
val securePendingIntent = PendingIntent.getBroadcast(
    context,
    0,
    secureIntent,
    PendingIntent.FLAG_IMMUTABLE
)
```

**Why This Matters**
- PendingIntents delegate your app's capabilities to other processes
- Implicit intents can be intercepted by malicious apps
- Mutable PendingIntents can be modified by receiving apps
- Always use the most restrictive configuration possible

---

## Binder & Messenger

### Best Practices

**Design Principles**
- Preferred for RPC-style IPC
- Design without interface-specific permission checks
- Inherit manifest permissions from Service/Activity
- Use `checkCallingPermission()` for access control

**Permission Management**
```kotlin
// Check calling permission
if (checkCallingPermission("com.example.PERMISSION")
    != PackageManager.PERMISSION_GRANTED) {
    throw SecurityException("Permission denied")
}

// Clear calling identity for cross-process calls
// Note: This is primarily needed for system services or privileged operations.
// Most third-party apps don't need to manipulate calling identity.
val identity = Binder.clearCallingIdentity()
try {
    // Perform operation
} finally {
    // Restore permissions
    Binder.restoreCallingIdentity(identity)
}
```

---

## WebView Security

### JavaScript Control

**Default Behavior**
- JavaScript is disabled by default (prevents XSS)
- Don't enable unless absolutely necessary

### JavaScript Interface

**Use with Extreme Caution**
It exposes native app methods to JavaScript in the WebView, allowing untrusted web content to invoke app functionality and potentially exploit sensitive operations if not properly restricted.

```kotlin
// Only expose a JavaScript interface when absolutely necessary
webView.addJavascriptInterface(jsInterface, "Android")
```

**Better Alternative (Android 6.0+)**
If your app must use web-to-app messaging on devices running Android 6.0 (API level 23) and higher, use HTML message channels over addJavascriptInterface(). However, message channels are not trusted by default. Always validate the sender origin and treat all incoming messages as untrusted input.

```kotlin
// Use HTML message channels for safer web-to-app communication
val ports = webView.createWebMessageChannel()
ports[0].setWebMessageCallback(object : WebMessagePort.WebMessageCallback() {
    override fun onMessage(port: WebMessagePort, message: WebMessage) {
        val data = message.data
        if (!isTrustedMessage(data)) return
        handleTrustedMessage(data)
    }
})
webView.postWebMessage(WebMessage("", arrayOf(ports[1])), Uri.parse("https://trusted-domain.com"))
```

### Content Restrictions

**Use Allowlists**
```kotlin
// Restrict content to specific domains
webView.webViewClient = object : WebViewClient() {
    override fun shouldOverrideUrlLoading(view: WebView, request: WebResourceRequest): Boolean {
        return !request.url.host.equals("trusted-domain.com")
    }
}
```

### Cache Management

**For Sensitive Data**
```kotlin
// Clear cache when handling sensitive data
webView.clearCache(true)

// Use server-side headers
// Cache-Control: no-store, no-cache
```

### Version-Specific Considerations

**Android < 4.4 (API Level 19)**
- Confirm WebView displays only trusted content

---

## Official Documentation

- [Networking](https://developer.android.com/privacy-and-security/security-tips#networking)
- [WebView](https://developer.android.com/privacy-and-security/security-tips#webview)
- [Interprocess communication](https://developer.android.com/privacy-and-security/security-tips#interprocess-communication)
