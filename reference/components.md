# Android Components Security

Security guidelines for Services, BroadcastReceivers, ContentProviders, Activities, and Dynamic Code Loading.

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
- Critical for Android 12 (API 31) and above

---

## Services Security

### Default Behavior

**Services Are Not Exported by Default**
- Safer default for security
- Must explicitly export if needed by other apps

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

### Runtime Permission Checks

**Verify Caller Permissions**
```kotlin
class SecureService : Service() {

    override fun onBind(intent: Intent): IBinder? {
        // Check calling permission
        if (checkCallingPermission(REQUIRED_PERMISSION)
            != PackageManager.PERMISSION_GRANTED) {
            throw SecurityException(
                "Caller lacks required permission: $REQUIRED_PERMISSION"
            )
        }
        return binder
    }

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {
        // Verify caller for started services
        verifyCallerIdentity()
        return super.onStartCommand(intent, flags, startId)
    }

    private fun verifyCallerIdentity() {
        val callerUid = Binder.getCallingUid()
        // Verify UID is from trusted app
    }

    companion object {
        const val REQUIRED_PERMISSION = "com.example.myapp.SERVICE_PERMISSION"
    }
}
```

### Background Services (Android 5.0+)

**Use JobScheduler**
```kotlin
// More secure and efficient than traditional services
val jobScheduler = getSystemService(Context.JOB_SCHEDULER_SERVICE) as JobScheduler

val jobInfo = JobInfo.Builder(
    JOB_ID,
    ComponentName(this, MyJobService::class.java)
)
    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)
    .setPersisted(true)
    .build()

jobScheduler.schedule(jobInfo)
```

**Benefits**
- System manages execution
- Better battery optimization
- Automatic retry handling
- More secure than always-running services

### Service Security Checklist

- [ ] `android:exported` explicitly declared
- [ ] Permissions required for exported services
- [ ] Calling permissions verified at runtime
- [ ] Signature-level permissions for same-developer apps
- [ ] JobScheduler used for background work (Android 5.0+)
- [ ] Input validation for all service calls

---

## BroadcastReceiver Security

### Default Behavior

**Receivers Are Exported by Default**
- If intent filter declared in manifest, receiver is exported
- Potentially accessible by any app

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

// Or use explicit intent
val intent = Intent(this, MyReceiver::class.java)
sendBroadcast(intent)
```

### Receiving Broadcasts Securely

**Validate Sender**
```kotlin
class SecureReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // Validate intent action
        if (intent.action != EXPECTED_ACTION) {
            return
        }

        // Validate sender (if possible)
        validateSender(context)

        // Validate all data from intent
        val data = intent.getStringExtra("data")
        if (!isValidData(data)) {
            return
        }

        // Process broadcast
        processBroadcast(data)
    }

    private fun validateSender(context: Context) {
        // Check sender's signature if needed
        val callerUid = Binder.getCallingUid()
        // Verify caller
    }

    private fun isValidData(data: String?): Boolean {
        return data != null && data.length < MAX_LENGTH
    }

    companion object {
        const val EXPECTED_ACTION = "com.example.myapp.ACTION"
        const val MAX_LENGTH = 1000
    }
}
```

### LocalBroadcastManager (Deprecated)

**Alternative: In-App Event Bus**
```kotlin
// Use in-app event bus for internal communication
// LiveData, Flow, or EventBus library
// More efficient and secure than broadcasts
```

### BroadcastReceiver Checklist

- [ ] `android:exported` explicitly set
- [ ] Permissions required for exported receivers
- [ ] Sender permissions enforced for sensitive broadcasts
- [ ] All broadcast data validated
- [ ] Local broadcasts used for internal communication
- [ ] Intent filters not relied upon for security

---

## ContentProvider Security

### Export Control

**Explicit Export Declaration**
```xml
<!-- Not exported - only accessible by same app -->
<provider
    android:name=".PrivateProvider"
    android:authorities="com.example.myapp.private"
    android:exported="false" />

<!-- Exported with permissions -->
<provider
    android:name=".PublicProvider"
    android:authorities="com.example.myapp.public"
    android:exported="true"
    android:readPermission="com.example.myapp.READ_PROVIDER"
    android:writePermission="com.example.myapp.WRITE_PROVIDER" />
```

### Permission Control

**Separate Read/Write Permissions**
```xml
<permission
    android:name="com.example.myapp.READ_PROVIDER"
    android:protectionLevel="normal" />

<permission
    android:name="com.example.myapp.WRITE_PROVIDER"
    android:protectionLevel="signature" />
```

**Path-Level Permissions**
```xml
<provider
    android:name=".MyProvider"
    android:authorities="com.example.myapp.provider"
    android:exported="true">
    <path-permission
        android:path="/sensitive/*"
        android:readPermission="com.example.myapp.READ_SENSITIVE"
        android:writePermission="com.example.myapp.WRITE_SENSITIVE" />
</provider>
```

### URI Permission Grants

**Granular Access Control**
```xml
<provider
    android:name=".MyProvider"
    android:authorities="com.example.myapp.provider"
    android:exported="true"
    android:grantUriPermissions="true">
    <grant-uri-permission android:pathPattern="/public/.*" />
</provider>
```

**Grant Temporary Access**
```kotlin
// Grant temporary URI permission
val uri = ContentUris.withAppendedId(
    Uri.parse("content://com.example.myapp.provider/items"),
    itemId
)

val intent = Intent(Intent.ACTION_VIEW).apply {
    setDataAndType(uri, "image/jpeg")
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}
startActivity(intent)
```

### SQL Injection Prevention

**Use Parameterized Queries**
```kotlin
class SecureProvider : ContentProvider() {

    override fun query(
        uri: Uri,
        projection: Array<String>?,
        selection: String?,
        selectionArgs: Array<String>?,
        sortOrder: String?
    ): Cursor? {
        // Good - parameterized query
        val db = dbHelper.readableDatabase
        return db.query(
            TABLE_NAME,
            projection,
            selection,  // Use placeholders: "user_id = ?"
            selectionArgs,  // Actual values
            null, null,
            sortOrder
        )
    }

    override fun update(
        uri: Uri,
        values: ContentValues?,
        selection: String?,
        selectionArgs: Array<String>?
    ): Int {
        // Never concatenate user input into SQL
        val db = dbHelper.writableDatabase
        return db.update(
            TABLE_NAME,
            values,
            selection,  // Use placeholders
            selectionArgs
        )
    }
}
```

**Never Do String Concatenation**
```kotlin
// BAD - SQL injection vulnerability
// val query = "SELECT * FROM users WHERE name = '$userName'"

// GOOD - parameterized query
val selection = "name = ?"
val selectionArgs = arrayOf(userName)
```

### ContentProvider Checklist

- [ ] `android:exported` explicitly declared
- [ ] Permissions required for exported providers
- [ ] Separate read/write permissions defined
- [ ] Path-level permissions for sensitive data
- [ ] URI permission grants configured appropriately
- [ ] SQL injection prevented with parameterized queries
- [ ] All input data validated
- [ ] No world-readable/writable content

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

### Intent Data Validation

**Validate Intent Extras**
```kotlin
class SecureActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Validate all data from intent
        val userId = intent.getStringExtra("user_id")
        if (!isValidUserId(userId)) {
            finish()
            return
        }

        // Process validated data
        loadUserData(userId)
    }

    private fun isValidUserId(userId: String?): Boolean {
        return userId != null &&
               userId.matches(Regex("^[0-9]+$")) &&
               userId.length <= 10
    }
}
```

### Prevent Task Hijacking

**Launch Mode Configuration**
```xml
<activity
    android:name=".SensitiveActivity"
    android:exported="false"
    android:launchMode="singleTask"
    android:taskAffinity="" />
```

**Check Calling Activity**
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // Verify caller for sensitive activities
    val callerUid = Binder.getCallingUid()
    if (!isTrustedCaller(callerUid)) {
        finish()
        return
    }
}
```

---

## Dynamic Code Loading

### Risks

**High-Risk Practice**
- Loading code outside your APK increases compromise likelihood
- Code injection risks
- Dynamically loaded code runs with same permissions as APK
- **Strongly discouraged**

### Never Load From

**Insecure Sources**
```kotlin
// BAD - Never load from these locations
// - Unsecured network locations
// - Unencrypted protocols (HTTP)
// - World-writable locations (external storage)
// - Untrusted sources

// AVOID - Dynamic code loading
// val dexFile = DexFile(externalCacheDir + "/code.dex")
```

### If Absolutely Necessary

**Verification Required**
```kotlin
// Only if absolutely required and from trusted source
fun loadCodeSecurely(codePath: File) {
    // 1. Verify signature
    if (!verifySignature(codePath)) {
        throw SecurityException("Invalid code signature")
    }

    // 2. Verify hash
    if (!verifyHash(codePath, expectedHash)) {
        throw SecurityException("Hash mismatch")
    }

    // 3. Load from internal storage only
    if (!codePath.absolutePath.startsWith(filesDir.absolutePath)) {
        throw SecurityException("Code must be in internal storage")
    }

    // Load code
    val classLoader = DexClassLoader(
        codePath.absolutePath,
        cacheDir.absolutePath,
        null,
        javaClass.classLoader
    )
}
```

### Code in APK

**Secure By Default**
- Code in APK cannot be modified by other applications
- Protected by application sandboxing
- Verified during installation

---

## Dalvik/ART Virtual Machine

### Not a Security Boundary

**Important Understanding**
- Dalvik/ART is NOT a security boundary
- OS-level sandbox is the security boundary
- Can interoperate with native code without security constraints

### Implications

**Dynamic Class Loading**
- Be careful with dynamic class loading from unverified sources
- Don't load from unsecured networks
- Don't load from external storage

---

## Native Code Security

### When to Use Native Code

**Considerations**
- **Prefer Android SDK** over NDK when possible
- Native code is more complex
- Less portable
- More error-prone
- Harder to secure

### Memory Management

**Common Vulnerabilities**
```c
// Prevent buffer overflows
void secureCopy(char *dest, const char *src, size_t destSize) {
    // Good - bounded copy
    strncpy(dest, src, destSize - 1);
    dest[destSize - 1] = '\0';

    // Bad - unbounded copy
    // strcpy(dest, src);  // Buffer overflow!
}

// Prevent use-after-free
void secureDelete(void **ptr) {
    if (ptr && *ptr) {
        free(*ptr);
        *ptr = NULL;  // Prevent use-after-free
    }
}

// Prevent off-by-one errors
void processArray(int *array, size_t size) {
    for (size_t i = 0; i < size; i++) {  // Not <=
        array[i] = 0;
    }
}
```

### Security Mitigations

**Platform Protections**
- ASLR (Address Space Layout Randomization)
- DEP (Data Execution Prevention)
- Stack canaries

**Limitations**
- These mitigate but don't prevent memory errors
- Careful coding still required

### Application Sandbox

**All Code Runs in Sandbox**
- Native applications run in sandbox
- Each app gets unique UID
- Limited permissions
- Familiarize with Android Security Overview

---

## Component Security Checklist

Before releasing your app:

- [ ] All components have `android:exported` explicitly set
- [ ] Exported services protected with permissions
- [ ] BroadcastReceivers validate all received data
- [ ] ContentProviders use parameterized queries
- [ ] ContentProvider permissions properly configured
- [ ] Activities validate intent extras
- [ ] Dynamic code loading avoided or secured
- [ ] Native code free of memory management errors
- [ ] JobScheduler used for background work
- [ ] Component permissions use signature level for same-developer apps

---

*Based on official Android documentation from developer.android.com and source.android.com*
