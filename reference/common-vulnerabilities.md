Last updated: April 2026

# Common Vulnerabilities

Security risks to avoid and testing strategies for Android applications.

---

## Configuration Vulnerabilities

### android:debuggable

**Critical Issue**
- **NEVER enable in production builds**
- Allows debugging and inspection of app
- Enables code injection
- Exposes sensitive data

**Prevention**
```xml
<!-- AndroidManifest.xml -->
<!-- Production builds should NEVER have this -->
<application
    android:debuggable="false"
    ...>
```

**Verification**
```gradle
// build.gradle
android {
    buildTypes {
        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt')
        }
        debug {
            debuggable true  // OK for debug builds only
        }
    }
}
```

### android:exported

**Issue**
- Components without explicit export declaration
- May be unexpectedly accessible by other apps
- Attack surface expansion

**Prevention**
```xml
<!-- Always explicitly declare exported attribute -->
<activity
    android:name=".MainActivity"
    android:exported="true" />

<service
    android:name=".MyService"
    android:exported="false" />

<receiver
    android:name=".MyReceiver"
    android:exported="false" />

<provider
    android:name=".MyProvider"
    android:authorities="com.example.provider"
    android:exported="false" />
```

**Critical for Android 12+**
- Required for all components with intent filters
- Build fails if not explicitly set

---

## Network Vulnerabilities

### Cleartext Communications

**Issue**
- Unencrypted HTTP traffic
- Man-in-the-middle attacks
- Data interception

**Prevention**
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

**Manifest Declaration**
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false"
    ...>
```

### Broken Cryptographic Algorithms

**Avoid Weak Algorithms**
```kotlin
// BAD - Weak algorithms
// MessageDigest.getInstance("MD5")
// MessageDigest.getInstance("SHA1")
// Cipher.getInstance("DES")
// Cipher.getInstance("RC4")

// GOOD - Strong algorithms
MessageDigest.getInstance("SHA-256")
Cipher.getInstance("AES/GCM/NoPadding")
Mac.getInstance("HmacSHA256")
```

### Insecure SSL/TLS Configuration

**Issues to Avoid**
```kotlin
// BAD - Trust all certificates
// class TrustAllCerts : X509TrustManager {
//     override fun checkServerTrusted(...) { }  // No validation!
// }

// BAD - Disable hostname verification
// hostnameVerifier = HostnameVerifier { _, _ -> true }

// GOOD - Use default SSL configuration
val connection = url.openConnection() as HttpsURLConnection
// Uses system default trust manager and hostname verifier
```

---

## Data Security Vulnerabilities

### Hardcoded Cryptographic Secrets

**Issue**
- Keys, passwords, or tokens in source code
- Easily extractable from APK
- Cannot be rotated without app update

**Prevention**
```kotlin
// BAD - Hardcoded secrets
// const val API_KEY = "sk_live_1234567890"
// const val ENCRYPTION_KEY = "my_secret_key"

// GOOD - Use Android Keystore
val keyStore = KeyStore.getInstance("AndroidKeyStore")
keyStore.load(null)
val key = keyStore.getKey("my_key", null)

// GOOD - Load from secure storage
val apiKey = SecurePreferences.getString("api_key")
```

### Insecure Data Storage

**Common Issues**
```kotlin
// BAD - Storing sensitive data in SharedPreferences
// val prefs = getSharedPreferences("data", MODE_WORLD_READABLE)
// prefs.edit().putString("password", userPassword).apply()

// BAD - Storing on external storage
// val file = File(Environment.getExternalStorageDirectory(), "secrets.txt")

// GOOD - Use internal storage
val file = File(filesDir, "user_data.txt")

// GOOD - Use EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
```

### Logging Sensitive Data

**Issue**
- PII, credentials, or sensitive data in logs
- Accessible via logcat
- Persists in system logs

**Prevention**
```kotlin
// BAD - Logging sensitive data
// Log.d(TAG, "User password: $password")
// Log.d(TAG, "Credit card: $cardNumber")
// Log.d(TAG, "API key: $apiKey")

// GOOD - Sanitized logging
Log.d(TAG, "User login attempt")
Log.d(TAG, "Payment processed successfully")

// GOOD - Debug-only logs
if (BuildConfig.DEBUG) {
    Log.d(TAG, "Debug info")
}

// GOOD - Custom log wrapper
object SecureLog {
    fun d(tag: String, message: String) {
        if (BuildConfig.DEBUG && !containsSensitiveData(message)) {
            Log.d(tag, message)
        }
    }
}
```

---

## Code Vulnerabilities

### SQL Injection

**Issue**
- Unsanitized user input in SQL queries
- Database manipulation
- Data exfiltration

**Prevention**
```kotlin
// BAD - SQL injection vulnerability
// val query = "SELECT * FROM users WHERE name = '$userName'"
// db.rawQuery(query, null)

// GOOD - Parameterized queries
val selection = "name = ?"
val selectionArgs = arrayOf(userName)
db.query("users", null, selection, selectionArgs, null, null, null)

// GOOD - Using Room
@Query("SELECT * FROM users WHERE name = :name")
fun getUserByName(name: String): User
```

### Cross-App Scripting

**Issue**
- Loading untrusted web content in WebView
- JavaScript execution on sensitive data
- Data leakage to malicious sites

**Prevention**
```kotlin
// GOOD - Disable JavaScript for untrusted content
webView.settings.javaScriptEnabled = false

// GOOD - Validate URLs
fun loadUrl(url: String) {
    if (isTrustedDomain(url)) {
        webView.loadUrl(url)
    } else {
        // Reject untrusted URLs
    }
}

// GOOD - Use allowlist
val TRUSTED_DOMAINS = setOf("example.com", "trusted.com")
fun isTrustedDomain(url: String): Boolean {
    val uri = Uri.parse(url)
    return TRUSTED_DOMAINS.contains(uri.host)
}
```

### Unsafe Deserialization

**Issue**
- Deserializing untrusted data
- Remote code execution
- Object injection attacks

**Prevention**
```kotlin
// BAD - Deserializing untrusted data
// val obj = ObjectInputStream(untrustedData).readObject()

// GOOD - Validate before deserialization
fun deserializeSafely(data: ByteArray): MyObject? {
    // Verify data signature
    if (!verifySignature(data)) return null

    // Verify data format
    if (!isValidFormat(data)) return null

    // Use safe serialization library
    return json.decodeFromString<MyObject>(String(data))
}

// BETTER - Use JSON instead of Java serialization
val gson = Gson()
val obj = gson.fromJson(jsonString, MyObject::class.java)
```

### XML External Entity (XXE) Injection

**Issue**
- Processing untrusted XML with external entities
- File disclosure
- Server-side request forgery

**Prevention**
```kotlin
// GOOD - Disable external entities
val factory = DocumentBuilderFactory.newInstance()
factory.setFeature(
    "http://apache.org/xml/features/disallow-doctype-decl",
    true
)
factory.setFeature(
    "http://xml.org/sax/features/external-general-entities",
    false
)
factory.setFeature(
    "http://xml.org/sax/features/external-parameter-entities",
    false
)
factory.setExpandEntityReferences(false)

val builder = factory.newDocumentBuilder()
val document = builder.parse(xmlInput)
```

---

## Intent Vulnerabilities

### Implicit Intent Hijacking

**Issue**
- Implicit intents can be intercepted
- Sensitive data exposure
- Malicious app substitution

**Prevention**
```kotlin
// BAD - Implicit intent with sensitive data
// val intent = Intent(Intent.ACTION_SEND)
// intent.putExtra(Intent.EXTRA_TEXT, sensitiveData)
// startActivity(intent)

// GOOD - Explicit intent
val intent = Intent(this, TrustedActivity::class.java)
intent.putExtra("data", data)
startActivity(intent)

// GOOD - Verify receiver before sending
val intent = Intent(Intent.ACTION_SEND)
if (intent.resolveActivity(packageManager) != null) {
    startActivity(Intent.createChooser(intent, "Share via"))
} else {
    // No app available
}
```

### Content Resolver Vulnerabilities

**Issue**
- Unvalidated ContentProvider queries
- SQL injection
- Data leakage

**Prevention**
```kotlin
// GOOD - Validate URI
fun queryProvider(uri: Uri): Cursor? {
    if (!isTrustedProvider(uri)) {
        throw SecurityException("Untrusted provider")
    }

    // Validate query parameters
    val cursor = contentResolver.query(
        uri,
        projection,
        selection,
        selectionArgs,
        sortOrder
    )

    return cursor
}

fun isTrustedProvider(uri: Uri): Boolean {
    val authority = uri.authority
    return authority == "com.example.myapp.provider"
}
```

---

## WebView Vulnerabilities

### JavaScript Enabled for Untrusted Content

**Issue**
- XSS attacks
- Data theft
- Unauthorized actions

**Prevention**
```kotlin
// GOOD - JavaScript disabled by default
webView.settings.javaScriptEnabled = false

// Only enable for trusted content
if (isTrustedUrl(url)) {
    webView.settings.javaScriptEnabled = true
}
```

### Insecure JavaScript Interface

**Issue**
- Exposes app functionality to JavaScript
- Can be exploited if WebView loads untrusted content

**Prevention**
```kotlin
// BAD - Exposing interface to untrusted content
// webView.addJavascriptInterface(myInterface, "Android")
// webView.loadUrl(untrustedUrl)

// GOOD - Only for trusted content
if (isTrustedUrl(url)) {
    webView.addJavascriptInterface(myInterface, "Android")
    webView.loadUrl(url)
}

// BETTER - Use WebMessagePort (Android 6.0+)
val ports = webView.createWebMessageChannel()
webView.postWebMessage(
    WebMessage("", arrayOf(ports[1])),
    Uri.parse("*")
)
```

### File Access

**Issue**
- File:// URLs can access local files
- Content:// URLs can access ContentProviders

**Prevention**
```kotlin
// GOOD - Disable file access
webView.settings.allowFileAccess = false
webView.settings.allowContentAccess = false
webView.settings.allowFileAccessFromFileURLs = false
webView.settings.allowUniversalAccessFromFileURLs = false
```

---

- [Common Risks](https://developer.android.com/privacy-and-security/risks)
