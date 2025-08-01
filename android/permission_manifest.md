# Permission Management, Manifest & build.gradle - Complete Guide

## Table of Contents
1. [Permission Management](#permission-management)
2. [Android Manifest](#android-manifest)
3. [build.gradle Configuration](#buildgradle-configuration)
4. [Advanced Configurations](#advanced-configurations)

---

## Permission Management

### Runtime Permissions (API 23+)

#### Basic Permission Request
```kotlin
class MainActivity : ComponentActivity() {
    
    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            // Permission granted
            handlePermissionGranted()
        } else {
            // Permission denied
            handlePermissionDenied()
        }
    }
    
    private fun checkAndRequestPermission() {
        when {
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) == PackageManager.PERMISSION_GRANTED -> {
                // Permission already granted
                handlePermissionGranted()
            }
            
            shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
                // Show rationale dialog
                showPermissionRationale()
            }
            
            else -> {
                // Request permission
                requestPermissionLauncher.launch(Manifest.permission.CAMERA)
            }
        }
    }
    
    private fun showPermissionRationale() {
        AlertDialog.Builder(this)
            .setTitle("Camera Permission Required")
            .setMessage("This app needs camera access to take photos")
            .setPositiveButton("Grant") { _, _ ->
                requestPermissionLauncher.launch(Manifest.permission.CAMERA)
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
}
```

#### Multiple Permissions
```kotlin
class PermissionManager : ComponentActivity() {
    
    private val requestMultiplePermissions = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        permissions.entries.forEach { entry ->
            when (entry.key) {
                Manifest.permission.CAMERA -> {
                    if (entry.value) {
                        handleCameraPermissionGranted()
                    } else {
                        handleCameraPermissionDenied()
                    }
                }
                Manifest.permission.RECORD_AUDIO -> {
                    if (entry.value) {
                        handleAudioPermissionGranted()
                    } else {
                        handleAudioPermissionDenied()
                    }
                }
                Manifest.permission.ACCESS_FINE_LOCATION -> {
                    if (entry.value) {
                        handleLocationPermissionGranted()
                    } else {
                        handleLocationPermissionDenied()
                    }
                }
            }
        }
    }
    
    private fun requestMultiplePermissions() {
        val permissions = arrayOf(
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO,
            Manifest.permission.ACCESS_FINE_LOCATION
        )
        
        requestMultiplePermissions.launch(permissions)
    }
    
    private fun checkAllPermissions(): Boolean {
        val permissions = arrayOf(
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO,
            Manifest.permission.ACCESS_FINE_LOCATION
        )
        
        return permissions.all { permission ->
            ContextCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_GRANTED
        }
    }
}
```

#### Permission Helper Class
```kotlin
class PermissionHelper(private val activity: Activity) {
    
    companion object {
        const val CAMERA_PERMISSION_CODE = 100
        const val STORAGE_PERMISSION_CODE = 101
        const val LOCATION_PERMISSION_CODE = 102
    }
    
    fun hasPermission(permission: String): Boolean {
        return ContextCompat.checkSelfPermission(
            activity,
            permission
        ) == PackageManager.PERMISSION_GRANTED
    }
    
    fun hasPermissions(permissions: Array<String>): Boolean {
        return permissions.all { hasPermission(it) }
    }
    
    fun requestPermission(
        permission: String,
        requestCode: Int,
        showRationale: Boolean = true
    ) {
        if (hasPermission(permission)) {
            return
        }
        
        if (showRationale && activity.shouldShowRequestPermissionRationale(permission)) {
            showPermissionRationale(permission, requestCode)
        } else {
            ActivityCompat.requestPermissions(
                activity,
                arrayOf(permission),
                requestCode
            )
        }
    }
    
    fun requestPermissions(
        permissions: Array<String>,
        requestCode: Int
    ) {
        val permissionsToRequest = permissions.filter { !hasPermission(it) }
        
        if (permissionsToRequest.isNotEmpty()) {
            ActivityCompat.requestPermissions(
                activity,
                permissionsToRequest.toTypedArray(),
                requestCode
            )
        }
    }
    
    private fun showPermissionRationale(permission: String, requestCode: Int) {
        val message = when (permission) {
            Manifest.permission.CAMERA -> "Camera access is needed to take photos"
            Manifest.permission.READ_EXTERNAL_STORAGE -> "Storage access is needed to save files"
            Manifest.permission.ACCESS_FINE_LOCATION -> "Location access is needed for location features"
            else -> "This permission is required for the app to function properly"
        }
        
        AlertDialog.Builder(activity)
            .setTitle("Permission Required")
            .setMessage(message)
            .setPositiveButton("Grant") { _, _ ->
                ActivityCompat.requestPermissions(
                    activity,
                    arrayOf(permission),
                    requestCode
                )
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    fun openAppSettings() {
        val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
        val uri = Uri.fromParts("package", activity.packageName, null)
        intent.data = uri
        activity.startActivity(intent)
    }
}
```

#### Jetpack Compose Permission Handling
```kotlin
@Composable
fun CameraPermissionHandler(
    onPermissionGranted: () -> Unit,
    onPermissionDenied: () -> Unit
) {
    val context = LocalContext.current
    var showRationale by remember { mutableStateOf(false) }
    
    val permissionLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            onPermissionGranted()
        } else {
            onPermissionDenied()
        }
    }
    
    val permission = Manifest.permission.CAMERA
    
    LaunchedEffect(Unit) {
        when {
            ContextCompat.checkSelfPermission(
                context,
                permission
            ) == PackageManager.PERMISSION_GRANTED -> {
                onPermissionGranted()
            }
            
            (context as? Activity)?.shouldShowRequestPermissionRationale(permission) == true -> {
                showRationale = true
            }
            
            else -> {
                permissionLauncher.launch(permission)
            }
        }
    }
    
    if (showRationale) {
        AlertDialog(
            onDismissRequest = { showRationale = false },
            title = { Text("Camera Permission Required") },
            text = { Text("This app needs camera access to take photos") },
            confirmButton = {
                TextButton(
                    onClick = {
                        showRationale = false
                        permissionLauncher.launch(permission)
                    }
                ) {
                    Text("Grant")
                }
            },
            dismissButton = {
                TextButton(onClick = { showRationale = false }) {
                    Text("Cancel")
                }
            }
        )
    }
}

// Usage in Compose
@Composable
fun CameraScreen() {
    var hasPermission by remember { mutableStateOf(false) }
    
    if (hasPermission) {
        CameraPreview()
    } else {
        CameraPermissionHandler(
            onPermissionGranted = { hasPermission = true },
            onPermissionDenied = { 
                // Handle permission denied
            }
        )
    }
}
```

### Common Permission Groups

#### Location Permissions
```kotlin
// Manifest permissions
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />

// Runtime request
private fun requestLocationPermission() {
    val permissions = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        arrayOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.ACCESS_COARSE_LOCATION,
            Manifest.permission.ACCESS_BACKGROUND_LOCATION
        )
    } else {
        arrayOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.ACCESS_COARSE_LOCATION
        )
    }
    
    requestMultiplePermissions.launch(permissions)
}
```

#### Storage Permissions
```kotlin
// For API 33+ (Android 13)
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

// For older versions
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

// Runtime handling
private fun getStoragePermissions(): Array<String> {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        arrayOf(
            Manifest.permission.READ_MEDIA_IMAGES,
            Manifest.permission.READ_MEDIA_VIDEO
        )
    } else {
        arrayOf(
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE
        )
    }
}
```

#### Camera and Audio Permissions
```kotlin
// Manifest
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-feature android:name="android.hardware.camera" android:required="true" />

// Runtime
private fun requestCameraAndAudio() {
    val permissions = arrayOf(
        Manifest.permission.CAMERA,
        Manifest.permission.RECORD_AUDIO
    )
    requestMultiplePermissions.launch(permissions)
}
```

---

## Android Manifest

### Complete Manifest Structure
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.myapp">
    
    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    
    <!-- Features -->
    <uses-feature
        android:name="android.hardware.camera"
        android:required="true" />
    <uses-feature
        android:name="android.hardware.location"
        android:required="false" />
    <uses-feature
        android:name="android.hardware.telephony"
        android:required="false" />
    
    <!-- Application -->
    <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApp"
        android:hardwareAccelerated="true"
        android:largeHeap="true"
        android:usesCleartextTraffic="false"
        tools:targetApi="31">
        
        <!-- Main Activity -->
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.MyApp"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="adjustResize|stateHidden"
            android:launchMode="singleTop">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            
            <!-- Deep linking -->
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="https"
                    android:host="myapp.com" />
            </intent-filter>
        </activity>
        
        <!-- Other Activities -->
        <activity
            android:name=".DetailActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity">
            <meta-data
                android:name="android.support.PARENT_ACTIVITY"
                android:value=".MainActivity" />
        </activity>
        
        <!-- Services -->
        <service
            android:name=".services.BackgroundService"
            android:enabled="true"
            android:exported="false" />
        
        <service
            android:name=".services.ForegroundService"
            android:enabled="true"
            android:exported="false"
            android:foregroundServiceType="location|camera" />
        
        <!-- Broadcast Receivers -->
        <receiver
            android:name=".receivers.NetworkReceiver"
            android:enabled="true"
            android:exported="false">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
        </receiver>
        
        <!-- Content Providers -->
        <provider
            android:name=".providers.MyContentProvider"
            android:authorities="com.example.myapp.provider"
            android:enabled="true"
            android:exported="false" />
        
        <!-- File Provider -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        
        <!-- Firebase Messaging -->
        <service
            android:name=".services.MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        
        <!-- Meta-data -->
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="@string/google_maps_key" />
        
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/ic_notification" />
        
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_color"
            android:resource="@color/notification_color" />
        
    </application>
    
    <!-- Queries (for API 30+) -->
    <queries>
        <intent>
            <action android:name="android.intent.action.SEND" />
            <data android:mimeType="image/*" />
        </intent>
        <package android:name="com.whatsapp" />
        <provider android:authorities="com.facebook.katana.provider.PlatformProvider" />
    </queries>
    
</manifest>
```

### Activity Configuration Options
```xml
<!-- Launch Modes -->
<activity
    android:name=".SingleTopActivity"
    android:launchMode="singleTop" />

<activity
    android:name=".SingleTaskActivity"
    android:launchMode="singleTask" />

<activity
    android:name=".SingleInstanceActivity"
    android:launchMode="singleInstance" />

<!-- Screen Orientations -->
<activity
    android:name=".PortraitActivity"
    android:screenOrientation="portrait" />

<activity
    android:name=".LandscapeActivity"
    android:screenOrientation="landscape" />

<activity
    android:name=".SensorActivity"
    android:screenOrientation="sensor" />

<!-- Window Soft Input Modes -->
<activity
    android:name=".FormActivity"
    android:windowSoftInputMode="adjustResize|stateVisible" />

<activity
    android:name=".ChatActivity"
    android:windowSoftInputMode="adjustPan|stateAlwaysVisible" />
```

### Intent Filters and Deep Linking
```xml
<!-- Custom URL Scheme -->
<activity android:name=".DeepLinkActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
</activity>

<!-- HTTP/HTTPS Deep Links -->
<activity android:name=".WebLinkActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https"
            android:host="myapp.com"
            android:pathPrefix="/product" />
    </intent-filter>
</activity>

<!-- File Associations -->
<activity android:name=".FileViewerActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="application/pdf" />
    </intent-filter>
</activity>

<!-- Share Intent -->
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

### Security Configuration
```xml
<!-- Network Security Config -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false">
    
    <!-- ... -->
</application>

<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">secure.example.com</domain>
    </domain-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">insecure.example.com</domain>
    </domain-config>
</network-security-config>

<!-- Backup Rules -->
<!-- res/xml/backup_rules.xml -->
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <exclude domain="sharedpref" path="sensitive_prefs.xml" />
    <exclude domain="database" path="secret.db" />
</full-backup-content>

<!-- res/xml/data_extraction_rules.xml -->
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup>
        <exclude domain="sharedpref" path="device_specific_prefs.xml" />
    </cloud-backup>
    <device-transfer>
        <exclude domain="sharedpref" path="analytics.xml" />
    </device-transfer>
</data-extraction-rules>
```

---

## build.gradle Configuration

### Project-level build.gradle (Kotlin DSL)
```kotlin
// build.gradle.kts (Project)
buildscript {
    ext {
        compose_bom_version = '2023.10.01'
        kotlin_version = '1.9.10'
        agp_version = '8.1.2'
        hilt_version = '2.48'
    }
}

plugins {
    id("com.android.application") version "8.1.2" apply false
    id("com.android.library") version "8.1.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.10" apply false
    id("com.google.dagger.hilt.android") version "2.48" apply false
    id("com.google.gms.google-services") version "4.4.0" apply false
    id("com.google.firebase.crashlytics") version "2.9.9" apply false
    id("androidx.navigation.safeargs.kotlin") version "2.7.5" apply false
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

### Module-level build.gradle (Kotlin DSL)
```kotlin
// build.gradle.kts (Module: app)
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("kotlin-kapt")
    id("dagger.hilt.android.plugin")
    id("kotlin-parcelize")
    id("androidx.navigation.safeargs.kotlin")
    id("com.google.gms.google-services")
    id("com.google.firebase.crashlytics")
}

android {
    namespace = "com.example.myapp"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
        
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        
        vectorDrawables {
            useSupportLibrary = true
        }
        
        // Build Config Fields
        buildConfigField("String", "API_BASE_URL", "\"https://api.example.com/\"")
        buildConfigField("boolean", "DEBUG_MODE", "true")
        
        // ProGuard files
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
    
    buildTypes {
        debug {
            isMinifyEnabled = false
            isDebuggable = true
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
            
            buildConfigField("String", "API_BASE_URL", "\"https://dev-api.example.com/\"")
            
            // Firebase Crashlytics
            firebaseCrashlytics {
                mappingFileUploadEnabled = false
            }
        }
        
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            isDebuggable = false
            
            buildConfigField("String", "API_BASE_URL", "\"https://api.example.com/\"")
            
            firebaseCrashlytics {
                mappingFileUploadEnabled = true
            }
            
            signingConfig = signingConfigs.getByName("release")
        }
        
        create("staging") {
            initWith(buildTypes.getByName("debug"))
            isMinifyEnabled = true
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"
            
            buildConfigField("String", "API_BASE_URL", "\"https://staging-api.example.com/\"")
        }
    }
    
    flavorDimensions += listOf("version", "environment")
    
    productFlavors {
        create("free") {
            dimension = "version"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
            
            buildConfigField("boolean", "IS_PREMIUM", "false")
            resValue("string", "app_name", "MyApp Free")
        }
        
        create("premium") {
            dimension = "version"
            applicationIdSuffix = ".premium"
            versionNameSuffix = "-premium"
            
            buildConfigField("boolean", "IS_PREMIUM", "true")
            resValue("string", "app_name", "MyApp Premium")
        }
        
        create("development") {
            dimension = "environment"
            buildConfigField("String", "SERVER_URL", "\"https://dev.example.com\"")
        }
        
        create("production") {
            dimension = "environment"
            buildConfigField("String", "SERVER_URL", "\"https://prod.example.com\"")
        }
    }
    
    signingConfigs {
        create("release") {
            storeFile = file("../keystore/release.keystore")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
        
        // Enable core library desugaring
        isCoreLibraryDesugaringEnabled = true
    }
    
    kotlinOptions {
        jvmTarget = "1.8"
        
        freeCompilerArgs += listOf(
            "-opt-in=kotlin.RequiresOptIn",
            "-opt-in=androidx.compose.material3.ExperimentalMaterial3Api",
            "-opt-in=androidx.compose.foundation.ExperimentalFoundationApi"
        )
    }
    
    buildFeatures {
        compose = true
        buildConfig = true
        viewBinding = true
        dataBinding = true
    }
    
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.4"
    }
    
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
            excludes += "/META-INF/gradle/incremental.annotation.processors"
        }
    }
    
    testOptions {
        unitTests {
            isIncludeAndroidResources = true
            isReturnDefaultValues = true
        }
    }
    
    // Lint options
    lint {
        disable += setOf("MissingTranslation", "ExtraTranslation")
        warningsAsErrors = false
        abortOnError = false
    }
    
    // Bundle configuration
    bundle {
        language {
            enableSplit = false
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}

dependencies {
    // Core Android
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.1")
    
    // Compose BOM
    implementation(platform("androidx.compose:compose-bom:2023.10.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    
    // Navigation
    implementation("androidx.navigation:navigation-compose:2.7.5")
    
    // ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")
    
    // Hilt Dependency Injection
    implementation("com.google.dagger:hilt-android:2.48")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
    kapt("com.google.dagger:hilt-compiler:2.48")
    
    // Networking
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    
    // Image Loading
    implementation("io.coil-kt:coil-compose:2.5.0")
    
    // Room Database
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    kapt("androidx.room:room-compiler:2.6.1")
    
    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.0.0")
    
    // Firebase
    implementation(platform("com.google.firebase:firebase-bom:32.6.0"))
    implementation("com.google.firebase:firebase-analytics-ktx")
    implementation("com.google.firebase:firebase-crashlytics-ktx")
    implementation("com.google.firebase:firebase-messaging-ktx")
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.firebase:firebase-firestore-ktx")
    
    // Google Play Services
    implementation("com.google.android.gms:play-services-auth:20.7.0")
    implementation("com.google.android.gms:play-services-maps:18.2.0")
    implementation("com.google.android.gms:play-services-location:21.0.1")
    
    // Permissions
    implementation("com.google.accompanist:accompanist-permissions:0.32.0")
    
    // Core Library Desugaring
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
    testImplementation("androidx.arch.core:core-testing:2.2.0")
    testImplementation("com.google.truth:truth:1.1.4")
    testImplementation("io.mockk:mockk:1.13.8")
    testImplementation("androidx.room:room-testing:2.6.1")
    
    // Android Testing
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation(platform("androidx.compose:compose-bom:2023.10.01"))
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
    androidTestImplementation("androidx.navigation:navigation-testing:2.7.5")
    androidTestImplementation("com.google.dagger:hilt-android-testing:2.48")
    kaptAndroidTest("com.google.dagger:hilt-compiler:2.48")
    
    // Debug Dependencies
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
    debugImplementation("com.squareup.leakcanary:leakcanary-android:2.12")
}

// Hilt Kapt Configuration
kapt {
    correctErrorTypes = true
}
```

### Build Variants and Flavors Configuration
```kotlin
// Advanced Build Configuration
android {
    // ... other configurations
    
    // Custom source sets
    sourceSets {
        getByName("main") {
            java.srcDirs("src/main/kotlin")
        }
        getByName("debug") {
            res.srcDirs("src/debug/res")
        }
        getByName("release") {
            res.srcDirs("src/release/res")
        }
    }
    
    // Exclude specific files from build
    packagingOptions {
        resources.excludes.addAll(
            listOf(
                "META-INF/DEPENDENCIES",
                "META-INF/LICENSE",
                "META-INF/LICENSE.txt",
                "META-INF/NOTICE",
                "META-INF/NOTICE.txt"
            )
        )
    }
    
    // Custom APK naming
    applicationVariants.all {
        val variant = this
        variant.outputs
            .map { it as com.android.build.gradle.internal.api.BaseVariantOutputImpl }
            .forEach { output ->
                val outputFileName = "MyApp-${variant.baseName}-${variant.versionName}.apk"
                output.outputFileName = outputFileName
            }
    }
}
```

### ProGuard Configuration
```kotlin
// proguard-rules.pro
-dontwarn **
-ignorewarnings

# Keep application class
-keep public class * extends android.app.Application

# Keep all classes with native methods
-keepclasseswithmembernames class * {
    native <methods>;
}

# Keep all enums
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Keep Parcelable implementations
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# Retrofit
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations
-keepattributes AnnotationDefault
-keepclassmembers,allowshrinking,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
-dontwarn javax.annotation.**
-dontwarn kotlin.Unit
-dontwarn retrofit2.KotlinExtensions
-dontwarn retrofit2.KotlinExtensions$*
-if interface * { @retrofit2.http.* <methods>; }
-keep,allowobfuscation interface <1>
-if interface * { @retrofit2.http.* <methods>; }
-keep,allowobfuscation interface * extends <1>
-keep,allowobfuscation,allowshrinking class kotlin.coroutines.Continuation
-if interface * { @retrofit2.http.* public *** *(...); }
-keep,allowoptimization,allowshrinking,allowobfuscation class <3>

# Gson
-keepattributes Signature
-keepattributes *Annotation*
-dontwarn sun.misc.**
-keep class com.google.gson.** { *; }
-keep class * extends com.google.gson.TypeAdapter
-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer
-keepclassmembers,allowobfuscation class * {
    @com.google.gson.annotations.SerializedName <fields>;
}

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# Hilt
-dontwarn dagger.hilt.**
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }
-keep class * extends dagger.hilt.android.HiltAndroidApp
-keepclasseswithmembers class * {
    @dagger.hilt.android.AndroidEntryPoint <methods>;
}

# Firebase
-keep class com.google.firebase.** { *; }
-keep class com.google.android.gms.** { *; }
-dontwarn com.google.firebase.**
-dontwarn com.google.android.gms.**

# Your data classes (replace with your package)
-keep class com.example.myapp.data.** { *; }
-keep class com.example.myapp.network.** { *; }
```

---

## Advanced Configurations

### Environment-based Configuration
```kotlin
// build.gradle.kts
android {
    // ... other configurations
    
    buildTypes {
        debug {
            buildConfigField("String", "BASE_URL", "\"https://dev-api.example.com/\"")
            buildConfigField("String", "DATABASE_NAME", "\"app_debug.db\"")
            buildConfigField("boolean", "ENABLE_LOGGING", "true")
            resValue("string", "app_name", "MyApp Debug")
            manifestPlaceholders["crashlyticsEnabled"] = false
        }
        
        release {
            buildConfigField("String", "BASE_URL", "\"https://api.example.com/\"")
            buildConfigField("String", "DATABASE_NAME", "\"app.db\"")
            buildConfigField("boolean", "ENABLE_LOGGING", "false")
            resValue("string", "app_name", "MyApp")
            manifestPlaceholders["crashlyticsEnabled"] = true
        }
    }
}

// Usage in code
class ApiConfig {
    companion object {
        const val BASE_URL = BuildConfig.BASE_URL
        const val ENABLE_LOGGING = BuildConfig.ENABLE_LOGGING
    }
}
```

### Signing Configuration
```kotlin
// build.gradle.kts
android {
    signingConfigs {
        create("release") {
            storeFile = file("../keystore/release.keystore")
            storePassword = System.getenv("KEYSTORE_PASSWORD") ?: project.findProperty("KEYSTORE_PASSWORD") as String?
            keyAlias = System.getenv("KEY_ALIAS") ?: project.findProperty("KEY_ALIAS") as String?
            keyPassword = System.getenv("KEY_PASSWORD") ?: project.findProperty("KEY_PASSWORD") as String?
        }
        
        create("debug") {
            storeFile = file("debug.keystore")
            storePassword = "android"
            keyAlias = "androiddebugkey"
            keyPassword = "android"
        }
    }
    
    buildTypes {
        debug {
            signingConfig = signingConfigs.getByName("debug")
        }
        
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}

// gradle.properties (not committed to version control)
KEYSTORE_PASSWORD=your_keystore_password
KEY_ALIAS=your_key_alias
KEY_PASSWORD=your_key_password
```

### Multi-Module Configuration
```kotlin
// settings.gradle.kts
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}

rootProject.name = "MyApp"
include(":app")
include(":core:common")
include(":core:network")
include(":core:database")
include(":feature:home")
include(":feature:profile")

// buildSrc/src/main/kotlin/Dependencies.kt
object Dependencies {
    const val compileSdk = 34
    const val minSdk = 24
    const val targetSdk = 34
    
    object Versions {
        const val kotlin = "1.9.10"
        const val compose = "2023.10.01"
        const val hilt = "2.48"
        const val retrofit = "2.9.0"
        const val room = "2.6.1"
    }
    
    object AndroidX {
        const val core = "androidx.core:core-ktx:1.12.0"
        const val lifecycle = "androidx.lifecycle:lifecycle-runtime-ktx:2.7.0"
        const val activity = "androidx.activity:activity-compose:1.8.1"
        const val navigation = "androidx.navigation:navigation-compose:2.7.5"
    }
    
    object Compose {
        const val bom = "androidx.compose:compose-bom:${Versions.compose}"
        const val ui = "androidx.compose.ui:ui"
        const val material3 = "androidx.compose.material3:material3"
        const val tooling = "androidx.compose.ui:ui-tooling-preview"
    }
    
    object Hilt {
        const val android = "com.google.dagger:hilt-android:${Versions.hilt}"
        const val compiler = "com.google.dagger:hilt-compiler:${Versions.hilt}"
        const val navigation = "androidx.hilt:hilt-navigation-compose:1.1.0"
    }
}

// Usage in module build.gradle.kts
dependencies {
    implementation(Dependencies.AndroidX.core)
    implementation(Dependencies.AndroidX.lifecycle)
}
```

### Performance Optimization
```kotlin
// build.gradle.kts
android {
    // ... other configurations
    
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    
    kotlinOptions {
        jvmTarget = "17"
        
        freeCompilerArgs += listOf(
            "-opt-in=kotlin.RequiresOptIn",
            "-Xjvm-default=all",
            "-Xuse-ir"
        )
    }
    
    // Enable R8 full mode
    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    
    // Optimize for size
    packagingOptions {
        resources.excludes.addAll(
            listOf(
                "DebugProbesKt.bin",
                "kotlin-tooling-metadata.json",
                "META-INF/com.android.tools/**",
                "META-INF/proguard/**"
            )
        )
    }
    
    // Enable build cache
    buildCache {
        local {
            isEnabled = true
        }
    }
}

// gradle.properties
org.gradle.jvmargs=-Xmx4g -Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.configureondemand=true
android.useAndroidX=true
android.enableJetifier=true
android.nonTransitiveRClass=true
android.nonFinalResIds=true
```

### Gradle Task Customization
```kotlin
// Custom tasks in build.gradle.kts
tasks.register("copyApkToDesktop", Copy::class) {
    dependsOn("assembleRelease")
    from("build/outputs/apk/release/")
    into("${System.getProperty("user.home")}/Desktop/")
    include("*.apk")
}

tasks.register("generateBuildInfo") {
    doLast {
        val buildInfoFile = File("$buildDir/generated/buildInfo.txt")
        buildInfoFile.parentFile.mkdirs()
        buildInfoFile.writeText("""
            Build Time: ${java.time.Instant.now()}
            Version: ${android.defaultConfig.versionName}
            Build Type: ${gradle.startParameter.taskNames}
            Git Commit: ${getGitCommitHash()}
        """.trimIndent())
    }
}

fun getGitCommitHash(): String {
    return try {
        val stdout = ByteArrayOutputStream()
        exec {
            commandLine("git", "rev-parse", "--short", "HEAD")
            standardOutput = stdout
        }
        stdout.toString().trim()
    } catch (e: Exception) {
        "unknown"
    }
}

// Add build info to debug builds
android.applicationVariants.all {
    if (buildType.isDebuggable) {
        preBuildProvider.get().dependsOn("generateBuildInfo")
    }
}
```

### Dependency Version Management
```kotlin
// Version Catalog (gradle/libs.versions.toml)
[versions]
kotlin = "1.9.10"
compose-bom = "2023.10.01"
hilt = "2.48"
retrofit = "2.9.0"
room = "2.6.1"

[libraries]
androidx-core = { group = "androidx.core", name = "core-ktx", version = "1.12.0" }
androidx-lifecycle = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version = "2.7.0" }
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }

[bundles]
compose = ["compose-ui", "compose-material3"]
hilt = ["hilt-android"]

[plugins]
android-application = { id = "com.android.application", version = "8.1.2" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }

// Usage in build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.hilt)
}

dependencies {
    implementation(libs.androidx.core)
    implementation(libs.androidx.lifecycle)
    implementation(platform(libs.compose.bom))
    implementation(libs.bundles.compose)
    implementation(libs.bundles.hilt)
    kapt(libs.hilt.compiler)
}
```

### Best Practices Summary

#### Permission Management
- Always check for permissions before using restricted APIs
- Provide clear rationale for permission requests
- Handle permission denial gracefully
- Use the latest permission APIs (ActivityResult contracts)
- Consider using libraries like Accompanist Permissions for Compose

#### Manifest Configuration
- Use `android:exported` explicitly for API 31+
- Implement proper intent filters for deep linking
- Configure network security properly
- Use file providers for sharing files
- Implement proper backup rules

#### build.gradle Optimization
- Use Kotlin DSL for better IDE support
- Implement proper build variants for different environments
- Configure ProGuard/R8 for release builds
- Use version catalogs for dependency management
- Enable build optimizations (parallel builds, caching)
- Separate concerns with multi-module architecture

#### Security
- Never commit signing keys or sensitive data
- Use environment variables for sensitive configuration
- Implement proper network security configuration
- Use ProGuard/R8 for code obfuscation
- Validate all inputs and sanitize data

This comprehensive guide covers all essential aspects of Permission Management, Android Manifest configuration, and build.gradle setup for modern Android development.