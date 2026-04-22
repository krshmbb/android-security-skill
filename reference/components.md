Last updated: April 2026

# Android Components Security

Security guidelines for Services, BroadcastReceivers and Activities.

---

## Component Export Control

### Understanding `android:exported`

**Default Behavior**
- Components with intent filters: Exported by default
- Components without intent filters: Not exported by default

**Explicit Declaration Required**
```xml
<!-- Always explicitly declare exported attribute -->
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<service
    android:name=".MyService"
    android:exported="false" />
```

**Best Practice**
- Always explicitly set `android:exported`
- Set to `false` unless component needs to be accessed by other apps
- Protect exported services with a permission, preferably `signature` when appropriate
- Validate all incoming intent data

---

## Services Security

### Recommendations
- Always explicitly declare `android:exported`
- Do not rely on default export behavior

### Service Protection

**Manifest Declaration**
```xml
<service
    android:name=".SecureService"
    android:exported="true"
    android:permission="com.example.myapp.SERVICE_PERMISSION">
</service>

<!-- Define custom permission -->
<permission
    android:name="com.example.myapp.SERVICE_PERMISSION"
    android:protectionLevel="signature" />
```

---

## BroadcastReceiver Security

### Default Behavior
- If intent filter declared in manifest, receiver is exported
- Potentially accessible by any app
- Apply security permissions within the manifest to prevent unauthorized apps from sending intents to the receiver

### Receiver Protection

**Manifest Declaration**
```xml
<receiver
    android:name=".SecureReceiver"
    android:exported="true"
    android:permission="com.example.myapp.RECEIVE_PERMISSION">
    <intent-filter>
        <action android:name="com.example.myapp.CUSTOM_ACTION" />
    </intent-filter>
</receiver>

<!-- Define permission -->
<permission
    android:name="com.example.myapp.RECEIVE_PERMISSION"
    android:protectionLevel="signature" />
```

**Private Receiver (Not Exported)**
- Not accessible to other apps through normal IPC
- Intended for internal app use only

```xml
<receiver
    android:name=".PrivateReceiver"
    android:exported="false">
    <!-- Only receives broadcasts from same app -->
</receiver>
```

### Sending Protected Broadcasts

**Require Permission**
```kotlin
// Send broadcast requiring receiver permission
sendBroadcast(
    Intent("com.example.myapp.ACTION"),
    "com.example.myapp.RECEIVE_PERMISSION"
)
```

---

## Activities Security

### Export Control

**Launch Activity**
```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

**Internal Activity**
```xml
<activity
    android:name=".InternalActivity"
    android:exported="false" />
```

### Activity Permissions

**Protected Activity**
```xml
<activity
    android:name=".SensitiveActivity"
    android:exported="true"
    android:permission="com.example.myapp.ACCESS_SENSITIVE">
    <intent-filter>
        <action android:name="com.example.myapp.OPEN_SENSITIVE" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>

<permission
    android:name="com.example.myapp.ACCESS_SENSITIVE"
    android:protectionLevel="signature" />
```

### Deep Link Security

**Validate Deep Link URIs**
```kotlin
class DeepLinkActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Handle deep link
        intent?.data?.let { uri ->
            if (!isValidDeepLink(uri)) {
                finish()
                return
            }
            handleDeepLink(uri)
        }
    }

    private fun isValidDeepLink(uri: Uri): Boolean {
        // Validate scheme
        if (uri.scheme != "https") return false

        // Validate host
        if (uri.host != "example.com") return false

        // Validate path
        val allowedPaths = listOf("/profile", "/settings", "/content")
        if (!allowedPaths.any { uri.path?.startsWith(it) == true }) {
            return false
        }

        // Validate query parameters
        uri.queryParameterNames.forEach { param ->
            if (!isValidQueryParam(param, uri.getQueryParameter(param))) {
                return false
            }
        }

        return true
    }

    private fun isValidQueryParam(name: String, value: String?): Boolean {
        value ?: return false
        // Prevent path traversal
        if (value.contains("..")) return false
        // Prevent script injection
        if (value.contains("<") || value.contains(">")) return false
        return true
    }
}
```

**Deep Link Manifest**
```xml
<activity
    android:name=".DeepLinkActivity"
    android:exported="true">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="https"
            android:host="example.com"
            android:pathPrefix="/app" />
    </intent-filter>
</activity>
```

### Task Hijacking Prevention

**Default vs Custom Affinity**
```xml
<!-- Secure: Empty affinity prevents task hijacking -->
<activity
    android:name=".BankingActivity"
    android:taskAffinity=""
    android:exported="false" />

<!-- Risky: Custom affinity can be exploited -->
<activity
    android:name=".SharedActivity"
    android:taskAffinity="com.example.shared"
    android:exported="true" />
```

**Task Affinity Attacks**
- Malicious apps can use same taskAffinity to inject activities
- UI redressing attacks (phishing overlays)
- Activity lifecycle manipulation

**Prevention**
- Use empty taskAffinity (`android:taskAffinity=""`) for sensitive activities
- Keep activities in separate tasks when handling sensitive data
- Avoid exported activities with custom task affinities

**Security Flags**
- `android:taskAffinity=""` - Prevents other apps from placing activities in your task
- `android:excludeFromRecents="true"` - Hides sensitive activities from recent apps list
- `android:documentLaunchMode="never"` - Prevents creating separate task documents

**Runtime Protection**
LayoutParams.FLAG_SECURE tells Android not to allow screenshots or to display the window view on a non-secure display e.g. Casting the screen.

```kotlin
class SensitiveActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Prevent screenshots and screen recording
        window.setFlags(
            WindowManager.LayoutParams.FLAG_SECURE,
            WindowManager.LayoutParams.FLAG_SECURE
        )

        // Verify task affinity
        if (!isTaskSecure()) {
            finish()
            return
        }
    }

    private fun isTaskSecure(): Boolean {
        val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
        val tasks = activityManager.appTasks
        return tasks.isNotEmpty() && tasks[0].taskInfo.numActivities == 1
    }
}
```

---

## Component Metadata Security

### Process Isolation

**Isolated Processes**
```xml
<!-- Run service in isolated process (no permissions) -->
<service
    android:name=".UntrustedContentService"
    android:isolatedProcess="true"
    android:exported="false" />
```

**Use Cases for Isolated Processes**
- Processing untrusted data (web content, user files)
- Rendering third-party content
- Parsing complex file formats
- Running potentially vulnerable code

**Limitations**
- No access to app's normal permissions
- Cannot bind to other services outside the isolated process
- Cannot access shared app data

### Document Launch Mode

**Secure Configuration**
```xml
<!-- Prevent document-based task creation -->
<activity
    android:name=".SecureDocumentActivity"
    android:documentLaunchMode="never"
    android:maxRecents="1" />
```

**Document Launch Modes**
- `never` - Don't create new task documents (most secure)
- `intoExisting` - Reuse existing document task
- `always` - Always create new document (least secure for sensitive content)
- `none` - System decides (default)

### Component Aliases

**Security Considerations**
```xml
<!-- Activity alias can be enabled/disabled at runtime -->
<activity-alias
    android:name=".AliasActivity"
    android:targetActivity=".RealActivity"
    android:exported="false"
    android:enabled="false" />
```

**Use Case: Dynamic Entry Points**
```kotlin
// Enable/disable alias based on feature flag or subscription
val componentName = ComponentName(this, ".AliasActivity")
packageManager.setComponentEnabledSetting(
    componentName,
    PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
    PackageManager.DONT_KILL_APP
)
```

**Security Implications**
- Aliases can be enabled without app restart
- Each alias needs separate export/permission configuration
- Aliases can leak app functionality if misconfigured

---

## Official Documentation

- [Interprocess communication](https://developer.android.com/privacy-and-security/security-tips#interprocess-communication)
