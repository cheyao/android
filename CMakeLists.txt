cmake_minimum_required(VERSION 3.9)
project(android_build VERSION 1.0 DESCRIPTION "Build process for SDL android apk")

unset(ANDROID PARENT_SCOPE)

set(NAME Game)

string(REPLACE "." "/" JAVADIR ${NAMESPACE})

configure_file(app/build.gradle.in ${CMAKE_CURRENT_SOURCE_DIR}/app/build.gradle @ONLY)
configure_file(strings.xml.in ${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/values/strings.xml)
configure_file(Activity.java.in ${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/java/${JAVADIR}/MainActivity.java)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	set(GRADLEW_COMMAND gradlew.bat)
else()
	set(GRADLEW_COMMAND gradlew)
endif()

file(GLOB_RECURSE JNI_SRC CONFIGURE_DEPENDS
	"${CMAKE_CURRENT_SOURCE_DIR}/app/jni/src/include/*.h"
	"${CMAKE_CURRENT_SOURCE_DIR}/app/jni/src/src/*.cpp"
)

# 1. Build apk with gradle
add_custom_command(
	PRE_BUILD
	OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk"
	COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/${GRADLEW_COMMAND}" ARGS "assembleRelease"
	WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
	DEPENDS ${JNI_SRC}
)

# 2. Copy the binary to the output folder
add_custom_target(
	build_apk ALL
	COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk" "${CMAKE_BINARY_DIR}/" 
	DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/app/build/outputs/apk/release/app-release.apk"
)
