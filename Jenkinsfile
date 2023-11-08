node{
  def appName="sonarProjectKey"
  def To="gitTeamName;anotherTeamName"  //like To="appNotifierTeam;DevOpsTeam;appApproverTeam"
  //def To="mailId1, maildId2, mailId3"
  def CC=""
  def Subject=""
  def build_status="SUCCESS"
  def Deploy_status="SUCCESS"
  def envName=""
  def Approver=BUILD_USER_ID
  def buildFileName=""
  def deployEnv="ucdEnvName"

  try{
      
      stage('Checkout SCM'){
          stage_step="${STAGE_NAME}"
          try{
              checkout([$class: 'GitSCM', branches: [[name: 'refs/heads/branchNameWhereJenkinsfile']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout'], [$class: 'WipeWorspace']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'credentials', refspec: '+refs/heads/branchNameWhereJenkinsfile:refs/remotes/origin/branchNameWhereJenkinsfile', url: 'sshUrlWhereJenkinsAndTxtRepo']]])
              
              def githubRepoURL =sh(returnStdout:true, script: """
                source \$WORKSPACE/xyztxtFile.txt
                echo "\$repositoryURL"
              """
              )
              githubRepoURL = githubRepoURL.trim()
              println(githubRepoURL)

              def branchToPull =sh(returnStdout:true, script: """
                source \$WORKSPACE/xyztxtFile.txt
                echo "\$branch"
              """
              )
              branchToPull = branchToPull.trim()
              println(branchToPull)

               def branchName = "refs/heads" + branchToPull

               def branchRefSpec = "refs/heads" + branchToPull + ":refs/remotes/origin" + branchToPull                                

              def credentialsId_App =sh(returnStdout:true, script: """
                source \$WORKSPACE/xyztxtFile.txt
                echo "\$credentialsId_App"
              """
              )
              credentialsId_App = credentialsId_App.trim()
              println(credentialsId_App)

              checkout([$class: 'GitSCM', branches: [[name: branchName]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout'], [$class: 'WipeWorspace']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: credentialsId_App, refspec: branchRefSpec, url: githubRepoURL]]])
              
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)
            error "Failed at 'Checkout SCM', exiting now..."
          }
      }

      stage('SAST Analysis with SonarQube'){
          stage_step="${STAGE_NAME}"
          try{
             withEnv(['JAVA_HOME=pathToJDK']) {
               withSonarQubeEnv('SonarQube'){
                 withCredentials([string(credentialsId: 'credentialsId', variable: 'sonarpass')]) {
                    sh "sonarScannerPath -e -Dsonar.host.url=IPAddr -Dsonar.login=admin -Dsonar.password='$sonarpass' -Dsonar.projectName=sonarProjectName -Dsonar.projectVersion=1.0 -Dsonar.language=java -Dsonar.source=1.8 -Dsonar.projectKey=sonarProjectKey -Dsonar.sources=$WORKSPACE/src -Dsonar.java.binaries=$WORKSPACE/"  // pathTo/sonar-scanner-version/bin/sonar-scanner
                  }
               }  
             }
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)
            error "Failed at 'SAST Analysis with SonarQube', exiting now..."
          }
       }  

       stage('JUnit Testing'){
          stage_step="${STAGE_NAME}"
          try{
             withEnv(['JAVA_HOME=pathToJDK']) {
               sh '$JAVA_HOME/bin/java -version'
               sh 'antPath -f $WORKSPACE/jacoco-build.xml'

               junit skipPublishingChecks: true, testResults: '**/build/targetFolder/test-reports/*.xml'
               publishCoverage adapters: [jacocoAdapter('build/targetFolder/site/jacoco/report.xml')], checksName: '', skipPublishingChecks: true, sourceFileResolver: sourceFiles('NEVER_STORE')
               
               withSonarQubeEnv('SonarQube'){
                 withCredentials([string(credentialsId: 'credentialsId', variable: 'sonarpass')]) {
                    sh "sonarScannerPath -e -Dsonar.host.url=IPAddr -Dsonar.login=admin -Dsonar.password='$sonarpass' -Dsonar.projectName=sonarProjectName -Dsonar.projectVersion=1.0 -Dsonar.language=java -Dsonar.source=1.8 -Dsonar.projectKey=sonarProjectKey -Dsonar.sources=$WORKSPACE/src -Dsonar.java.binaries=$WORKSPACE/ -Dsonar.tests=$WORKSPACE/test/ -Dsonar.java.test.libraries=$WORKSPACE/jacoco/lib -Dsonar.core.codeCoveragePlugin=jacoco -Dsonar.junit.reportPaths=$WORKSPACE/build/targetFolder/test-reports -Dsonar.coverage.jacoco.xmlReportPaths=$WORKSPACE/build/targetFolder/site/jacoco/report.xml"  // pathTo/sonar-scanner-version/bin/sonar-scanner
                  }
               }  
             }
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)
            error "Failed at 'JUnit Testing', exiting now..."
          }
       } 
                
  }catch(Exception e){
      
  }
  
}
