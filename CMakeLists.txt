cmake_minimum_required(VERSION 3.9)
project(android_build C)

unset(ANDROID PARENT_SCOPE)

set(KEYFILE ~/Developer/cheyao-android-key.keystore)
set(NAME Game)

string(REPLACE "." "/" JAVADIR ${NAMESPACE})

configure_file(app/build.gradle.in ${CMAKE_CURRENT_SOURCE_DIR}/app/build.gradle @ONLY)
configure_file(strings.xml.in ${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/res/values/strings.xml)
configure_file(Activity.java.in ${CMAKE_CURRENT_SOURCE_DIR}/app/src/main/java/${JAVADIR}/MainActivity.java)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	message("Unsupported")

	#execute_process(OUTPUT
	#	COMMAND "./gradlew.bat build && cp app/build/outputs/apk/release/app-release-unsigned.apk ."
	#	DEPENDS app/src/jni/src/*
	#	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
else()
	file(GLOB SRCC
		"app/jni/src/include/*.h"
		"app/jni/src/src/*.cpp"
	)
	
	add_custom_command(
		OUTPUT foo.c
		COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/gradlew build && cp app/build/outputs/apk/release/app-release-unsigned.apk ${CMAKE_BINARY_DIR}/unsigned.apk && ${CMAKE_CURRENT_SOURCE_DIR}/sign.sh ${CMAKE_BINARY_DIR}/unsigned.apk ${KEYFILE} && cp ${CMAKE_CURRENT_SOURCE_DIR}/foo.c ${CMAKE_CURRENT_BINARY_DIR}/foo.c 
		DEPENDS ${SRCC} 
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
	add_executable(tmp foo.c)
endif()

