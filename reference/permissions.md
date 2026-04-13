# Permissions Best Practices

Guidelines for requesting, managing, and minimizing permissions in Android applications.

---

## Core Principles

### Minimize Permission Requests

**Fundamental Rule**
- Only request permissions your app actually needs
- Request permissions at appropriate times (just-in-time)
- Explain why permissions are needed

**Benefits**
- Improves user trust
- Reduces security risks
- Better user experience
- Fewer permission denials

---

## Use Intents Instead of Permissions

### Defer to Other Apps

**Strategy**
- Let apps with existing permissions handle tasks
- Reduces your app's permission footprint
- Improves security

### Common Examples

**Contacts**
```kotlin
// Good - Use intent instead of requesting permission
val intent = Intent(Intent.ACTION_INSERT).apply {
    type = ContactsContract.Contacts.CONTENT_TYPE
    putExtra(ContactsContract.Intents.Insert.NAME, "John Doe")
    putExtra(ContactsContract.Intents.Insert.EMAIL, "john@example.com")
}
startActivity(intent)

// Avoid - Requesting READ_CONTACTS and WRITE_CONTACTS
// <uses-permission android:name="android.permission.READ_CONTACTS" />
// <uses-permission android:name="android.permission.WRITE_CONTACTS" />
```

**Calendar**
```kotlin
// Good - Use intent for calendar events
val intent = Intent(Intent.ACTION_INSERT).apply {
    data = CalendarContract.Events.CONTENT_URI
    putExtra(CalendarContract.Events.TITLE, "Meeting")
    putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, startTime)
    putExtra(CalendarContract.EXTRA_EVENT_END_TIME, endTime)
}
startActivity(intent)

// Avoids - READ_CALENDAR and WRITE_CALENDAR permissions
```

**Phone Calls**
```kotlin
// Good - Use intent to make calls
val intent = Intent(Intent.ACTION_DIAL).apply {
    data = Uri.parse("tel:1234567890")
}
startActivity(intent)

// Avoids - CALL_PHONE permission
```

**Camera**
```kotlin
// Good - Use intent for simple photo capture
val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
startActivityForResult(intent, REQUEST_IMAGE_CAPTURE)

// Avoids - CAMERA permission (for simple use cases)
```

### File Operations

**No Permissions Needed**
- Apps don't need special permissions for file I/O via intents
- Use Storage Access Framework for file selection
- User grants access through picker UI

```kotlin
// Good - Use Storage Access Framework
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
    addCategory(Intent.CATEGORY_OPENABLE)
    type = "application/pdf"
}
startActivityForResult(intent, REQUEST_OPEN_DOCUMENT)
```

---

## Permission Declaration

### Manifest Declaration

**Basic Permission Request**
```xml
<manifest>
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
</manifest>
```

### Runtime Permissions (Dangerous Permissions)

**Request at Runtime**
```kotlin
// Check if permission is granted
if (ContextCompat.checkSelfPermission(
        this,
        Manifest.permission.ACCESS_FINE_LOCATION
    ) != PackageManager.PERMISSION_GRANTED
) {
    // Request permission
    ActivityCompat.requestPermissions(
        this,
        arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
        REQUEST_LOCATION_PERMISSION
    )
} else {
    // Permission already granted
    useLocation()
}

// Handle result
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<String>,
    grantResults: IntArray
) {
    when (requestCode) {
        REQUEST_LOCATION_PERMISSION -> {
            if (grantResults.isNotEmpty() &&
                grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                useLocation()
            } else {
                // Permission denied
                showPermissionDeniedMessage()
            }
        }
    }
}
```

---

## Custom Permissions

### Define Custom Permissions

**When to Use**
- Protecting your own components
- Controlling access to your app's data/services
- Communication between your apps

**Permission Declaration**
```xml
<permission
    android:name="com.example.myapp.permission.CUSTOM_PERMISSION"
    android:label="@string/permission_label"
    android:description="@string/permission_description"
    android:protectionLevel="normal" />
```

### Protection Levels

**normal**
- Low-risk permissions
- Granted automatically
- User not prompted

**dangerous**
- High-risk permissions
- Requires user approval
- Runtime permission request

**signature**
- Only granted to apps signed with same certificate
- No user confirmation
- Best for apps you control

**signatureOrSystem**
- Granted to apps signed with same cert or system apps
- Restricted use

**Example: Signature Protection**
```xml
<permission
    android:name="com.example.myapp.permission.PRIVILEGED_ACTION"
    android:protectionLevel="signature" />

<service
    android:name=".PrivilegedService"
    android:permission="com.example.myapp.permission.PRIVILEGED_ACTION"
    android:exported="true" />
```

---

## Secure Data Sharing

### URI Permissions

**Grant Temporary Access**
```kotlin
// Grant read permission
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

// Grant write permission
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION)

// Grant both
intent.addFlags(
    Intent.FLAG_GRANT_READ_URI_PERMISSION or
    Intent.FLAG_GRANT_WRITE_URI_PERMISSION
)
```

**Use Content URIs**
```kotlin
// Good - Use content:// URIs
val contentUri = FileProvider.getUriForFile(
    context,
    "com.example.myapp.fileprovider",
    file
)
intent.setDataAndType(contentUri, "application/pdf")
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

// Bad - Don't use file:// URIs
// val fileUri = Uri.fromFile(file)  // Insecure!
```

### FileProvider Configuration

**AndroidManifest.xml**
```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="com.example.myapp.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

**res/xml/file_paths.xml**
```xml
<paths>
    <files-path name="my_docs" path="documents/" />
    <cache-path name="my_cache" path="cache/" />
    <external-files-path name="my_external" path="/" />
</paths>
```

---

## Permission Best Practices

### Request Timing

**Just-In-Time Requests**
```kotlin
// Good - Request when user takes action
fun shareLocation() {
    if (hasLocationPermission()) {
        getLocation()
    } else {
        requestLocationPermission()
    }
}

// Bad - Request on app launch
// onCreate() {
//     requestAllPermissions()
// }
```

### Explain Permission Need

**Show Rationale**
```kotlin
if (ActivityCompat.shouldShowRequestPermissionRationale(
        this,
        Manifest.permission.ACCESS_FINE_LOCATION
    )) {
    // Show explanation
    showPermissionRationaleDialog(
        "Location permission is needed to show nearby places"
    ) {
        // Request permission after explanation
        requestLocationPermission()
    }
} else {
    // Request directly
    requestLocationPermission()
}
```

### Handle Denial Gracefully

**Provide Alternatives**
```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<String>,
    grantResults: IntArray
) {
    when (requestCode) {
        REQUEST_LOCATION -> {
            if (grantResults.isEmpty() ||
                grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                // Permission denied - provide alternative
                showManualLocationEntry()
                // Or disable location-dependent features
                disableLocationFeatures()
            }
        }
    }
}
```

---

## Avoid Dangerous Protection Levels

### Why Avoid

**User Confusion**
- Dangerous permissions confuse users
- Users may deny unclear permission requests
- Trust erosion

**Better Alternatives**
- Use signature protection for same-developer apps
- Use normal protection for low-risk permissions
- Use intents instead of permissions when possible

---

## Permission Alternatives

### Device Identifiers

**Avoid Device IDs**
```kotlin
// Bad - Requires READ_PHONE_STATE permission
// val deviceId = telephonyManager.deviceId

// Good - Use UUID instead
val appInstanceId = UUID.randomUUID().toString()
// Or use Firebase Installation ID
```

### Location Services

**Coarse vs Fine Location**
```kotlin
// If approximate location is sufficient
// <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

// Only request fine location if needed
// <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### Storage Access

**Scoped Storage (Android 10+)**
```kotlin
// Good - Use MediaStore for media files
val contentValues = ContentValues().apply {
    put(MediaStore.MediaColumns.DISPLAY_NAME, "photo.jpg")
    put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
}
val uri = contentResolver.insert(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    contentValues
)

// Avoids - WRITE_EXTERNAL_STORAGE permission
```

---

## Don't Leak Permission-Protected Data

### Prevent Data Leaks

**Scenario**
- Your app has permission to access sensitive data
- Other apps might not have that permission
- Don't expose that data without proper protection

**Example: Contacts**
```kotlin
// Bad - Leaking contact data
fun getContact(contactId: String): Contact {
    // Your app has READ_CONTACTS permission
    // Returns contact data to any app
    return loadContact(contactId)
}

// Good - Protect contact data
fun getContact(contactId: String): Contact {
    // Verify caller has permission
    if (checkCallingPermission(
            Manifest.permission.READ_CONTACTS
        ) != PackageManager.PERMISSION_GRANTED
    ) {
        throw SecurityException("Caller lacks READ_CONTACTS")
    }
    return loadContact(contactId)
}
```

---

## Permission Groups

### Understanding Permission Groups

**Related Permissions**
- Permissions grouped by functionality
- Granting one may grant others in same group
- Users see group in permission prompt

**Example Groups**
- CONTACTS: READ_CONTACTS, WRITE_CONTACTS
- LOCATION: ACCESS_FINE_LOCATION, ACCESS_COARSE_LOCATION
- STORAGE: READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE

**Implication**
```kotlin
// After user grants ACCESS_FINE_LOCATION
// ACCESS_COARSE_LOCATION is also granted
// Don't rely on this behavior - always check permissions
```

---

## Special Permissions

### System Alert Window

**Use Case**
- Overlay windows on top of other apps
- Chat heads, floating widgets

**Request**
```kotlin
if (!Settings.canDrawOverlays(this)) {
    val intent = Intent(
        Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
        Uri.parse("package:$packageName")
    )
    startActivityForResult(intent, REQUEST_OVERLAY_PERMISSION)
}
```

### Write Settings

**Use Case**
- Modify system settings
- Requires special permission

**Request**
```kotlin
if (!Settings.System.canWrite(this)) {
    val intent = Intent(
        Settings.ACTION_MANAGE_WRITE_SETTINGS,
        Uri.parse("package:$packageName")
    )
    startActivity(intent)
}
```

---

## Permissions Checklist

Before releasing your app:

- [ ] Only necessary permissions requested
- [ ] Intents used instead of permissions where possible
- [ ] Runtime permissions requested at appropriate times
- [ ] Permission rationale shown to users
- [ ] Permission denials handled gracefully
- [ ] Custom permissions use appropriate protection levels
- [ ] Signature protection used for same-developer apps
- [ ] File sharing uses FileProvider and content:// URIs
- [ ] Permission-protected data not leaked
- [ ] Device identifiers avoided (UUID used instead)
- [ ] Scoped storage used (Android 10+)
- [ ] Special permissions properly requested

---

*Based on official Android documentation from developer.android.com and source.android.com*
