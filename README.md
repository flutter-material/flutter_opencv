# flutter_opencv

1. 建立 plugin

```
flutter create --platforms=android,ios --template=plugin native_opencv
```

2. 新增 ffi 套件
在 `pubspec.yaml` 中新增

```
native_opencv:
    path: native_opencv
ffi: ^1.1.2
```

3. 設定 iOS plugin
在 `flutter_opencv` plugin 中的 ios 資料夾下有個 `native_opencv.podspec` 文件，並加入以下資訊。

```
# telling CocoaPods not to remove framework
s.preserve_paths = 'opencv2.framework' 

# telling linker to include opencv2 framework
s.xcconfig = { 'OTHER_LDFLAGS' => '-framework opencv2' }

# including OpenCV framework
s.vendored_frameworks = 'opencv2.framework' 

# including native framework
s.frameworks = 'AVFoundation'

# including C++ library
s.library = 'c++'
```

4. 設定 Android plugin
將下載的 OpenCV Android SDK 解壓縮到以下資料夾中。

```
sdk/native/libs/* -> native_opencv/android/src/main/jniLibs/*
sdk/native/jni/include -> native_opencv
```

在 `flutter_opencv` plugin 中的 Android 資料夾下建立 `CMakeLists.txt`。

```txt
cmake_minimum_required(VERSION 3.4.1)

include_directories(../include)
add_library(lib_opencv SHARED IMPORTED)
set_target_properties(lib_opencv PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java4.so)

find_library(log-lib log)

add_library(native_opencv SHARED ../ios/Classes/native_opencv.cpp)
target_link_libraries(native_opencv lib_opencv ${log-lib})
```

另外確認 `build.gradle` 加入以下資訊

```
android {
    defaultConfig {
        externalNativeBuild {
            cmake {
                // Enabling exceptions, RTTI
                // And setting C++ standard version
                cppFlags '-frtti -fexceptions -std=c++11'

                // Shared runtime for shared libraries
                arguments "-DANDROID_STL=c++_shared"
            }
        }
    }

    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}

android {
    packagingOptions {
        pickFirst 'lib/armeabi-v7a/libopencv_java4.so'
        pickFirst 'lib/arm64-v8a/libopencv_java4.so'
        pickFirst 'lib/x86/libopencv_java4.so'
        pickFirst 'lib/x86_64/libopencv_java4.so'
    }
}
```

5. 建立 cpp
在 iOS  Classes 資料夾下建立 `native_opencv.cpp`。

6. dart 連動 opencv
在最外層專案 lib 資料夾下新增 `native_opencv.dart`。