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

stage 'Clean and Assembles'

node {
    
// send to email
  emailext (
      subject: " ${env.JOB_NAME} - ${env.BUILD_NUMBER} Build Process Started",
      body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: 'Manjunath.arakere@objectfrontier.com',
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
    
    
// Check out the source code
 git 'https://github.com/sasikumarm/android-topeka.git'
// Build the app using the 'debug' build type,
// and allow SDK components to auto-install
withAndroidSdk {
 sh './gradlew clean assembleDebug'
}
 
 

// Store the APK that was built
 archive '**/*-debug.apk'
}

stage 'Run the JUnit Tests'

node {

withAndroidSdk {
sh './gradlew test'
}
 
 // Analyse the JUnit test results
 junit '**/TEST-*.xml'

}


stage 'Installs the Debug build'

node {

withAndroidSdk {
 //sh './gradlew installDebug'
}

// Store the APK that was built
 archive '**/*-debug.apk'
}

stage 'Deploy into HockyApp' 

node {
 
step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'sasikumarmofs1@gmail.com', sendToIndividuals: true])
step([$class: 'HockeyappRecorder', applications: [[apiToken: '1be3ad79e663459f9931e1df327a3255', downloadAllowed: true, filePath: '**/*-debug.apk', mandatory: false, notifyTeam: true, releaseNotesMethod: [$class: 'NoReleaseNotes'], uploadMethod: [$class: 'AppCreation', publicPage: false]]], debugMode: false, failGracefully: false])

}



stage 'Runs the tests for Debug build on connected devices'

node {

withAndroidSdk {
sh './gradlew connectedAndroidTest'
}

}

stage 'Deploy Into Googel Store'

node {
input message: 'Do you want deploy the latest build into Google Store', ok: 'Depoly'   

//emailext body: 'Build', recipientProviders: [[$class: 'DevelopersRecipientProvider']], subject: 'Build', to: 'sasikumar.mani@objectfrontier.com'

emailext (
      subject: " ${env.JOB_NAME} - ${env.BUILD_NUMBER} Build deployed Into Google Store",
      body: """<p>Deployed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: 'Manjunath.arakere@objectfrontier.com',
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
)
   
}


