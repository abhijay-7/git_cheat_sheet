# Android Studio & Platform Tools Cheat Sheet

## ADB (Android Debug Bridge) Commands

### Device Management
- `adb devices` - List all connected devices and emulators
- `adb connect <ip:port>` - Connect to device over network (wireless debugging)
- `adb disconnect` - Disconnect from all TCP/IP devices
- `adb kill-server` - Kill the ADB server process
- `adb start-server` - Start the ADB server process
- `adb reboot` - Reboot the connected device
- `adb reboot bootloader` - Reboot device into bootloader mode
- `adb reboot recovery` - Reboot device into recovery mode

### App Installation & Management
- `adb install <apk_file>` - Install APK to device
- `adb install -r <apk_file>` - Reinstall existing app (keep data)
- `adb uninstall <package_name>` - Uninstall app from device
- `adb shell pm list packages` - List all installed packages
- `adb shell pm list packages -3` - List only third-party packages
- `adb shell pm clear <package_name>` - Clear app data and cache
- `adb shell pm disable <package_name>` - Disable app without uninstalling

### File Operations
- `adb push <local_path> <remote_path>` - Copy file from computer to device
- `adb pull <remote_path> <local_path>` - Copy file from device to computer
- `adb shell ls` - List files in device directory
- `adb shell cd <directory>` - Change directory on device
- `adb shell mkdir <directory>` - Create directory on device
- `adb shell rm <file>` - Delete file on device

### Logging & Debugging
- `adb logcat` - View device logs in real-time
- `adb logcat -c` - Clear logcat buffer
- `adb logcat *:E` - Show only error logs
- `adb logcat -s <tag>` - Filter logs by specific tag
- `adb shell dumpsys` - Dump system services information
- `adb shell dumpsys battery` - Show battery information
- `adb shell dumpsys meminfo <package>` - Show memory usage for app

### Screen & Input
- `adb shell screencap <path>` - Take screenshot and save to device
- `adb shell screenrecord <path>` - Record screen video
- `adb shell input text "text"` - Input text into focused field
- `adb shell input keyevent <keycode>` - Send key event (e.g., KEYCODE_HOME)
- `adb shell input tap <x> <y>` - Tap at specific coordinates
- `adb shell input swipe <x1> <y1> <x2> <y2>` - Swipe gesture

## Fastboot Commands

### Device Information
- `fastboot devices` - List devices in fastboot mode
- `fastboot getvar all` - Display all device variables
- `fastboot getvar version` - Show fastboot version
- `fastboot getvar product` - Show device product name

### Flashing Operations
- `fastboot flash <partition> <image>` - Flash image to specific partition
- `fastboot flash boot <boot.img>` - Flash boot image
- `fastboot flash recovery <recovery.img>` - Flash recovery image
- `fastboot flash system <system.img>` - Flash system image
- `fastboot flashall` - Flash all partitions from current directory

### Device Control
- `fastboot reboot` - Reboot device normally
- `fastboot reboot-bootloader` - Reboot back to bootloader
- `fastboot continue` - Continue normal boot process
- `fastboot oem unlock` - Unlock bootloader (device-specific)
- `fastboot oem lock` - Lock bootloader (device-specific)

### Temporary Operations
- `fastboot boot <boot.img>` - Boot from image without flashing
- `fastboot erase <partition>` - Erase specific partition
- `fastboot format <partition>` - Format specific partition

## Android Studio Shortcuts

### Navigation
- `Ctrl + N` (Windows/Linux) / `Cmd + O` (Mac) - Open class
- `Ctrl + Shift + N` / `Cmd + Shift + O` - Open file
- `Ctrl + Alt + Shift + N` / `Cmd + Alt + O` - Open symbol
- `Ctrl + E` / `Cmd + E` - Recent files
- `Ctrl + Shift + E` / `Cmd + Shift + E` - Recent locations
- `Alt + F1` - Select target (navigate to file in project view)
- `Ctrl + B` / `Cmd + B` - Go to declaration
- `Ctrl + Alt + B` / `Cmd + Alt + B` - Go to implementation

### Code Editing
- `Ctrl + Space` - Basic code completion
- `Ctrl + Shift + Space` / `Cmd + Shift + Space` - Smart code completion
- `Ctrl + /` / `Cmd + /` - Comment/uncomment line
- `Ctrl + Shift + /` / `Cmd + Shift + /` - Block comment
- `Ctrl + D` / `Cmd + D` - Duplicate line
- `Ctrl + Y` / `Cmd + Delete` - Delete line
- `Alt + Enter` - Show intention actions and quick fixes
- `Ctrl + Alt + L` / `Cmd + Alt + L` - Reformat code

### Refactoring
- `Shift + F6` - Rename
- `Ctrl + Alt + M` / `Cmd + Alt + M` - Extract method
- `Ctrl + Alt + V` / `Cmd + Alt + V` - Extract variable
- `Ctrl + Alt + F` / `Cmd + Alt + F` - Extract field
- `Ctrl + Alt + C` / `Cmd + Alt + C` - Extract constant

### Build & Run
- `Ctrl + F9` / `Cmd + F9` - Make project
- `Ctrl + Shift + F9` / `Cmd + Shift + F9` - Compile selected file
- `Shift + F10` / `Ctrl + R` - Run app
- `Shift + F9` / `Ctrl + D` - Debug app
- `Ctrl + F2` / `Cmd + F2` - Stop app

### Search & Replace
- `Ctrl + F` / `Cmd + F` - Find in current file
- `Ctrl + R` / `Cmd + R` - Replace in current file
- `Ctrl + Shift + F` / `Cmd + Shift + F` - Find in path (global search)
- `Ctrl + Shift + R` / `Cmd + Shift + R` - Replace in path
- `Shift + Shift` - Search everywhere

## Gradle Commands

### Build Operations
- `./gradlew build` - Build the project
- `./gradlew clean` - Clean build artifacts
- `./gradlew assembleDebug` - Build debug APK
- `./gradlew assembleRelease` - Build release APK
- `./gradlew installDebug` - Build and install debug APK
- `./gradlew installRelease` - Build and install release APK

### Testing
- `./gradlew test` - Run unit tests
- `./gradlew connectedAndroidTest` - Run instrumented tests
- `./gradlew testDebugUnitTest` - Run debug unit tests
- `./gradlew jacocoTestReport` - Generate test coverage report

### Project Information
- `./gradlew projects` - List all projects in build
- `./gradlew tasks` - List available tasks
- `./gradlew dependencies` - Show project dependencies
- `./gradlew properties` - Show project properties

### Performance & Debugging
- `./gradlew build --profile` - Build with performance profiling
- `./gradlew build --scan` - Build with build scan (requires gradle.com account)
- `./gradlew build --debug` - Build with debug logging
- `./gradlew build --offline` - Build in offline mode

## Emulator Commands

### Starting Emulator
- `emulator -list-avds` - List available virtual devices
- `emulator @<avd_name>` - Start specific AVD
- `emulator @<avd_name> -no-audio` - Start AVD without audio
- `emulator @<avd_name> -netdelay none -netspeed full` - Start with best network performance

### Emulator Control
- `Ctrl + M` - Toggle hardware keyboard
- `F2` - Toggle fullscreen mode
- `Ctrl + Shift + P` - Power button
- `Ctrl + Shift + V` - Volume up
- `Ctrl + Shift + Down` - Volume down
- `Ctrl + Shift + H` - Home button
- `Ctrl + Shift + B` - Back button

## ProGuard/R8 Commands

### Common ProGuard Rules
- `-keep class com.example.MyClass` - Keep specific class
- `-keepclassmembers class * { public <methods>; }` - Keep all public methods
- `-dontwarn com.example.**` - Suppress warnings for package
- `-printmapping mapping.txt` - Output obfuscation mapping

### R8 Optimization
- `android.enableR8.fullMode=true` - Enable full R8 optimization in gradle.properties
- `-dontobfuscate` - Disable code obfuscation
- `-dontoptimize` - Disable code optimization

## Common File Paths

### Android SDK
- Windows: `C:\Users\<username>\AppData\Local\Android\Sdk`
- macOS: `~/Library/Android/sdk`
- Linux: `~/Android/Sdk`

### Important Directories
- `/data/data/<package_name>/` - App private data directory
- `/sdcard/Android/data/<package_name>/` - App external data directory
- `/system/app/` - System apps directory
- `/data/app/` - User-installed apps directory

## Useful Environment Variables

- `ANDROID_HOME` - Path to Android SDK
- `ANDROID_SDK_ROOT` - Alternative to ANDROID_HOME
- `JAVA_HOME` - Path to Java installation
- `PATH` - Include `$ANDROID_HOME/platform-tools` and `$ANDROID_HOME/tools`

## Quick Tips

### Performance Optimization
- Use `android:hardwareAccelerated="true"` for better graphics performance
- Enable R8 code shrinking in release builds
- Use vector drawables instead of multiple PNG files
- Implement proper memory management to avoid leaks

### Debugging Tips
- Use `Log.d()`, `Log.i()`, `Log.w()`, `Log.e()` for different log levels
- Set breakpoints in Android Studio for step-by-step debugging
- Use Layout Inspector to debug UI issues
- Monitor memory usage with Memory Profiler

### Best Practices
- Always test on multiple device configurations
- Use version control (Git) for your projects
- Keep your SDK and tools updated
- Follow Material Design guidelines for UI consistency