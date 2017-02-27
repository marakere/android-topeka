// Enables SDK auto-install, and uses it to run the given block
def withAndroidSdk(String sdkDir = '/home/sasikumar/android-sdk-linux',
 Closure body) {
 withEnv(["ANDROID_HOME=${sdkDir}"]) {
 body()
 }
}

node {
    
    stage('Checkout') {
    
                // send to email
                emailext (
                      subject: " ${env.JOB_NAME} - ${env.BUILD_NUMBER} Build Process Started",
                      body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                      to: 'sasikumar.mani@objectfrontier.com',
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                )
            
                try {
                   git 'https://github.com/sasikumarm/android-topeka.git'
                   currentBuild.result = 'SUCCESS'
                } catch (err) {
                    currentBuild.result = 'UNSTABLE'
                }
      
    }
    
    stage ('Clean') {
        
                withAndroidSdk {
                    try {
                      sh './gradlew clean assembleDebug'
                    } catch (err) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }   
                // Store the APK that was built
                archive '**/*-debug.apk'
    }
    
    stage ('Lint'){
        
                withAndroidSdk {
                    try {
                      sh './gradlew lint'
                    } catch (err) {
                        currentBuild.result = 'SUCCESS'
                    }
                stash includes: '*/build/outputs/lint-results*.xml', name: 'lint-reports'
                }    
    }
    
    stage('Junit') {
        
                withAndroidSdk {
                    try {
                      sh './gradlew test'
                      //step([$class: 'JacocoPublisher', inclusionPattern: 'build/jacoco/*.exec'])
                    } catch (err) {
                        currentBuild.result = 'UNSTABLE'
                    }
                    stash includes: '**/test-results/**/*.xml', name: 'junit-reports'
                }  
    }
    
    stage('Install') {
        
                withAndroidSdk {
                    try {
                        sh './gradlew installDebug'
                    } catch (err) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }  
                
                // Store the APK that was built
                archive '**/*-debug.apk'
           
    }
    
    stage ('HockyApp'){ 
        
                try {
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'sasikumarmofs1@gmail.com', sendToIndividuals: true])
                    step([$class: 'HockeyappRecorder', applications: [[apiToken: '1be3ad79e663459f9931e1df327a3255', downloadAllowed: true, filePath: '**/*-debug.apk', mandatory: false, notifyTeam: true, releaseNotesMethod: [$class: 'NoReleaseNotes'], uploadMethod: [$class: 'AppCreation', publicPage: false]]], debugMode: false, failGracefully: false])
                } catch (err) {
                    currentBuild.result = 'UNSTABLE'
                }
    }
    
    stage ('DeviceTest'){
        
                withAndroidSdk {
                    try {
                     sh './gradlew connectedAndroidTest'
                    } catch (err) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }    
    }
    
    stage ('AWSDeviceFarm'){
        
                withAndroidSdk {
                    try {
                      build job: 'AWSDeviceFarm'
                    } catch (err) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }    
    }
    
    
    stage ('Deploy'){ 
        
                try {
                        
                        if (currentBuild.result == 'SUCCESS') {
                            
                            //Deployment Confirmation
                            input message: 'Do you want deploy the ${env.BUILD_NUMBER} build into Google Store', ok: 'Depoly'
                            
                            //Sucess Email notification
                            emailext (
                                  subject: " ${env.JOB_NAME} - ${env.BUILD_NUMBER} Build deployed Into Google Store",
                                  body: """<p>Deployed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                                    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                                  to: 'sasikumar.mani@objectfrontier.com',
                                  recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                            )
                            
                        } else {
                            
                            //Failure Email notification
                            emailext (
                                  subject: " ${env.JOB_NAME} - ${env.BUILD_NUMBER} Deployment canceled due to build failures",
                                  body: """<p>Deployed canceled: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                                    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                                  to: 'sasikumar.mani@objectfrontier.com',
                                  recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                            )
                            
                            //JIRA Ticket Creation
                            withEnv(['JIRA_SITE=JIRA_SITE']) {
                              def testIssue = [fields: [ project: [id: 10000],
                                                   summary: "Fix the ${env.JOB_NAME} - ${env.BUILD_NUMBER} Build Failures",
                                                   description: "${env.JOB_NAME} - ${env.BUILD_NUMBER} Build Failures , URL : ${env.BUILD_URL}",
                                                   issuetype: [id: 10103]]]
                        
                              response = jiraNewIssue issue: testIssue
                            }
                            
                            
                        }
                } catch (err) {
                    currentBuild.result = 'UNSTABLE'
                }
    }
    
    stage('Report') {
        
                try {
                  unstash 'junit-reports'
                  step([$class: 'JUnitResultArchiver', testResults: '**/test-results/**/*.xml'])
                
                  unstash 'lint-reports'
                  step([$class: 'LintPublisher', canComputeNew: false, canRunOnFailed: true, defaultEncoding: '', healthy: '', pattern: '*/build/outputs/lint-results*.xml', unHealthy: ''])
                } catch (err) {
                   print 'Report unstash'
                }
        
    }
}
