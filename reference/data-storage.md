# Data Storage Security

Guidelines for secure storage of sensitive data in Android applications.

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
- Authentication tokens
- Encryption keys (use Keystore for long-term keys)
- Personal identifiable information (PII)

### Implementation

**Write to Internal Storage**
```kotlin
val FILE_NAME = "sensitive_info.txt"
val fileContents = "This is some top-secret information!"

// Write file
File(filesDir, FILE_NAME).bufferedWriter().use { writer ->
    writer.write(fileContents)
}
```

**Read from Internal Storage**
```kotlin
val contents = File(filesDir, FILE_NAME).bufferedReader().useLines { lines ->
    lines.fold("") { working, line -> "$working\n$line" }
}
```

**File Paths**
```kotlin
// Get files directory
val filesDir = context.filesDir

// Get cache directory
val cacheDir = context.cacheDir
```

---

## External Storage

### Security Considerations

**Important Warnings**
- No security enforcement
- Any app with `WRITE_EXTERNAL_STORAGE` permission can access (Android 10 or lower)
- Store only non-sensitive data
- Perform input validation on all data read from external storage
- Cryptographically verify data before use

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

**Critical Rules**
- Don't store executables or class files on external storage
- Cryptographically verify files before dynamic loading
- Validate all input from external storage
- Never assume external data is trustworthy

---

## Cache Files

### Cache Directory Selection

**Small Cache (≤ 1 MB)**
```kotlin
// Use internal cache
val cacheDir = context.cacheDir
val cacheFile = File(cacheDir, "temp_data.txt")
```

**Large Cache (> 1 MB)**
```kotlin
// Use external cache
val externalCacheDir = context.externalCacheDir
val largeCacheFile = File(externalCacheDir, "large_temp_data.bin")
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

**Never Use Deprecated Modes**
- `MODE_WORLD_WRITEABLE` - Deprecated, insecure
- `MODE_WORLD_READABLE` - Deprecated, insecure

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
- Authentication tokens (use Keystore)
- Sensitive user data
- Encryption keys

---

## Input Validation

### Validate All External Data

**Sources Requiring Validation**
- Files from external storage
- Network data
- IPC data
- User input
- Intent extras
- Broadcast data

### Validation Strategies

**Type Validation**
```kotlin
// Validate data types
fun validateInput(data: Any): Boolean {
    return when (data) {
        is String -> data.length <= MAX_LENGTH
        is Int -> data in MIN_VALUE..MAX_VALUE
        else -> false
    }
}
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

**Hash Verification**
```kotlin
// Cryptographic verification
fun verifyIntegrity(file: File, expectedHash: String): Boolean {
    val actualHash = MessageDigest.getInstance("SHA-256").let { digest ->
        file.inputStream().use { input ->
            val buffer = ByteArray(8192)
            var bytesRead = input.read(buffer)
            while (bytesRead != -1) {
                digest.update(buffer, 0, bytesRead)
                bytesRead = input.read(buffer)
            }
            digest.digest().joinToString("") { "%02x".format(it) }
        }
    }
    return actualHash == expectedHash
}
```

### SQL Injection Prevention

**Use Parameterized Queries**
```kotlin
// Good - Parameterized query
val selection = "user_id = ?"
val selectionArgs = arrayOf(userId.toString())
val cursor = db.query(
    "users",
    projection,
    selection,
    selectionArgs,
    null, null, null
)
```

**Never Use String Concatenation**
```kotlin
// BAD - SQL Injection vulnerability
val query = "SELECT * FROM users WHERE user_id = $userId"
// Never do this!
```

**Use Safe Query Methods**
- `query()`
- `update()`
- `delete()`
- `insert()`

---

## Native Code Considerations

### Memory Management Errors to Prevent

**Buffer Overflows**
```c
// Bad - buffer overflow risk
char buffer[100];
strcpy(buffer, user_input);  // Unsafe!

// Good - bounded copy
char buffer[100];
strncpy(buffer, user_input, sizeof(buffer) - 1);
buffer[sizeof(buffer) - 1] = '\0';
```

**Use-After-Free**
```c
// Bad - use after free
free(ptr);
*ptr = value;  // Dangerous!

// Good - null after free
free(ptr);
ptr = NULL;
```

**Off-By-One Errors**
```c
// Bad - off by one
for (int i = 0; i <= array_size; i++) {
    array[i] = value;  // Writes past end!
}

// Good - correct bounds
for (int i = 0; i < array_size; i++) {
    array[i] = value;
}
```

### Android Security Mitigations

**Available Mitigations**
- ASLR (Address Space Layout Randomization)
- DEP (Data Execution Prevention)

**Important Note**
- These mitigate but don't prevent memory errors
- Careful pointer handling and buffer management still required

---

## Type-Safe Languages

### Prefer High-Level Languages

**Recommended**
- Kotlin (preferred)
- Java

**Use With Caution**
- JNI/NDK (native code)
- C/C++ (more error-prone)

**When Native Code is Necessary**
- Familiarize with Linux security best practices
- Be extra careful with memory management
- Test thoroughly for memory errors
- Consider using safer alternatives when possible

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

**Delete Files Securely**
```kotlin
// Delete file
val file = File(filesDir, "sensitive.txt")
if (file.exists()) {
    file.delete()
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
```kotlin
// Bad - logs sensitive data
Log.d(TAG, "User password: $password")

// Good - no sensitive data
Log.d(TAG, "Authentication attempt")

// Use debug flags
if (BuildConfig.DEBUG) {
    Log.d(TAG, "Debug info")
}
```

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

## Storage Security Checklist

Before releasing your app:

- [ ] Sensitive data stored in internal storage only
- [ ] External storage used only for non-sensitive data
- [ ] Input validation implemented for all external data
- [ ] Files from external storage cryptographically verified
- [ ] SharedPreferences uses MODE_PRIVATE
- [ ] No deprecated storage modes used
- [ ] SQL injection prevented with parameterized queries
- [ ] Cache files contain only non-sensitive data
- [ ] PII not logged
- [ ] Memory management secure in native code
- [ ] Data minimization principles followed
- [ ] Secure deletion implemented where needed

---

*Based on official Android documentation from developer.android.com and source.android.com*
