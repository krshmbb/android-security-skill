Last updated: April 2026

# Communication Security

Guidelines for secure network communication, inter-app communication, IPC, and WebView security.

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
- Disable cleartext traffic: `cleartextTrafficPermitted="false"`
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

## Inter-App Communication

### Show App Chooser for Sensitive Data

**When to Use**
- Implicit intent can launch multiple apps
- User needs to select trusted app for sensitive data

**Implementation**
```kotlin
// Query for at least 2 possible apps before showing chooser
val intent = Intent(Intent.ACTION_SEND)
val chooser = Intent.createChooser(intent, "Share via")
if (intent.resolveActivity(packageManager) != null) {
    startActivity(chooser)
}
```

**Benefits**
- Gives users control over which app receives their data
- Prevents accidental data sharing with untrusted apps

---

## Intents

### Prefer Explicit Intents

**Best Practices**
- Use explicit intents over implicit intents
- **NEVER use implicit intents for Services** (security hazard)
- Perform input validation in intent receivers

**Explicit Intent Example**
```kotlin
val intent = Intent(this, TargetActivity::class.java)
startActivity(intent)
```

**Implicit Intent with Verification**
```kotlin
val intent = Intent(Intent.ACTION_VIEW)
if (intent.resolveActivity(packageManager) != null) {
    startActivity(Intent.createChooser(intent, "Open with"))
}
```

---

## PendingIntent Security

PendingIntents allow your app to grant other apps or the system the ability to execute a predefined action with your app's permissions. This capability delegation mechanism requires careful security consideration.

### Immutability Requirement (Android 12+)

**Always Use FLAG_IMMUTABLE**
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

**Legitimate Use Cases for FLAG_MUTABLE**
- Inline reply actions in notifications
- Intents used with Notification.CarExtender
- Location-based intents that need updates
- Media controls with dynamic actions

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
```kotlin
// Only to JavaScript within your APK
webView.addJavascriptInterface(jsInterface, "Android")
```

**Better Alternative (Android 6.0+)**
If your app must use JavaScript interface support on devices running Android 6.0 (API level 23) and higher, use HTML message channels instead of communicating between a website and your app.

```kotlin
// Use HTML message channels for safer web-to-app communication
val ports = webView.createWebMessageChannel()
ports[0].setWebMessageCallback(object : WebMessagePort.WebMessageCallback() {
    override fun onMessage(port: WebMessagePort, message: WebMessage) {
        // Handle messages from JavaScript securely
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

**Android < 4.4**
- Confirm WebView displays only trusted content
- Use updatable security Provider for SSL protection

---

## Official Documentation

- [Networking](https://developer.android.com/privacy-and-security/security-tips#networking)
- [WebView](https://developer.android.com/privacy-and-security/security-tips#webview)
- [Interprocess communication](https://developer.android.com/privacy-and-security/security-tips#interprocess-communication)
