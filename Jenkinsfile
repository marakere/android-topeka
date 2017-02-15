// Enables SDK auto-install, and uses it to run the given block
def withAndroidSdk(String sdkDir = '/home/sasikumar/android-sdk-linux',
 Closure body) {
 // Create the SDK directory, and accept the licences
 // (see: d.android.com/r/studio-ui/export-licenses.html)
 //writeFile file: "${sdkDir}/licenses/android-sdk-license",
 //text: '\n8933bad161af4178b1185d1a37fbf41ea5269c55'
 // Run the given closure with this SDK directory
 withEnv(["ANDROID_HOME=${sdkDir}"]) {
 body()
 }
}

stage 'Check out the source code'

node {
     git 'https://github.com/sasikumarm/android-topeka.git'
}

stage 'Assembles all Debug builds'

node {
    
withAndroidSdk {
sh './gradlew clean assembleDebug'
}

// Store the APK that was built
 archive '**/*-debug.apk'
}

stage 'Installs the Debug build'

node {
    
withAndroidSdk {
sh './gradlew installDebug'
}

// Store the APK that was built
 archive '**/*-debug.apk'
}

stage 'Run the JUnit Tests'

node {

withAndroidSdk {
sh './gradlew testDebugUnitTest'
}

}
