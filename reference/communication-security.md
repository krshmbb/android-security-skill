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
        <!-- Development-only overrides -->
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
- Use custom trust managers for custom/new CAs not trusted by device

### IP Networking

**Security Requirements**
- Validate input from HTTP and insecure protocols
- Don't trust data downloaded over insecure protocols
- Use SSL/HTTPS for all API communication
- Validate and sanitize user input

### Telephony Networking

**Best Practices**
- Use Firebase Cloud Messaging (FCM) instead of SMS for data transfer
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

### Signature-Based Permissions

**When to Use**
- Sharing data/functionality between apps you control
- Apps signed with same key

**Implementation**
```xml
<permission
    android:name="com.example.myapp.CUSTOM_PERMISSION"
    android:protectionLevel="signature" />
```

**Benefits**
- No user confirmation needed
- Verifies apps are signed with same key
- More secure than standard permissions

### Content Provider Security

**Export Control**
```xml
<!-- Disable access if not sharing with other apps -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.provider"
    android:exported="false" />
```

**Critical for Old Versions**
- Android 4.1.1 (API 16) and lower default is `exported="true"`
- Explicitly set to `false` if not sharing data

**Permission Control**
```xml
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.provider"
    android:exported="true"
    android:readPermission="com.example.READ_PROVIDER"
    android:writePermission="com.example.WRITE_PROVIDER"
    android:grantUriPermissions="true" />
```

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

### Broadcast Security

**Sending Broadcasts**
```kotlin
// Use sendBroadcast() with non-null permission parameter
sendBroadcast(intent, "com.example.CUSTOM_PERMISSION")

// Or use explicit broadcasts
val intent = Intent(this, MyReceiver::class.java)
sendBroadcast(intent)
```

**Important Note**
- Intent filters are NOT security features
- Don't rely on intent filters for security
- Implement proper permission checks

---

## Services

### Service Export Rules

**Default Behavior**
- Services are not exported by default
- Add `android:exported` attribute explicitly

**Security Configuration**
```xml
<service
    android:name=".MyService"
    android:exported="false"
    android:permission="com.example.SERVICE_PERMISSION" />
```

**Runtime Permission Checks**
```kotlin
override fun onBind(intent: Intent): IBinder? {
    if (checkCallingPermission("com.example.SERVICE_PERMISSION")
        != PackageManager.PERMISSION_GRANTED) {
        throw SecurityException("Permission denied")
    }
    return binder
}
```

**Background Services (Android 5.0+)**
- Use `JobScheduler` for background services
- More secure and efficient

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

## Broadcast Receivers

### Security Configuration

**Default Behavior**
- Receivers are exported by default
- Apply security via `<receiver>` element in manifest

**Manifest Configuration**
```xml
<receiver
    android:name=".MyReceiver"
    android:exported="true"
    android:permission="com.example.RECEIVE_PERMISSION">
    <intent-filter>
        <action android:name="com.example.ACTION" />
    </intent-filter>
</receiver>
```

**Best Practices**
- Prevent unpermitted apps from sending intents
- Specify permissions for senders
- Validate all received data

---

## WebView Security

### JavaScript Control

**Default Behavior**
- JavaScript is disabled by default (prevents XSS)
- Don't enable unless absolutely necessary

**When You Must Enable JavaScript**
```kotlin
// Only enable for content you fully control
webView.settings.javaScriptEnabled = true
```

### JavaScript Interface

**Use with Extreme Caution**
```kotlin
// Only expose to trustworthy content
// Only to JavaScript within your APK
webView.addJavascriptInterface(jsInterface, "Android")
```

**Better Alternative (Android 6.0+)**
- Use HTML message channels instead of JavaScript bridges
- More secure communication mechanism

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

## IPC General Security

### Export Control

**Component Declaration**
```xml
<!-- Set android:exported="false" if not for other apps -->
<service android:name=".MyService" android:exported="false" />
<receiver android:name=".MyReceiver" android:exported="false" />
<activity android:name=".MyActivity" android:exported="false" />
```

### Permission Application

**Manifest Permissions**
```xml
<!-- Apply security policies via <permission> element -->
<permission
    android:name="com.example.CUSTOM_PERMISSION"
    android:protectionLevel="signature" />

<!-- Use in components -->
<service
    android:name=".MyService"
    android:permission="com.example.CUSTOM_PERMISSION" />
```

### Same-Developer Apps

**Signature-Level Permissions**
- Use `signature-level` permission for apps you control
- Verifies same signing key
- No user interaction required

---

## Security Testing

### Network Security Testing

**Verify Configuration**
- Test that cleartext traffic is blocked
- Verify HTTPS is used for all network calls
- Test certificate pinning implementation
- Validate custom trust managers

### IPC Testing

**Test Components**
- Verify exported components are properly secured
- Test permission enforcement
- Validate input from all IPC sources
- Test intent filter behavior

### WebView Testing

**Security Checks**
- Verify JavaScript is disabled for untrusted content
- Test JavaScript interface restrictions
- Validate content allowlists
- Check cache clearing for sensitive operations

---

*Based on official Android documentation from developer.android.com and source.android.com*
