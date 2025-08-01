# Gradle Commands & Android Studio Terminal Control Cheat Sheet

## Gradle Wrapper Commands

### Basic Build Commands
- `./gradlew build` - Build the entire project (debug + release)
- `./gradlew clean` - Delete all build outputs and intermediate files
- `./gradlew rebuild` - Clean and build the project
- `./gradlew assemble` - Build all variants without running tests
- `./gradlew check` - Run all checks (tests, lint, etc.) without building APKs

### APK Generation
- `./gradlew assembleDebug` - Build debug APK only
- `./gradlew assembleRelease` - Build release APK only
- `./gradlew bundleDebug` - Create debug Android App Bundle (AAB)
- `./gradlew bundleRelease` - Create release Android App Bundle (AAB)
- `./gradlew installDebug` - Build and install debug APK to connected device
- `./gradlew installRelease` - Build and install release APK to connected device
- `./gradlew uninstallDebug` - Uninstall debug version from device
- `./gradlew uninstallRelease` - Uninstall release version from device

### Multi-Module Commands
- `./gradlew :app:assembleDebug` - Build debug APK for specific module (app)
- `./gradlew :library:build` - Build specific library module
- `./gradlew app:dependencies` - Show dependencies for app module only
- `./gradlew :app:clean :app:build` - Clean and build specific module

## Testing Commands

### Unit Testing
- `./gradlew test` - Run all unit tests across all modules
- `./gradlew testDebug` - Run unit tests for debug variant only
- `./gradlew testRelease` - Run unit tests for release variant only
- `./gradlew testDebugUnitTest` - Run debug unit tests specifically
- `./gradlew :app:testDebugUnitTest` - Run unit tests for specific module
- `./gradlew test --tests="*LoginTest*"` - Run specific test classes matching pattern
- `./gradlew test --tests="com.example.LoginTest.testValidLogin"` - Run specific test method

### Instrumented Testing
- `./gradlew connectedAndroidTest` - Run instrumented tests on connected devices
- `./gradlew connectedDebugAndroidTest` - Run debug instrumented tests only
- `./gradlew :app:connectedAndroidTest` - Run instrumented tests for specific module
- `./gradlew createDebugCoverageReport` - Generate test coverage report
- `./gradlew jacocoTestReport` - Generate JaCoCo test coverage report

### Test Reports
- `./gradlew testDebugUnitTest jacocoTestDebugUnitTestReport` - Unit tests with coverage
- `./gradlew connectedCheck` - Run all device tests and generate reports

## Code Quality & Analysis

### Lint Commands
- `./gradlew lint` - Run lint checks on all variants
- `./gradlew lintDebug` - Run lint on debug variant only
- `./gradlew lintRelease` - Run lint on release variant only
- `./gradlew lintVital` - Run lint checks for critical issues only (release builds)
- `./gradlew :app:lint` - Run lint on specific module

### Static Analysis
- `./gradlew detekt` - Run Detekt static analysis (if configured)
- `./gradlew ktlintCheck` - Check Kotlin code style with ktlint
- `./gradlew ktlintFormat` - Auto-format Kotlin code with ktlint
- `./gradlew spotlessCheck` - Check code formatting with Spotless
- `./gradlew spotlessApply` - Apply code formatting with Spotless

## Project Information Commands

### Dependencies
- `./gradlew dependencies` - Show all project dependencies
- `./gradlew :app:dependencies` - Show dependencies for specific module
- `./gradlew dependencyInsight --dependency <name>` - Analyze specific dependency
- `./gradlew :app:dependencies --configuration debugCompileClasspath` - Show debug compile dependencies
- `./gradlew buildEnvironment` - Show build script dependencies

### Project Structure
- `./gradlew projects` - List all modules/subprojects
- `./gradlew tasks` - Show all available tasks
- `./gradlew tasks --all` - Show all tasks including dependencies
- `./gradlew :app:tasks` - Show tasks for specific module
- `./gradlew help --task <taskname>` - Get help for specific task

### Properties & Configuration
- `./gradlew properties` - Show all project properties
- `./gradlew :app:properties` - Show properties for specific module
- `./gradlew model` - Show project model information

## Performance & Debugging

### Build Performance
- `./gradlew build --profile` - Build with performance profiling
- `./gradlew build --scan` - Generate build scan (upload to gradle.com)
- `./gradlew build --dry-run` - Show what would be executed without running
- `./gradlew build --continue` - Continue build even after task failures
- `./gradlew assembleDebug --rerun-tasks` - Force re-execution of all tasks

### Build Cache & Daemon
- `./gradlew build --build-cache` - Enable build cache for this build
- `./gradlew --stop` - Stop all Gradle daemons
- `./gradlew --status` - Show status of Gradle daemons
- `./gradlew cleanBuildCache` - Clean local build cache

### Debugging Options
- `./gradlew build --debug` - Run with debug logging
- `./gradlew build --info` - Run with info logging
- `./gradlew build --stacktrace` - Show stack traces for failures
- `./gradlew build --full-stacktrace` - Show full stack traces
- `./gradlew build -Dorg.gradle.debug=true` - Enable remote debugging

### Offline & Network
- `./gradlew build --offline` - Build without network access
- `./gradlew build --refresh-dependencies` - Refresh all dependencies from repositories
- `./gradlew build --no-daemon` - Run without Gradle daemon

## Android Studio Terminal Control

### Opening Terminals
- `Alt + F12` (Windows/Linux) / `Option + F12` (Mac) - Toggle terminal panel
- `Ctrl + Shift + T` / `Cmd + Shift + T` - New terminal tab
- `Alt + Right/Left` - Switch between terminal tabs
- `Ctrl + Shift + F12` / `Cmd + Shift + F12` - Hide all tool windows (including terminal)

### Terminal Navigation
- `Ctrl + A` / `Cmd + A` - Move cursor to beginning of line
- `Ctrl + E` / `Cmd + E` - Move cursor to end of line
- `Ctrl + K` / `Cmd + K` - Clear line from cursor to end
- `Ctrl + U` / `Cmd + U` - Clear entire line
- `Ctrl + L` / `Cmd + L` - Clear terminal screen
- `Ctrl + C` / `Cmd + C` - Cancel current command
- `Ctrl + D` / `Cmd + D` - Exit terminal (if empty line)

### Command History
- `Up/Down arrows` - Navigate command history
- `Ctrl + R` / `Cmd + R` - Search command history (reverse search)
- `history` - Show command history
- `!!` - Run last command
- `!n` - Run command number n from history

### Terminal Configuration
- Right-click terminal → "Terminal Options" - Configure terminal settings
- Settings → Tools → Terminal - Global terminal settings
- Settings → Keymap → "Terminal" - Customize terminal shortcuts

## Advanced Gradle Configurations

### Build Variants
- `./gradlew assembleDebug` - Build debug variant
- `./gradlew assembleRelease` - Build release variant
- `./gradlew assembleFreeDebug` - Build specific flavor + build type combination
- `./gradlew assemblePaidRelease` - Build paid flavor with release build type

### Custom Tasks
```bash
# Run custom task defined in build.gradle
./gradlew customTaskName

# Run task with parameters
./gradlew myTask -PmyParam=value

# Run task with system properties
./gradlew myTask -Dmyprop=value
```

### Parallel Execution
- `./gradlew build --parallel` - Enable parallel execution
- `./gradlew build --max-workers=4` - Limit parallel workers
- `./gradlew build --no-parallel` - Disable parallel execution

## Gradle Properties Configuration

### gradle.properties Settings
```properties
# Performance settings
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.configureondemand=true
org.gradle.jvmargs=-Xmx4g -XX:MaxPermSize=512m

# Android specific
android.useAndroidX=true
android.enableJetifier=true
android.enableR8.fullMode=true
```

### Command Line Properties
- `./gradlew build -Pkey=value` - Set project property
- `./gradlew build -Dkey=value` - Set system property
- `./gradlew build -PbuildNumber=123` - Pass build number
- `./gradlew build -Prelease=true` - Conditional build logic

## Troubleshooting Commands

### Clean Operations
- `./gradlew clean` - Clean all build outputs
- `./gradlew cleanBuildCache` - Clean build cache
- `./gradlew --stop` - Stop all Gradle daemons
- `rm -rf ~/.gradle/caches/` - Manual cache cleanup (Unix/Mac)
- `rmdir /s ~/.gradle/caches/` - Manual cache cleanup (Windows)

### Dependency Issues
- `./gradlew dependencies --configuration debugRuntimeClasspath` - Debug runtime deps
- `./gradlew dependencyInsight --dependency gson` - Analyze specific dependency
- `./gradlew build --refresh-dependencies` - Force dependency refresh

### Build Issues
- `./gradlew build --stacktrace --debug` - Maximum debugging info
- `./gradlew build --no-build-cache` - Disable build cache
- `./gradlew build --rerun-tasks` - Force task re-execution

## Useful Terminal Aliases

Add these to your shell profile (.bashrc, .zshrc, etc.):

```bash
# Gradle shortcuts
alias gw='./gradlew'
alias gwb='./gradlew build'
alias gwc='./gradlew clean'
alias gwr='./gradlew rebuild'
alias gwad='./gradlew assembleDebug'
alias gwar='./gradlew assembleRelease'
alias gwid='./gradlew installDebug'
alias gwt='./gradlew test'
alias gwl='./gradlew lint'

# Combined operations
alias gwcb='./gradlew clean build'
alias gwcid='./gradlew clean installDebug'
```

## Quick Tips

### Performance Optimization
- Use `--parallel` flag for multi-module projects
- Enable build cache with `--build-cache`
- Use `--offline` when network is slow
- Increase heap size in gradle.properties: `org.gradle.jvmargs=-Xmx4g`

### Debugging Build Issues
- Always use `--stacktrace` when investigating failures
- Use `--info` or `--debug` for verbose logging
- Check daemon status with `--status` if builds seem slow
- Clear caches if experiencing odd behavior

### Terminal Productivity
- Use command history (Up arrow) instead of retyping
- Create aliases for frequently used commands
- Use tab completion for file paths and Gradle tasks
- Keep multiple terminal tabs open for different purposes