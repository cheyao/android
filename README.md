# SDL3 Android build
This is a cmake build module for SDL

Freetype2 and SDL3 is included within

Usage:
```
set(NAMESPACE com.${NAMESPACE}.${PROJECT_NAME})
set(BUILD_NAME ${PROJECT_NAME})

add_subdirectory(android)
```

Configure your gradlew's credentials
```
File: ~/.gradle/gradle.properties

RELEASE_STORE_FILE=[KEYSTORE PATH RELATIVE FROM ./android]
RELEASE_STORE_PASSWORD=[KEYSTORE PASSWORD]
RELEASE_KEY_ALIAS=[KEY NAME]
RELEASE_KEY_PASSWORD=[KEY PASSWORD]
```

Output: app-release.apk
