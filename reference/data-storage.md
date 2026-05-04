Last updated: April 2026

# Data Storage Security

Guidelines for secure storage of sensitive data and logging in Android applications.

---

## Internal Storage (Most Secure)

### Overview

**Security Characteristics**
- Sandboxed per app
- Accessible only to your app by default
- No special permissions needed
- Auto-deleted on app uninstall
- Most secure storage option for private data

**When to Use**
- All private user data
- Sensitive information
- Authentication tokens (preferably encrypted, with encryption key anchored in the KeyStore)
- Personal identifiable information (PII)

**Never Use Deprecated Modes**
- `MODE_WORLD_WRITEABLE` - Deprecated, insecure
- `MODE_WORLD_READABLE` - Deprecated, insecure
- If you want to share data with other app processes, use content provider instead.

---

## External Storage

### Security Considerations

**Important Warnings**
- No security enforcement
- Any app with `WRITE_EXTERNAL_STORAGE` permission can access (Android 10 or lower)
- Android 11+ enforces Scoped Storage, providing better isolation
- Store only non-sensitive data (if you have to, encrypt sensitive data with encryption key anchored in keystore)
- Perform input validation on all data read from external storage
- Don't store executables or class files (If you do, ensure files are signed and cryptographically verified)

### When to Use

**App-Specific Files**
- Large, non-sensitive files
- Cache files > 1 MB
- Files that can be public

**Shared Files**
- Media files: Use Media Store API
- Other files: Use Storage Access Framework

### Validation Requirements

**Storage Availability**
```kotlin
// Verify storage is available before access
if (Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED) {
    // Storage is available for read/write
}
```

**Data Validation**
```kotlin
// Hash verify data validity
fun validateFile(file: File, expectedHash: String): Boolean {
    val hash = calculateHash(file.inputStream())
    return hash == expectedHash
}

// Never trust external data without validation
```

**Content Validation**
```kotlin
// Validate file content
fun validateFileContent(file: File): Boolean {
    // Check file size
    if (file.length() > MAX_FILE_SIZE) return false

    // Verify file signature/magic bytes
    val header = file.inputStream().use { it.readNBytes(4) }
    return header.contentEquals(EXPECTED_HEADER)
}
```

**Critical Rules**
- Don't store executables or class files on external storage
- Cryptographically verify files before dynamic loading
- Validate all input from external storage
- Never assume external data is trustworthy

---

## Cache Files

### Cache Directory Selection
Provides faster access to non-sensitive app data

**Small Cache (≤ 1 MB)**
```kotlin
// Use internal cache
val cacheDir = context.cacheDir
val cacheFile = File(cacheDir, "temp_data.txt")
```

**Large Cache (> 1 MB)**
```kotlin
// Use external cache
val largeCacheFile = context.externalCacheDir?.let { externalCacheDir ->
    File(externalCacheDir, "large_temp_data.bin")
}
```

### Security Considerations

**Internal Cache**
- Sandboxed to your app
- More secure
- Limited space

**External Cache**
- No security enforcement
- Apps with `WRITE_EXTERNAL_STORAGE` can access (Android 10 or lower)
- More space available
- Store only non-sensitive data

**Best Practices**
- Store only non-sensitive data in cache
- Don't rely on cache for critical data
- Cache can be cleared by system at any time

---

## SharedPreferences

### Secure Usage

**Always Use MODE_PRIVATE**
```kotlin
// Correct - Private mode
val sharedPref = context.getSharedPreferences(
    "my_prefs",
    Context.MODE_PRIVATE
)

// Write data
with (sharedPref.edit()) {
    putString("key", "value")
    apply()
}

// Read data
val value = sharedPref.getString("key", "default")
```

### Security Rules

**Never Use for Cross-App Sharing**
- Don't use SharedPreferences to share data between apps
- Use ContentProviders for cross-app data sharing

**What to Store**
- User preferences
- Settings
- Non-sensitive configuration
- UI state

**What NOT to Store**
- Passwords or credentials
- Authentication tokens (use private app storage)
- Sensitive user data
- Encryption keys

---

## Content Providers

### Overview

Content providers offer a structured storage mechanism that can be limited to your own application or exported to allow access by other applications.

### Export Configuration

**Private Content Provider (Default)**
```xml
<!-- AndroidManifest.xml -->
<!-- For app-only access -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="false" />
```

**Public Content Provider**
```xml
<!-- For cross-app access -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="true"
    android:readPermission="com.example.app.READ_PROVIDER"
    android:writePermission="com.example.app.WRITE_PROVIDER" />
```

### Permission Configuration

**Single Permission**
```xml
<!-- Define custom "normal" permission -->
<!-- For low-risk shared data only. Use stronger controls for sensitive data. -->
<permission
    android:name="com.example.app.ACCESS_PROVIDER"
    android:protectionLevel="normal" />

<!-- Apply to provider -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="true"
    android:permission="com.example.app.ACCESS_PROVIDER" />
```

**Separate Read/Write Permissions**
```xml
<!-- Define "normal" permissions -->
<!-- For low-risk shared data only. Use stronger controls for sensitive data. -->
<permission
    android:name="com.example.app.READ_PROVIDER"
    android:protectionLevel="normal" />
<permission
    android:name="com.example.app.WRITE_PROVIDER"
    android:protectionLevel="normal" />

<!-- Apply to provider -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="true"
    android:readPermission="com.example.app.READ_PROVIDER"
    android:writePermission="com.example.app.WRITE_PROVIDER" />
```

**Normal Permission Limitations**
- `normal` permissions are granted automatically and are only appropriate for low-risk data
- Use `signature` permissions for data shared only between apps signed with the same key
- Add app-level authorization checks before returning sensitive rows or fields
- Treat exported providers as public APIs, even when protected by manifest permissions

**Signature Protection (Same-Developer Apps)**
```xml
<!-- For sharing between your own apps only -->
<permission
    android:name="com.example.app.ACCESS_PROVIDER"
    android:protectionLevel="signature" />

<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="true"
    android:permission="com.example.app.ACCESS_PROVIDER" />
```

**Benefits of Signature Protection**
- No user confirmation required
- Better user experience
- Controlled access when apps signed with same key
- Recommended for same-developer app sharing

### Granular URI Permissions

**Temporary Access Grants**
```xml
<!-- Enable URI permissions -->
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.app.provider"
    android:exported="true"
    android:grantUriPermissions="true">

    <!-- Limit scope of grants -->
    <grant-uri-permission
        android:pathPattern="/shared/.*" />
</provider>
```

```kotlin
// Grant temporary read/write access
val intent = Intent(Intent.ACTION_VIEW).apply {
    data = contentUri
    flags = Intent.FLAG_GRANT_READ_URI_PERMISSION or
            Intent.FLAG_GRANT_WRITE_URI_PERMISSION
}
startActivity(intent)
```

### SQL Injection Prevention

**Always Use Parameterized Queries**
```kotlin
// Good - Safe parameterized query
override fun query(
    uri: Uri,
    projection: Array<String>?,
    selection: String?,
    selectionArgs: Array<String>?,
    sortOrder: String?
): Cursor? {
    val db = dbHelper.readableDatabase

    // Use selection and selectionArgs - never concatenate
    return db.query(
        "table_name",
        projection,
        selection,  // Use placeholder: "column = ?"
        selectionArgs,  // Provide values separately
        null, null, sortOrder
    )
}
```

**Avoid String Concatenation**
```kotlin
// BAD - SQL Injection vulnerability
val selection = "user_id = $userId"  // NEVER do this!

// GOOD - Use parameterized query
val selection = "user_id = ?"
val selectionArgs = arrayOf(userId.toString())
```

**Important Warning**
- Parameterized methods alone are not sufficient
- Building selection argument by concatenating user data is still vulnerable
- Always use selection placeholders (?) with separate arguments

### Write Permission Security Warning

**Critical Security Consideration**

The write permission allows SQL statements that can be exploited to read data:

**Attack Scenario**
```kotlin
// Attacker can probe for specific data
// Update row only if phone number exists
val selection = "phone_number = '555-1234'"
val values = ContentValues().apply {
    put("note", "probed")
}
// If update count > 0, phone number exists
val count = contentResolver.update(uri, values, selection, null)
```

**Key Points**
- Write permission enables creative WHERE clauses
- Attackers can parse results to confirm data presence
- Predictable content structure makes write ≈ read+write
- Don't assume write permission is less sensitive than read

### Security Best Practices

**Permission Guidelines**
- Mark as `android:exported="false"` if not sharing with other apps
- Set `android:exported="true"` only when cross-app access is required
- Limit permissions to minimum required
- Easier to add permissions later than remove them
- Use signature protection for same-developer apps

**Access Control**
- Implement separate read/write permissions when possible
- Use URI permissions for temporary, scoped access
- Validate all input to prevent SQL injection
- Never concatenate user input in queries

**Data Validation**
- Use parameterized query methods (query, update, delete)
- Validate selection arguments before use
- Treat all external data as untrusted
- Remember write permission security implications

---

## Data Protection Best Practices

### Minimize Data Collection

**Principles**
- Minimize sensitive/personal information usage
- Avoid storing/transmitting user data if possible
- Use hashes or non-reversible forms of data
- Create UUID for app instead of using device identifiers

**Example**
```kotlin
// Good - use UUID instead of device ID
val appInstanceId = UUID.randomUUID().toString()

// Bad - avoid device identifiers
// val deviceId = Settings.Secure.getString(
//     contentResolver,
//     Settings.Secure.ANDROID_ID
// )
```

### Secure Deletion

**Clear Sensitive Data**
```kotlin
// Clear from memory
val sensitiveData = CharArray(size)
try {
    // Use data
} finally {
    sensitiveData.fill('\u0000')  // Overwrite with zeros
}
```

---

## Logging Security

### Never Log PII

**What NOT to Log**
- Personally Identifiable Information (PII)
- Passwords or credentials
- Authentication tokens
- Credit card numbers
- Social security numbers
- Personal addresses
- Phone numbers

**Safe Logging Practice**
- Sanitize any logs in production containing sensitive data
- Redact sensitive data in logs (sanitize using tokenization, data masking, redaction or filtering)
- Don’t use data masking when partial exposure of sensitive data can still compromise security (e.g. passwords)
- Log printing should only be performed through a “logs sanitizer” component

**Production Logging**
```kotlin
// Custom log class for production
object SecureLog {
    fun d(tag: String, message: String) {
        if (BuildConfig.DEBUG) {
            Log.d(tag, message)
        }
    }

    fun e(tag: String, message: String) {
        // Log errors even in production
        // But sanitize sensitive data first
        Log.e(tag, sanitize(message))
    }
}
```

---

## Official Documentation

- [Data Storage](https://developer.android.com/privacy-and-security/security-tips#data-storage)
- [Security with Dynamically Loaded Code](https://developer.android.com/privacy-and-security/security-tips#dynamic-code)
- [Log Info Disclosure](https://developer.android.com/privacy-and-security/risks/log-info-disclosure)
