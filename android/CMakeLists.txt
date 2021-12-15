cmake_minimum_required(VERSION 3.5.1)

set (CMAKE_VERBOSE_MAKEFILE ON)
set (CMAKE_CXX_STANDARD 14)
set (CMAKE_CXX_FLAGS "-DFOLLY_NO_CONFIG=1 -DFOLLY_HAVE_CLOCK_GETTIME=1 -DFOLLY_HAVE_MEMRCHR=1 -DFOLLY_USE_LIBCPP=1 -DFOLLY_MOBILE=1 -DON_ANDROID -DONANDROID -fexceptions -fno-omit-frame-pointer -frtti -Wno-sign-compare")

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON CACHE INTERNAL "")

set (PACKAGE_NAME "reanimated")
set (BUILD_DIR ${CMAKE_SOURCE_DIR}/build)
set (FBJNI_HEADERS_DIR ${NODE_MODULES_DIR}/react-native/ReactAndroid/src/main/jni/first-party/fbjni/headers)

# reanimated shared

file(GLOB sources_native_modules  "./src/main/Common/cpp/NativeModules/*.cpp")
file(GLOB sources_shared_items  "./src/main/Common/cpp/SharedItems/*.cpp")
file(GLOB sources_registries  "./src/main/Common/cpp/Registries/*.cpp")
file(GLOB sources_android  "./src/main/cpp/*.cpp")

add_library(
        ${PACKAGE_NAME}
        SHARED
        ${sources_native_modules}
        ${sources_shared_items}
        ${sources_registries}
        ${sources_android}
        "${NODE_MODULES_DIR}/react-native/ReactCommon/jsi/jsi/JSIDynamic.cpp"
        "./src/main/Common/cpp/Tools/JSIStoreValueUser.cpp"
        "./src/main/Common/cpp/Tools/Mapper.cpp"
        "./src/main/Common/cpp/Tools/RuntimeDecorator.cpp"
        "./src/main/Common/cpp/Tools/Scheduler.cpp"
        "./src/main/Common/cpp/Tools/WorkletEventHandler.cpp"
        "./src/main/Common/cpp/Tools/FeaturesConfig.cpp"
        "./src/main/Common/cpp/LayoutAnimations/LayoutAnimationsProxy.cpp"
)


file (GLOB LIBFBJNI_INCLUDE_DIR ${FBJNI_HEADERS_DIR})

target_include_directories(
        ${PACKAGE_NAME}
        PRIVATE
        ${LIBFBJNI_INCLUDE_DIR}
        ${BUILD_DIR}/third-party-ndk/boost
        ${BUILD_DIR}/third-party-ndk/double-conversion
        ${BUILD_DIR}/third-party-ndk/folly
        "${BUILD_DIR}/third-party-ndk/glog/exported"
        "${NODE_MODULES_DIR}/react-native/React"
        "${NODE_MODULES_DIR}/react-native/React/Base"
        "${NODE_MODULES_DIR}/react-native/ReactAndroid/src/main/jni"
        "${NODE_MODULES_DIR}/react-native/ReactAndroid/src/main/java/com/facebook/react/turbomodule/core/jni"
        "${NODE_MODULES_DIR}/react-native/ReactCommon"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/callinvoker"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/jsi"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/turbomodule/core"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/turbomodule"
        "./src/main/Common/cpp/headers/Tools"
        "./src/main/Common/cpp/headers/SpecTools"
        "./src/main/Common/cpp/headers/NativeModules"
        "./src/main/Common/cpp/headers/SharedItems"
        "./src/main/Common/cpp/headers/Registries"
        "./src/main/Common/cpp/headers/LayoutAnimations"
        "./src/main/Common/cpp/hidden_headers"
        "./src/main/cpp/headers"
)

# find libraries
file (GLOB LIBRN_DIR "${BUILD_DIR}/react-native-0*/jni/${ANDROID_ABI}")
file (GLOB HERMES_DIR "${BUILD_DIR}/third-party-ndk/hermes/jni/${ANDROID_ABI}")


find_library(
        LOG_LIB
        log
)
find_library(
        HERMES_LIB
        hermes
        PATHS ${HERMES_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        V8EXECUTOR_LIB
        v8executor
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        JSEXECUTOR_LIB
        jscexecutor
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        FOLLY_JSON_LIB
        folly_json
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        REACT_NATIVE_JNI_LIB
        reactnativejni
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)

find_library(
        GLOG_LIB
        glog
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_package(fbjni REQUIRED CONFIG)

find_library(
        JSI_LIB
        jsi
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)


# build shared lib


if(${JS_RUNTIME} STREQUAL "hermes")
        target_link_libraries( 
                ${PACKAGE_NAME}
                ${LOG_LIB}
                ${JSI_LIB}
                ${HERMES_LIB}
                ${GLOG_LIB}
                fbjni::fbjni
                ${FOLLY_JSON_LIB}
                ${REACT_NATIVE_JNI_LIB}
                android
        )
    string(APPEND CMAKE_CXX_FLAGS " -DJS_RUNTIME_HERMES=1")
elseif(${JS_RUNTIME} STREQUAL "v8")
        target_link_libraries( 
                ${PACKAGE_NAME}
                ${LOG_LIB}
                ${JSI_LIB}
                ${V8EXECUTOR_LIB}
                ${GLOG_LIB}
                fbjni::fbjni
                ${FOLLY_JSON_LIB}
                ${REACT_NATIVE_JNI_LIB}
                android
        )
        target_include_directories(
                ${PACKAGE_NAME}
                PRIVATE
                "${NODE_MODULES_DIR}/react-native-v8/src"
        )
    string(APPEND CMAKE_CXX_FLAGS " -DJS_RUNTIME_V8=1")
else()
        target_link_libraries( 
                ${PACKAGE_NAME}
                ${LOG_LIB}
                ${JSI_LIB}
                ${JSEXECUTOR_LIB}
                ${GLOG_LIB}
                fbjni::fbjni
                ${FOLLY_JSON_LIB}
                ${REACT_NATIVE_JNI_LIB}
                android
        )
    string(APPEND CMAKE_CXX_FLAGS " -DJS_RUNTIME_JSC=1")
endif()
