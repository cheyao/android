cmake_minimum_required(VERSION 3.9)
project(android_build VERSION 1.0 DESCRIPTION "Build process for SDL android apk")

unset(ANDROID PARENT_SCOPE)

if(NOT DEFINED NAME)
    set(NAME "Set $\{NAME} to name")
endif()

string(REPLACE "." "/" JAVADIR ${NAMESPACE})

if(IMGUI STREQUAL ON)
    set(BUILD_IMGUI ", \"-DIMGUI=ON\"")
else()
    set(BUILD_IMGUI "")
endif()

# We need to configure these files
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/templates/build.gradle.in"  "${CMAKE_CURRENT_SOURCE_DIR}/app/build.gradle"                               @ONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/templates/strings.xml.in"   "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/values/strings.xml"            @ONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/templates/Activity.java.in"          "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/java/${JAVADIR}/MainActivity.java"    @ONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/templates/UsbSerialService.java.in" "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/java/${JAVADIR}/UsbSerialService.java" @ONLY)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	set(GRADLEW_COMMAND gradlew.bat)
else()
	set(GRADLEW_COMMAND gradlew)
endif()

file(GLOB_RECURSE JNI_SRC CONFIGURE_DEPENDS
	"${CMAKE_CURRENT_SOURCE_DIR}/app/jni/src/include/*.h"
	"${CMAKE_CURRENT_SOURCE_DIR}/app/jni/src/src/*.cpp"
)

add_custom_target(
    app_icon ALL
    COMMAND "magick" "${CMAKE_SOURCE_DIR}/assets/icon.bmp" "-resize" "192x192\!" "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png"
    COMMAND "magick" "${CMAKE_SOURCE_DIR}/assets/icon.bmp" "-resize" "144x144\!" "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/mipmap-xxhdpi/ic_launcher.png"
    COMMAND "magick" "${CMAKE_SOURCE_DIR}/assets/icon.bmp" "-resize" "96x96\!"   "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/mipmap-xhdpi/ic_launcher.png"
    COMMAND "magick" "${CMAKE_SOURCE_DIR}/assets/icon.bmp" "-resize" "72x72\!"   "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/mipmap-hdpi/ic_launcher.png"
    COMMAND "magick" "${CMAKE_SOURCE_DIR}/assets/icon.bmp" "-resize" "48x48\!"   "${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/mipmap-mdpi/ic_launcher.png"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk"
)

# 1. Build apk with gradle
add_custom_command(
	PRE_BUILD
	OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk"
	COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/${GRADLEW_COMMAND}" ARGS "assembleRelease"
	WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
	DEPENDS ${JNI_SRC} app_icon
)

# 2. Copy the binary to the output folder
add_custom_target(
	build_apk ALL
    COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk" "${CMAKE_BINARY_DIR}/${BUILD_NAME}-${VERSION}-Android.apk" 
	DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk"
)

