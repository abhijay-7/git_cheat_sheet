# Android NDK Comprehensive Cheat Sheet (Kotlin + Jetpack Compose)

## What is Android NDK?

The Android Native Development Kit (NDK) allows you to implement parts of your app in native code (C/C++) and call them from Kotlin/Java using JNI (Java Native Interface). This is useful for:
- Performance-critical operations (image processing, games, mathematical computations)
- Reusing existing C/C++ libraries
- Platform-specific optimizations
- Security-sensitive code obfuscation

## NDK Setup & Configuration

### 1. Installing NDK

#### Via Android Studio
```bash
# In Android Studio: Tools > SDK Manager > SDK Tools
# Check "NDK (Side by side)" and install
```

#### Via Command Line
```bash
# Using sdkmanager
sdkmanager "ndk;25.2.9519653"  # Latest stable version
sdkmanager --list | grep ndk   # List available NDK versions
```

#### Environment Variables
```bash
# Add to ~/.bashrc or ~/.zshrc
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/25.2.9519653
export PATH=$PATH:$ANDROID_NDK_HOME
```

### 2. Project Configuration

#### build.gradle (Module: app)
```kotlin
android {
    compileSdk 34
    
    defaultConfig {
        ndk {
            // Specify target architectures
            abiFilters += listOf("arm64-v8a", "armeabi-v7a", "x86", "x86_64")
        }
        
        externalNativeBuild {
            cmake {
                // CMake arguments
                arguments += listOf("-DANDROID_STL=c++_shared")
                cppFlags += listOf("-std=c++17", "-frtti", "-fexceptions")
            }
        }
    }
    
    externalNativeBuild {
        cmake {
            path = file("src/main/cpp/CMakeLists.txt")
            version = "3.22.1"
        }
    }
    
    ndkVersion = "25.2.9519653"
}
```

#### gradle.properties
```properties
# Enable prefab for prebuilt libraries
android.prefabVersion=2.0.0
android.useAndroidX=true
```

## CMake Configuration

### Basic CMakeLists.txt
```cmake
# Minimum CMake version
cmake_minimum_required(VERSION 3.22.1)

# Project name
project("myapp")

# C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find required packages
find_library(log-lib log)
find_library(android-lib android)

# Add your native library
add_library(
    myapp              # Library name
    SHARED             # Shared library
    src/main/cpp/native.cpp
    src/main/cpp/utils.cpp
)

# Link libraries
target_link_libraries(
    myapp
    ${log-lib}
    ${android-lib}
)

# Include directories
target_include_directories(myapp PRIVATE
    src/main/cpp/include
)

# Compiler flags
target_compile_options(myapp PRIVATE
    -Wall
    -Wextra
    -O2
)
```

### Advanced CMakeLists.txt with External Libraries
```cmake
cmake_minimum_required(VERSION 3.22.1)
project("advancedapp")

set(CMAKE_CXX_STANDARD 17)

# Enable prefab
find_package(prefab REQUIRED CONFIG)

# Find external libraries (if using prefab)
find_package(oboe REQUIRED CONFIG)
find_package(opencv REQUIRED CONFIG)

# Source files
file(GLOB_RECURSE SOURCES
    "src/main/cpp/*.cpp"
    "src/main/cpp/*.c"
)

# Create library
add_library(advancedapp SHARED ${SOURCES})

# System libraries
find_library(log-lib log)
find_library(android-lib android)
find_library(gles-lib GLESv3)
find_library(egl-lib EGL)

# Link all libraries
target_link_libraries(advancedapp
    ${log-lib}
    ${android-lib}
    ${gles-lib}
    ${egl-lib}
    oboe::oboe
    opencv::opencv
)

# Preprocessor definitions
target_compile_definitions(advancedapp PRIVATE
    ANDROID_NDK
    GL_GLEXT_PROTOTYPES
)
```

## JNI Integration with Kotlin

### 1. Kotlin Native Interface Declaration

#### MainActivity.kt (Jetpack Compose)
```kotlin
package com.example.myapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

class MainActivity : ComponentActivity() {
    
    // Load native library
    companion object {
        init {
            System.loadLibrary("myapp")
        }
    }
    
    // Native method declarations
    external fun stringFromJNI(): String
    external fun addNumbers(a: Int, b: Int): Int
    external fun processImage(imageData: ByteArray, width: Int, height: Int): ByteArray
    external fun initializeNativeEngine(): Boolean
    external fun destroyNativeEngine()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Initialize native components
        initializeNativeEngine()
        
        setContent {
            MyAppTheme {
                MainScreen()
            }
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        destroyNativeEngine()
    }
    
    @Composable
    fun MainScreen() {
        var result by remember { mutableStateOf("") }
        
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Text(text = "NDK Integration Demo")
            
            Button(onClick = {
                result = stringFromJNI()
            }) {
                Text("Get Native String")
            }
            
            Button(onClick = {
                val sum = addNumbers(15, 25)
                result = "Sum: $sum"
            }) {
                Text("Add Numbers")
            }
            
            Text(text = result)
        }
    }
}
```

### 2. Data Transfer Classes

#### NativeDataManager.kt
```kotlin
class NativeDataManager {
    companion object {
        init {
            System.loadLibrary("myapp")
        }
    }
    
    // Primitive data types
    external fun processInt(value: Int): Int
    external fun processFloat(value: Float): Float
    external fun processDouble(value: Double): Double
    external fun processBoolean(value: Boolean): Boolean
    
    // Arrays
    external fun processIntArray(array: IntArray): IntArray
    external fun processFloatArray(array: FloatArray): FloatArray
    external fun processByteArray(array: ByteArray): ByteArray
    
    // Strings
    external fun processString(input: String): String
    external fun processStringArray(array: Array<String>): Array<String>
    
    // Complex objects
    external fun processImageData(
        pixels: IntArray,
        width: Int,
        height: Int,
        format: String
    ): IntArray
    
    // Callbacks
    external fun setProgressCallback(callback: (Int) -> Unit)
    external fun setErrorCallback(callback: (String) -> Unit)
}

// Data classes for complex operations
data class ImageData(
    val pixels: IntArray,
    val width: Int,
    val height: Int,
    val format: String
)

data class ProcessingResult(
    val success: Boolean,
    val data: ByteArray,
    val message: String
)
```

## C++ Implementation

### 1. Basic JNI Implementation

#### native.cpp
```cpp
#include <jni.h>
#include <string>
#include <vector>
#include <memory>
#include <android/log.h>

// Logging macros
#define LOG_TAG "MyApp_NDK"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__)

// Global variables
static JavaVM* g_jvm = nullptr;
static jobject g_callback_object = nullptr;

// JNI_OnLoad - called when library is loaded
JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
    LOGI("JNI_OnLoad called");
    g_jvm = vm;
    return JNI_VERSION_1_6;
}

// JNI_OnUnload - called when library is unloaded
JNIEXPORT void JNI_OnUnload(JavaVM* vm, void* reserved) {
    LOGI("JNI_OnUnload called");
    g_jvm = nullptr;
}

extern "C" {

// Basic string return
JNIEXPORT jstring JNICALL
Java_com_example_myapp_MainActivity_stringFromJNI(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++ NDK!";
    LOGI("Returning string: %s", hello.c_str());
    return env->NewStringUTF(hello.c_str());
}

// Simple arithmetic
JNIEXPORT jint JNICALL
Java_com_example_myapp_MainActivity_addNumbers(JNIEnv *env, jobject thiz, 
                                               jint a, jint b) {
    jint result = a + b;
    LOGI("Adding %d + %d = %d", a, b, result);
    return result;
}

// Process byte array (image data)
JNIEXPORT jbyteArray JNICALL
Java_com_example_myapp_MainActivity_processImage(JNIEnv *env, jobject thiz,
                                                jbyteArray imageData, 
                                                jint width, jint height) {
    // Get array elements
    jbyte* inputData = env->GetByteArrayElements(imageData, nullptr);
    jsize dataSize = env->GetArrayLength(imageData);
    
    LOGI("Processing image: %dx%d, size: %d bytes", width, height, dataSize);
    
    // Create output array
    jbyteArray outputArray = env->NewByteArray(dataSize);
    jbyte* outputData = env->GetByteArrayElements(outputArray, nullptr);
    
    // Process image (example: invert colors)
    for (int i = 0; i < dataSize; i++) {
        outputData[i] = 255 - inputData[i];
    }
    
    // Release arrays
    env->ReleaseByteArrayElements(imageData, inputData, JNI_ABORT);
    env->ReleaseByteArrayElements(outputArray, outputData, 0);
    
    return outputArray;
}

// Initialize native engine
JNIEXPORT jboolean JNICALL
Java_com_example_myapp_MainActivity_initializeNativeEngine(JNIEnv *env, jobject thiz) {
    LOGI("Initializing native engine");
    
    // Initialize your native components here
    // Return true on success, false on failure
    
    return JNI_TRUE;
}

// Cleanup native engine
JNIEXPORT void JNICALL
Java_com_example_myapp_MainActivity_destroyNativeEngine(JNIEnv *env, jobject thiz) {
    LOGI("Destroying native engine");
    
    // Cleanup your native components here
}

} // extern "C"
```

### 2. Advanced Data Processing

#### image_processor.cpp
```cpp
#include <jni.h>
#include <android/bitmap.h>
#include <android/log.h>
#include <vector>
#include <algorithm>
#include <cmath>

#define LOG_TAG "ImageProcessor"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)

// Image processing utilities
class ImageProcessor {
public:
    static void gaussianBlur(uint32_t* pixels, int width, int height, float sigma) {
        // Gaussian blur implementation
        int radius = static_cast<int>(ceil(3.0f * sigma));
        std::vector<float> kernel = createGaussianKernel(sigma, radius);
        
        // Apply horizontal blur
        std::vector<uint32_t> temp(width * height);
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                applyKernelHorizontal(pixels, temp.data(), x, y, width, height, kernel, radius);
            }
        }
        
        // Apply vertical blur
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                applyKernelVertical(temp.data(), pixels, x, y, width, height, kernel, radius);
            }
        }
    }
    
    static void adjustBrightness(uint32_t* pixels, int width, int height, float factor) {
        for (int i = 0; i < width * height; i++) {
            uint32_t pixel = pixels[i];
            
            int r = (pixel >> 16) & 0xFF;
            int g = (pixel >> 8) & 0xFF;
            int b = pixel & 0xFF;
            int a = (pixel >> 24) & 0xFF;
            
            r = std::min(255, static_cast<int>(r * factor));
            g = std::min(255, static_cast<int>(g * factor));
            b = std::min(255, static_cast<int>(b * factor));
            
            pixels[i] = (a << 24) | (r << 16) | (g << 8) | b;
        }
    }

private:
    static std::vector<float> createGaussianKernel(float sigma, int radius) {
        std::vector<float> kernel(2 * radius + 1);
        float sum = 0.0f;
        
        for (int i = -radius; i <= radius; i++) {
            kernel[i + radius] = exp(-(i * i) / (2.0f * sigma * sigma));
            sum += kernel[i + radius];
        }
        
        // Normalize
        for (float& k : kernel) {
            k /= sum;
        }
        
        return kernel;
    }
    
    static void applyKernelHorizontal(uint32_t* src, uint32_t* dst, 
                                    int x, int y, int width, int height,
                                    const std::vector<float>& kernel, int radius) {
        float r = 0, g = 0, b = 0, a = 0;
        
        for (int i = -radius; i <= radius; i++) {
            int px = std::max(0, std::min(width - 1, x + i));
            uint32_t pixel = src[y * width + px];
            
            float weight = kernel[i + radius];
            r += ((pixel >> 16) & 0xFF) * weight;
            g += ((pixel >> 8) & 0xFF) * weight;
            b += (pixel & 0xFF) * weight;
            a += ((pixel >> 24) & 0xFF) * weight;
        }
        
        dst[y * width + x] = (static_cast<uint32_t>(a) << 24) |
                           (static_cast<uint32_t>(r) << 16) |
                           (static_cast<uint32_t>(g) << 8) |
                           static_cast<uint32_t>(b);
    }
    
    static void applyKernelVertical(uint32_t* src, uint32_t* dst,
                                  int x, int y, int width, int height,
                                  const std::vector<float>& kernel, int radius) {
        float r = 0, g = 0, b = 0, a = 0;
        
        for (int i = -radius; i <= radius; i++) {
            int py = std::max(0, std::min(height - 1, y + i));
            uint32_t pixel = src[py * width + x];
            
            float weight = kernel[i + radius];
            r += ((pixel >> 16) & 0xFF) * weight;
            g += ((pixel >> 8) & 0xFF) * weight;
            b += (pixel & 0xFF) * weight;
            a += ((pixel >> 24) & 0xFF) * weight;
        }
        
        dst[y * width + x] = (static_cast<uint32_t>(a) << 24) |
                           (static_cast<uint32_t>(r) << 16) |
                           (static_cast<uint32_t>(g) << 8) |
                           static_cast<uint32_t>(b);
    }
};

extern "C" {

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_processIntArray(JNIEnv *env, jobject thiz,
                                                         jintArray inputArray) {
    jint* elements = env->GetIntArrayElements(inputArray, nullptr);
    jsize length = env->GetArrayLength(inputArray);
    
    // Process array elements
    for (int i = 0; i < length; i++) {
        elements[i] *= 2; // Example: double each element
    }
    
    env->ReleaseIntArrayElements(inputArray, elements, 0);
}

JNIEXPORT jstring JNICALL
Java_com_example_myapp_NativeDataManager_processString(JNIEnv *env, jobject thiz,
                                                       jstring input) {
    const char* inputStr = env->GetStringUTFChars(input, nullptr);
    
    // Process string (example: reverse it)
    std::string result(inputStr);
    std::reverse(result.begin(), result.end());
    
    env->ReleaseStringUTFChars(input, inputStr);
    return env->NewStringUTF(result.c_str());
}

// Bitmap processing
JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_processBitmap(JNIEnv *env, jobject thiz,
                                                       jobject bitmap, jfloat blurSigma) {
    AndroidBitmapInfo info;
    void* pixels;
    
    // Get bitmap info
    if (AndroidBitmap_getInfo(env, bitmap, &info) < 0) {
        LOGE("Failed to get bitmap info");
        return;
    }
    
    // Lock pixels
    if (AndroidBitmap_lockPixels(env, bitmap, &pixels) < 0) {
        LOGE("Failed to lock bitmap pixels");
        return;
    }
    
    // Process bitmap
    if (info.format == ANDROID_BITMAP_FORMAT_RGBA_8888) {
        ImageProcessor::gaussianBlur(static_cast<uint32_t*>(pixels), 
                                   info.width, info.height, blurSigma);
    }
    
    // Unlock pixels
    AndroidBitmap_unlockPixels(env, bitmap);
}

} // extern "C"
```

## Advanced NDK Features

### 1. Threading and Concurrency

#### thread_manager.cpp
```cpp
#include <jni.h>
#include <thread>
#include <future>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <atomic>

class ThreadPool {
private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    std::atomic<bool> stop{false};

public:
    ThreadPool(size_t threads) {
        for (size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        condition.wait(lock, [this] { return stop || !tasks.empty(); });
                        if (stop && tasks.empty()) return;
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) 
        -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;

        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if (stop) {
                throw std::runtime_error("enqueue on stopped ThreadPool");
            }
            tasks.emplace([task]() { (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        stop = true;
        condition.notify_all();
        for (std::thread &worker: workers) {
            worker.join();
        }
    }
};

static ThreadPool* g_thread_pool = nullptr;

extern "C" {

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_initializeThreadPool(JNIEnv *env, jobject thiz,
                                                              jint threadCount) {
    if (g_thread_pool == nullptr) {
        g_thread_pool = new ThreadPool(threadCount);
    }
}

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_processInBackground(JNIEnv *env, jobject thiz,
                                                            jbyteArray data, jobject callback) {
    if (g_thread_pool == nullptr) return;
    
    // Create global reference to callback
    jobject globalCallback = env->NewGlobalRef(callback);
    
    // Get data
    jbyte* dataPtr = env->GetByteArrayElements(data, nullptr);
    jsize dataSize = env->GetArrayLength(data);
    
    // Copy data for background processing
    std::vector<uint8_t> dataCopy(dataPtr, dataPtr + dataSize);
    env->ReleaseByteArrayElements(data, dataPtr, JNI_ABORT);
    
    // Submit task to thread pool
    g_thread_pool->enqueue([globalCallback, dataCopy]() {
        // Attach to current thread
        JNIEnv* env;
        JavaVM* jvm = g_jvm;
        jvm->AttachCurrentThread(&env, nullptr);
        
        // Process data (example: heavy computation)
        std::vector<uint8_t> result;
        for (uint8_t byte : dataCopy) {
            result.push_back(byte * 2); // Simple processing
        }
        
        // Create result array
        jbyteArray resultArray = env->NewByteArray(result.size());
        env->SetByteArrayRegion(resultArray, 0, result.size(), 
                              reinterpret_cast<const jbyte*>(result.data()));
        
        // Call callback
        jclass callbackClass = env->GetObjectClass(globalCallback);
        jmethodID onResult = env->GetMethodID(callbackClass, "onResult", "([B)V");
        env->CallVoidMethod(globalCallback, onResult, resultArray);
        
        // Cleanup
        env->DeleteGlobalRef(globalCallback);
        env->DeleteLocalRef(resultArray);
        env->DeleteLocalRef(callbackClass);
        
        jvm->DetachCurrentThread();
    });
}

} // extern "C"
```

### 2. Memory Management and Optimization

#### memory_manager.cpp
```cpp
#include <jni.h>
#include <memory>
#include <unordered_map>
#include <mutex>
#include <android/log.h>

#define LOG_TAG "MemoryManager"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)

// Custom memory pool for frequent allocations
class MemoryPool {
private:
    struct Block {
        size_t size;
        bool free;
        Block* next;
        char data[];
    };
    
    Block* head;
    std::mutex mutex;
    size_t total_size;
    size_t used_size;

public:
    MemoryPool(size_t pool_size = 1024 * 1024) : total_size(pool_size), used_size(0) {
        head = static_cast<Block*>(malloc(pool_size));
        head->size = pool_size - sizeof(Block);
        head->free = true;
        head->next = nullptr;
    }
    
    void* allocate(size_t size) {
        std::lock_guard<std::mutex> lock(mutex);
        
        Block* current = head;
        while (current) {
            if (current->free && current->size >= size) {
                current->free = false;
                used_size += size;
                
                // Split block if it's much larger
                if (current->size > size + sizeof(Block) + 64) {
                    Block* new_block = reinterpret_cast<Block*>(
                        current->data + size);
                    new_block->size = current->size - size - sizeof(Block);
                    new_block->free = true;
                    new_block->next = current->next;
                    
                    current->size = size;
                    current->next = new_block;
                }
                
                return current->data;
            }
            current = current->next;
        }
        
        // Fallback to system malloc
        LOGI("Pool exhausted, using system malloc for %zu bytes", size);
        return malloc(size);
    }
    
    void deallocate(void* ptr) {
        if (!ptr) return;
        
        std::lock_guard<std::mutex> lock(mutex);
        
        Block* current = head;
        while (current) {
            if (current->data == ptr) {
                current->free = true;
                used_size -= current->size;
                
                // Merge with next block if it's free
                if (current->next && current->next->free) {
                    current->size += current->next->size + sizeof(Block);
                    current->next = current->next->next;
                }
                
                return;
            }
            current = current->next;
        }
        
        // Not found in pool, use system free
        free(ptr);
    }
    
    ~MemoryPool() {
        free(head);
    }
};

static std::unique_ptr<MemoryPool> g_memory_pool;

// Smart pointer for automatic JNI cleanup
template<typename T>
class JNIPtr {
private:
    JNIEnv* env;
    T* ptr;
    std::function<void(JNIEnv*, T*)> deleter;

public:
    JNIPtr(JNIEnv* e, T* p, std::function<void(JNIEnv*, T*)> d) 
        : env(e), ptr(p), deleter(d) {}
    
    ~JNIPtr() {
        if (ptr && deleter) {
            deleter(env, ptr);
        }
    }
    
    T* get() const { return ptr; }
    T* operator->() const { return ptr; }
    T& operator*() const { return *ptr; }
    
    JNIPtr(const JNIPtr&) = delete;
    JNIPtr& operator=(const JNIPtr&) = delete;
    
    JNIPtr(JNIPtr&& other) noexcept 
        : env(other.env), ptr(other.ptr), deleter(std::move(other.deleter)) {
        other.ptr = nullptr;
    }
};

// Helper function to create JNI smart pointers
template<typename T>
JNIPtr<T> make_jni_ptr(JNIEnv* env, T* ptr, std::function<void(JNIEnv*, T*)> deleter) {
    return JNIPtr<T>(env, ptr, deleter);
}

extern "C" {

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_initializeMemoryPool(JNIEnv *env, jobject thiz,
                                                              jint poolSize) {
    g_memory_pool = std::make_unique<MemoryPool>(poolSize);
    LOGI("Memory pool initialized with %d bytes", poolSize);
}

JNIEXPORT jbyteArray JNICALL
Java_com_example_myapp_NativeDataManager_processLargeData(JNIEnv *env, jobject thiz,
                                                          jbyteArray inputData) {
    // Use smart pointer for automatic array cleanup
    auto input_ptr = make_jni_ptr(env, 
        env->GetByteArrayElements(inputData, nullptr),
        [inputData](JNIEnv* e, jbyte* p) { 
            e->ReleaseByteArrayElements(inputData, p, JNI_ABORT); 
        });
    
    jsize dataSize = env->GetArrayLength(inputData);
    
    // Use memory pool for temporary buffer
    void* tempBuffer = nullptr;
    if (g_memory_pool) {
        tempBuffer = g_memory_pool->allocate(dataSize * 2);
    } else {
        tempBuffer = malloc(dataSize * 2);
    }
    
    if (!tempBuffer) {
        return nullptr;
    }
    
    // Process data with temporary buffer
    // ... processing logic ...
    
    // Create result array
    jbyteArray result = env->NewByteArray(dataSize);
    env->SetByteArrayRegion(result, 0, dataSize, input_ptr.get());
    
    // Cleanup
    if (g_memory_pool) {
        g_memory_pool->deallocate(tempBuffer);
    } else {
        free(tempBuffer);
    }
    
    return result;
}

} // extern "C"
```

## Performance Optimization Techniques

### 1. SIMD Instructions (ARM NEON)

#### simd_processor.cpp
```cpp
#include <jni.h>
#include <arm_neon.h>
#include <android/log.h>

#define LOG_TAG "SIMDProcessor"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)

// SIMD-optimized image processing
class SIMDProcessor {
public:
    // Convert RGB to grayscale using NEON SIMD
    static void rgbToGrayscaleNEON(const uint8_t* rgb, uint8_t* gray, int pixels) {
        const float32x4_t weights = {0.299f, 0.587f, 0.114f, 0.0f};
        
        int i = 0;
        // Process 4 pixels at a time
        for (; i <= pixels - 4; i += 4) {
            // Load RGB data (12 bytes for 4 pixels)
            uint8x16_t rgb_data = vld1q_u8(&rgb[i * 3]);
            
            // Convert to float and separate channels
            uint8x8_t rgb_low = vget_low_u8(rgb_data);
            uint8x8_t rgb_high = vget_high_u8(rgb_data);
            
            uint16x8_t r16 = vmovl_u8(rgb_low);
            uint16x8_t g16 = vmovl_u8(rgb_high);
            
            float32x4_t r = vcvtq_f32_u32(vmovl_u16(vget_low_u16(r16)));
            float32x4_t g = vcvtq_f32_u32(vmovl_u16(vget_high_u16(r16)));
            float32x4_t b = vcvtq_f32_u32(vmovl_u16(vget_low_u16(g16)));
            
            // Apply weights
            float32x4_t result = vmulq_f32(r, vdupq_n_f32(0.299f));
            result = vmlaq_f32(result, g, vdupq_n_f32(0.587f));
            result = vmlaq_f32(result, b, vdupq_n_f32(0.114f));
            
            // Convert back to uint8
            uint32x4_t result_u32 = vcvtq_u32_f32(result);
            uint16x4_t result_u16 = vmovn_u32(result_u32);
            uint8x8_t result_u8 = vmovn_u16(vcombine_u16(result_u16, result_u16));
            
            // Store result
            vst1_lane_u32(reinterpret_cast<uint32_t*>(&gray[i]), 
                         vreinterpret_u32_u8(result_u8), 0);
        }
        
        // Handle remaining pixels
        for (; i < pixels; i++) {
            gray[i] = static_cast<uint8_t>(
                0.299f * rgb[i * 3] + 
                0.587f * rgb[i * 3 + 1] + 
                0.114f * rgb[i * 3 + 2]
            );
        }
    }
    
    // Vector addition using NEON
    static void addVectorsNEON(const float* a, const float* b, float* result, int size) {
        int i = 0;
        // Process 4 floats at a time
        for (; i <= size - 4; i += 4) {
            float32x4_t va = vld1q_f32(&a[i]);
            float32x4_t vb = vld1q_f32(&b[i]);
            float32x4_t vr = vaddq_f32(va, vb);
            vst1q_f32(&result[i], vr);
        }
        
        // Handle remaining elements
        for (; i < size; i++) {
            result[i] = a[i] + b[i];
        }
    }
    
    // Matrix multiplication using NEON
    static void matrixMultiplyNEON(const float* a, const float* b, float* c, 
                                  int m, int n, int k) {
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j += 4) {
                float32x4_t sum = vdupq_n_f32(0.0f);
                
                for (int l = 0; l < k; l++) {
                    float32x4_t va = vdupq_n_f32(a[i * k + l]);
                    float32x4_t vb = vld1q_f32(&b[l * n + j]);
                    sum = vmlaq_f32(sum, va, vb);
                }
                
                vst1q_f32(&c[i * n + j], sum);
            }
        }
    }
};

extern "C" {

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_convertRGBToGrayscale(JNIEnv *env, jobject thiz,
                                                               jbyteArray rgb, jbyteArray gray) {
    jbyte* rgbData = env->GetByteArrayElements(rgb, nullptr);
    jbyte* grayData = env->GetByteArrayElements(gray, nullptr);
    jsize rgbSize = env->GetArrayLength(rgb);
    
    int pixels = rgbSize / 3;
    
    SIMDProcessor::rgbToGrayscaleNEON(
        reinterpret_cast<uint8_t*>(rgbData),
        reinterpret_cast<uint8_t*>(grayData),
        pixels
    );
    
    env->ReleaseByteArrayElements(rgb, rgbData, JNI_ABORT);
    env->ReleaseByteArrayElements(gray, grayData, 0);
}

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_addFloatVectors(JNIEnv *env, jobject thiz,
                                                         jfloatArray a, jfloatArray b, 
                                                         jfloatArray result) {
    jfloat* aData = env->GetFloatArrayElements(a, nullptr);
    jfloat* bData = env->GetFloatArrayElements(b, nullptr);
    jfloat* resultData = env->GetFloatArrayElements(result, nullptr);
    jsize size = env->GetArrayLength(a);
    
    SIMDProcessor::addVectorsNEON(aData, bData, resultData, size);
    
    env->ReleaseFloatArrayElements(a, aData, JNI_ABORT);
    env->ReleaseFloatArrayElements(b, bData, JNI_ABORT);
    env->ReleaseFloatArrayElements(result, resultData, 0);
}

} // extern "C"

### 2. GPU Computing with OpenGL ES

#### gpu_processor.cpp
```cpp
#include <jni.h>
#include <GLES3/gl3.h>
#include <EGL/egl.h>
#include <android/log.h>
#include <string>

#define LOG_TAG "GPUProcessor"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)

class GPUProcessor {
private:
    EGLDisplay display;
    EGLContext context;
    EGLSurface surface;
    GLuint framebuffer;
    GLuint texture;
    GLuint program;
    
    static const char* vertexShaderSource;
    static const char* fragmentShaderSource;

public:
    bool initialize() {
        // Initialize EGL
        display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
        if (display == EGL_NO_DISPLAY) {
            LOGE("Failed to get EGL display");
            return false;
        }
        
        if (!eglInitialize(display, nullptr, nullptr)) {
            LOGE("Failed to initialize EGL");
            return false;
        }
        
        // Choose EGL config
        EGLConfig config;
        EGLint numConfigs;
        EGLint configAttribs[] = {
            EGL_RENDERABLE_TYPE, EGL_OPENGL_ES3_BIT,
            EGL_RED_SIZE, 8,
            EGL_GREEN_SIZE, 8,
            EGL_BLUE_SIZE, 8,
            EGL_ALPHA_SIZE, 8,
            EGL_NONE
        };
        
        if (!eglChooseConfig(display, configAttribs, &config, 1, &numConfigs)) {
            LOGE("Failed to choose EGL config");
            return false;
        }
        
        // Create EGL context
        EGLint contextAttribs[] = {
            EGL_CONTEXT_CLIENT_VERSION, 3,
            EGL_NONE
        };
        
        context = eglCreateContext(display, config, EGL_NO_CONTEXT, contextAttribs);
        if (context == EGL_NO_CONTEXT) {
            LOGE("Failed to create EGL context");
            return false;
        }
        
        // Create pbuffer surface
        EGLint surfaceAttribs[] = {
            EGL_WIDTH, 1,
            EGL_HEIGHT, 1,
            EGL_NONE
        };
        
        surface = eglCreatePbufferSurface(display, config, surfaceAttribs);
        if (surface == EGL_NO_SURFACE) {
            LOGE("Failed to create EGL surface");
            return false;
        }
        
        if (!eglMakeCurrent(display, surface, surface, context)) {
            LOGE("Failed to make EGL context current");
            return false;
        }
        
        // Initialize OpenGL resources
        return initializeGL();
    }
    
    bool initializeGL() {
        // Create shader program
        GLuint vertexShader = compileShader(GL_VERTEX_SHADER, vertexShaderSource);
        GLuint fragmentShader = compileShader(GL_FRAGMENT_SHADER, fragmentShaderSource);
        
        if (vertexShader == 0 || fragmentShader == 0) {
            return false;
        }
        
        program = glCreateProgram();
        glAttachShader(program, vertexShader);
        glAttachShader(program, fragmentShader);
        glLinkProgram(program);
        
        GLint linkStatus;
        glGetProgramiv(program, GL_LINK_STATUS, &linkStatus);
        if (linkStatus != GL_TRUE) {
            GLchar infoLog[512];
            glGetProgramInfoLog(program, 512, nullptr, infoLog);
            LOGE("Shader program linking failed: %s", infoLog);
            return false;
        }
        
        glDeleteShader(vertexShader);
        glDeleteShader(fragmentShader);
        
        return true;
    }
    
    GLuint compileShader(GLenum type, const char* source) {
        GLuint shader = glCreateShader(type);
        glShaderSource(shader, 1, &source, nullptr);
        glCompileShader(shader);
        
        GLint compileStatus;
        glGetShaderiv(shader, GL_COMPILE_STATUS, &compileStatus);
        if (compileStatus != GL_TRUE) {
            GLchar infoLog[512];
            glGetShaderInfoLog(shader, 512, nullptr, infoLog);
            LOGE("Shader compilation failed: %s", infoLog);
            glDeleteShader(shader);
            return 0;
        }
        
        return shader;
    }
    
    void processTexture(GLuint inputTexture, int width, int height) {
        // Create framebuffer
        glGenFramebuffers(1, &framebuffer);
        glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
        
        // Create output texture
        glGenTextures(1, &texture);
        glBindTexture(GL_TEXTURE_2D, texture);
        glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, width, height, 0, 
                     GL_RGBA, GL_UNSIGNED_BYTE, nullptr);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        
        glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, 
                              GL_TEXTURE_2D, texture, 0);
        
        if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE) {
            LOGE("Framebuffer not complete");
            return;
        }
        
        // Set viewport
        glViewport(0, 0, width, height);
        
        // Use shader program
        glUseProgram(program);
        
        // Bind input texture
        glActiveTexture(GL_TEXTURE0);
        glBindTexture(GL_TEXTURE_2D, inputTexture);
        glUniform1i(glGetUniformLocation(program, "inputTexture"), 0);
        
        // Draw full-screen quad
        drawFullScreenQuad();
        
        // Cleanup
        glBindFramebuffer(GL_FRAMEBUFFER, 0);
    }
    
    void drawFullScreenQuad() {
        float vertices[] = {
            -1.0f, -1.0f, 0.0f, 0.0f,
             1.0f, -1.0f, 1.0f, 0.0f,
             1.0f,  1.0f, 1.0f, 1.0f,
            -1.0f,  1.0f, 0.0f, 1.0f
        };
        
        unsigned int indices[] = {
            0, 1, 2,
            2, 3, 0
        };
        
        GLuint VAO, VBO, EBO;
        glGenVertexArrays(1, &VAO);
        glGenBuffers(1, &VBO);
        glGenBuffers(1, &EBO);
        
        glBindVertexArray(VAO);
        
        glBindBuffer(GL_ARRAY_BUFFER, VBO);
        glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
        
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
        glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
        
        glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 4 * sizeof(float), (void*)0);
        glEnableVertexAttribArray(0);
        
        glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 4 * sizeof(float), 
                             (void*)(2 * sizeof(float)));
        glEnableVertexAttribArray(1);
        
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
        
        glDeleteVertexArrays(1, &VAO);
        glDeleteBuffers(1, &VBO);
        glDeleteBuffers(1, &EBO);
    }
    
    ~GPUProcessor() {
        if (program) glDeleteProgram(program);
        if (texture) glDeleteTextures(1, &texture);
        if (framebuffer) glDeleteFramebuffers(1, &framebuffer);
        
        if (display != EGL_NO_DISPLAY) {
            eglMakeCurrent(display, EGL_NO_SURFACE, EGL_NO_SURFACE, EGL_NO_CONTEXT);
            if (context != EGL_NO_CONTEXT) eglDestroyContext(display, context);
            if (surface != EGL_NO_SURFACE) eglDestroySurface(display, surface);
            eglTerminate(display);
        }
    }
};

// Shader sources
const char* GPUProcessor::vertexShaderSource = R"(
#version 300 es
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

void main() {
    gl_Position = vec4(aPos, 0.0, 1.0);
    TexCoord = aTexCoord;
}
)";

const char* GPUProcessor::fragmentShaderSource = R"(
#version 300 es
precision mediump float;

in vec2 TexCoord;
out vec4 FragColor;

uniform sampler2D inputTexture;

void main() {
    vec4 color = texture(inputTexture, TexCoord);
    
    // Apply sepia effect
    float r = color.r;
    float g = color.g;
    float b = color.b;
    
    FragColor.r = min(1.0, (r * 0.393) + (g * 0.769) + (b * 0.189));
    FragColor.g = min(1.0, (r * 0.349) + (g * 0.686) + (b * 0.168));
    FragColor.b = min(1.0, (r * 0.272) + (g * 0.534) + (b * 0.131));
    FragColor.a = color.a;
}
)";

static std::unique_ptr<GPUProcessor> g_gpu_processor;

extern "C" {

JNIEXPORT jboolean JNICALL
Java_com_example_myapp_NativeDataManager_initializeGPUProcessor(JNIEnv *env, jobject thiz) {
    g_gpu_processor = std::make_unique<GPUProcessor>();
    return g_gpu_processor->initialize() ? JNI_TRUE : JNI_FALSE;
}

} // extern "C"

## Debugging and Profiling

### 1. Debug Configuration

#### Android.mk (if using ndk-build)
```makefile
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := myapp
LOCAL_SRC_FILES := native.cpp utils.cpp

# Debug flags
ifeq ($(APP_OPTIM),debug)
    LOCAL_CFLAGS += -DDEBUG -O0 -g
    LOCAL_CPPFLAGS += -DDEBUG -O0 -g -std=c++17
else
    LOCAL_CFLAGS += -DNDEBUG -O2
    LOCAL_CPPFLAGS += -DNDEBUG -O2 -std=c++17
endif

LOCAL_LDLIBS := -llog -landroid

include $(BUILD_SHARED_LIBRARY)
```

#### Application.mk
```makefile
APP_ABI := arm64-v8a armeabi-v7a x86 x86_64
APP_PLATFORM := android-21
APP_STL := c++_shared
APP_CPPFLAGS := -frtti -fexceptions

# Debug/Release configuration
APP_OPTIM := debug  # or release
```

### 2. Advanced Debugging

#### debug_utils.cpp
```cpp
#include <jni.h>
#include <android/log.h>
#include <signal.h>
#include <execinfo.h>
#include <unistd.h>
#include <cxxabi.h>
#include <memory>

#define LOG_TAG "DebugUtils"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)

// Stack trace utility
class StackTrace {
public:
    static void printStackTrace() {
        void* callstack[128];
        int frames = backtrace(callstack, 128);
        char** symbols = backtrace_symbols(callstack, frames);
        
        LOGI("Stack trace (%d frames):", frames);
        for (int i = 0; i < frames; ++i) {
            // Demangle C++ names
            std::string symbol = symbols[i];
            size_t start = symbol.find('(');
            size_t end = symbol.find('+');
            
            if (start != std::string::npos && end != std::string::npos && start < end) {
                std::string mangled = symbol.substr(start + 1, end - start - 1);
                
                int status;
                char* demangled = abi::__cxa_demangle(mangled.c_str(), nullptr, nullptr, &status);
                
                if (status == 0) {
                    LOGI("  #%d: %s", i, demangled);
                    free(demangled);
                } else {
                    LOGI("  #%d: %s", i, symbols[i]);
                }
            } else {
                LOGI("  #%d: %s", i, symbols[i]);
            }
        }
        
        free(symbols);
    }
};

// Signal handler for crashes
void crashHandler(int signal) {
    LOGE("Application crashed with signal %d", signal);
    StackTrace::printStackTrace();
    
    // Re-raise the signal to get system crash handler
    struct sigaction sa;
    sa.sa_handler = SIG_DFL;
    sigaction(signal, &sa, nullptr);
    raise(signal);
}

// Memory leak detector
class MemoryTracker {
private:
    struct Allocation {
        size_t size;
        const char* file;
        int line;
    };
    
    static std::unordered_map<void*, Allocation> allocations;
    static std::mutex mutex;
    static size_t total_allocated;

public:
    static void* allocate(size_t size, const char* file, int line) {
        void* ptr = malloc(size);
        if (ptr) {
            std::lock_guard<std::mutex> lock(mutex);
            allocations[ptr] = {size, file, line};
            total_allocated += size;
            LOGI("Allocated %zu bytes at %p (%s:%d)", size, ptr, file, line);
        }
        return ptr;
    }
    
    static void deallocate(void* ptr) {
        if (ptr) {
            std::lock_guard<std::mutex> lock(mutex);
            auto it = allocations.find(ptr);
            if (it != allocations.end()) {
                total_allocated -= it->second.size;
                LOGI("Deallocated %zu bytes at %p", it->second.size, ptr);
                allocations.erase(it);
            }
            free(ptr);
        }
    }
    
    static void reportLeaks() {
        std::lock_guard<std::mutex> lock(mutex);
        if (!allocations.empty()) {
            LOGE("Memory leaks detected! %zu allocations, %zu bytes total", 
                 allocations.size(), total_allocated);
            
            for (const auto& [ptr, alloc] : allocations) {
                LOGE("  Leak: %zu bytes at %p (%s:%d)", 
                     alloc.size, ptr, alloc.file, alloc.line);
            }
        } else {
            LOGI("No memory leaks detected");
        }
    }
};

std::unordered_map<void*, MemoryTracker::Allocation> MemoryTracker::allocations;
std::mutex MemoryTracker::mutex;
size_t MemoryTracker::total_allocated = 0;

// Debug macros
#ifdef DEBUG
#define DEBUG_ALLOC(size) MemoryTracker::allocate(size, __FILE__, __LINE__)
#define DEBUG_FREE(ptr) MemoryTracker::deallocate(ptr)
#define DEBUG_ASSERT(condition) \
    do { \
        if (!(condition)) { \
            LOGE("Assertion failed: %s at %s:%d", #condition, __FILE__, __LINE__); \
            StackTrace::printStackTrace(); \
            abort(); \
        } \
    } while(0)
#else
#define DEBUG_ALLOC(size) malloc(size)
#define DEBUG_FREE(ptr) free(ptr)
#define DEBUG_ASSERT(condition)
#endif

extern "C" {

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_installCrashHandler(JNIEnv *env, jobject thiz) {
    signal(SIGSEGV, crashHandler);
    signal(SIGABRT, crashHandler);
    signal(SIGFPE, crashHandler);
    signal(SIGILL, crashHandler);
    LOGI("Crash handler installed");
}

JNIEXPORT void JNICALL
Java_com_example_myapp_NativeDataManager_reportMemoryLeaks(JNIEnv *env, jobject thiz) {
    MemoryTracker::reportLeaks();
}

} // extern "C"

## Best Practices and Common Patterns

### 1. Jetpack Compose Integration Example

#### ComposableWithNDK.kt
```kotlin
@Composable
fun ImageProcessingScreen() {
    var originalBitmap by remember { mutableStateOf<Bitmap?>(null) }
    var processedBitmap by remember { mutableStateOf<Bitmap?>(null) }
    var isProcessing by remember { mutableStateOf(false) }
    var processingTime by remember { mutableStateOf(0L) }
    
    val launcher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.GetContent()
    ) { uri ->
        uri?.let { imageUri ->
            // Load bitmap from URI
            // originalBitmap = loadBitmapFromUri(imageUri)
        }
    }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        // Image display
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            // Original image
            Card(modifier = Modifier.weight(1f)) {
                Column(
                    modifier = Modifier.padding(8.dp),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    Text("Original")
                    originalBitmap?.let { bitmap ->
                        Image(
                            bitmap = bitmap.asImageBitmap(),
                            contentDescription = "Original",
                            modifier = Modifier.size(150.dp)
                        )
                    } ?: Box(
                        modifier = Modifier.size(150.dp),
                        contentAlignment = Alignment.Center
                    ) {
                        Text("No image")
                    }
                }
            }
            
            Spacer(modifier = Modifier.width(8.dp))
            
            // Processed image
            Card(modifier = Modifier.weight(1f)) {
                Column(
                    modifier = Modifier.padding(8.dp),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    Text("Processed")
                    processedBitmap?.let { bitmap ->
                        Image(
                            bitmap = bitmap.asImageBitmap(),
                            contentDescription = "Processed",
                            modifier = Modifier.size(150.dp)
                        )
                    } ?: Box(
                        modifier = Modifier.size(150.dp),
                        contentAlignment = Alignment.Center
                    ) {
                        if (isProcessing) {
                            CircularProgressIndicator()
                        } else {
                            Text("No processed image")
                        }
                    }
                }
            }
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // Controls
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            Button(
                onClick = { launcher.launch("image/*") }
            ) {
                Text("Select Image")
            }
            
            Button(
                onClick = {
                    originalBitmap?.let { bitmap ->
                        isProcessing = true
                        
                        // Process image in background thread
                        Thread {
                            val startTime = System.currentTimeMillis()
                            
                            // Call native processing function
                            val processed = processImageNative(bitmap)
                            
                            val endTime = System.currentTimeMillis()
                            processingTime = endTime - startTime
                            
                            // Update UI on main thread
                            Handler(Looper.getMainLooper()).post {
                                processedBitmap = processed
                                isProcessing = false
                            }
                        }.start()
                    }
                },
                enabled = originalBitmap != null && !isProcessing
            ) {
                Text("Process with NDK")
            }
        }
        
        if (processingTime > 0) {
            Text(
                text = "Processing time: ${processingTime}ms",
                style = MaterialTheme.typography.bodySmall
            )
        }
    }
}

private fun processImageNative(bitmap: Bitmap): Bitmap {
    val width = bitmap.width
    val height = bitmap.height
    val pixels = IntArray(width * height)
    
    bitmap.getPixels(pixels, 0, width, 0, 0, width, height)
    
    // Call native function
    NativeDataManager().processPixels(pixels, width, height)
    
    return Bitmap.createBitmap(pixels, width, height, Bitmap.Config.ARGB_8888)
}
```

### 2. Error Handling Patterns

#### error_handling.cpp
```cpp
// Error codes
enum class NDKError {
    SUCCESS = 0,
    INVALID_PARAMETER = -1,
    OUT_OF_MEMORY = -2,
    INITIALIZATION_FAILED = -3,
    PROCESSING_FAILED = -4,
    UNSUPPORTED_FORMAT = -5
};

// Error handling utility
class ErrorHandler {
public:
    static void setLastError(NDKError error, const std::string& message) {
        std::lock_guard<std::mutex> lock(mutex_);
        lastError_ = error;
        lastMessage_ = message;
        LOGE("NDK Error %d: %s", static_cast<int>(error), message.c_str());
    }
    
    static NDKError getLastError() {
        std::lock_guard<std::mutex> lock(mutex_);
        return lastError_;
    }
    
    static std::string getLastMessage() {
        std::lock_guard<std::mutex> lock(mutex_);
        return lastMessage_;
    }
    
    static void clearError() {
        std::lock_guard<std::mutex> lock(mutex_);
        lastError_ = NDKError::SUCCESS;
        lastMessage_.clear();
    }

private:
    static std::mutex mutex_;
    static NDKError lastError_;
    static std::string lastMessage_;
};

std::mutex ErrorHandler::mutex_;
NDK