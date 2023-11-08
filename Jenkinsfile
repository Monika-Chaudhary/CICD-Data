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

       stage('Compile and Build EAR'){
          stage_step="${STAGE_NAME}"
          try{
             withEnv(['JAVA_HOME=pathToJDK']) {
               sh '''
               antPath -f $WORKSPACE/build.xml
               '''
             }
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)
            error "Failed at 'Compile and Build EAR', exiting now..."
          }
       }

       stage('Nexus Upload of Ear Artifact'){
          stage_step="${STAGE_NAME}"
          try{
             withCredentials([string(credentialsId: 'credentialsId', variable: 'nexus')]) {
               sh '''
               dir_to_fetch=$WORKSPACE/build/target-EAR/
               if[ ! -d $dir_to_fetch ]; then
                 echo "$dir_to_fetch is not valid directory, Exiting"
                 exit 1
               fi
               cd $dir_to_fetch
               cnt=`ls -1 *.ear | wc -l`
               if [ $cnt -ls 1 ]; then
                 echo "Directory is empty. Nothing to upload, Exiting"
                 exit 1
               fi
               ls -1 | while read filename
               do
                 framed_name=${dir_to_fetch}/${filename}
                 curl -v -u admin:${nexus} nexusIpAddr/nexus/content/repositories/repoAppName/${BUILD_ID}/directoryIfAny -T ${framed_name}     // directoryIfAny -> com/company/appName
                 echo "Uploaded ${framed_name} to Nexus"
               done
               '''
             }
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)
            error "Failed at 'Nexus Upload of Ear Artifact', exiting now..."
          }
       }

       stage("Default Deployment(${deployEnv})"){
          stage_step="${STAGE_NAME}"
          def UCDProcess="SUCCESS"
          try{
            envName =sh(returnStdout:true, script: """
                source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                echo "\$defaultEnvName"               // defaultEnvName - variable which contains default env value
             """
            )
            envName = envName.trim()
            println(envName)

            buildFileName =sh(returnStdout:true, script: """
                source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                echo "\$envSpecificBuildFileNameVale"    // envSpecificBuildFileNameVale - variable which contains env specific build file name vale
            """
            )
            buildFileName = buildFileName.trim()
            println(buildFileName)

            def comp_name="ucdCompName"
            def applicationProcess="ucdAppProcess"
            def earName="buildArtifact"+""+"${buildFileName}.ear"    //buildArtifact - name will be whatever build artifact mentio in build.xml file
            def warName="buildArtifact"+""+"${buildFileName}.war"
            def folderName="buildArtifact"+""+"${buildFileName}"

            //change ear war name according to envs
            sh'''
            mv $WORKSPACE/build/target-EAR/buildArtifact*.ear $WORKSPACE/build/target-EAR/'''+earName+'''
            mv $WORKSPACE/build/target-EAR/buildArtifact*.war $WORKSPACE/build/target-EAR/'''+warName+'''
            '''
            appDeploy(comp_name, envName, earName, warName, folderName, applicationProcess, UCDProcess, buildFileName)    //appDeploy - function in shared library for reusability
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, deployEnv)
            error "Failed at 'Default Deployment(${deployEnv})', exiting now..."
          }
       }

       stage("${deployEnv} Smoke Test"){
          stage_step="${STAGE_NAME}"
          try{
            envName =sh(returnStdout:true, script: """
                source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                echo "\$defaultEnvName"               // defaultEnvName - variable which contains default env value
             """
            )
            envName = envName.trim()
            println(envName)

            def comp_name="ucdCompName"
            def applicationProcess="ucdAppProcess"
            def wasSysOutLog="valueOfLogFilePath"     //wasSysOutLog - this variable update value in UCD comp variable according to env to trace system log for verification
            def appTimerLog="valueOfTimerLogFilePath"     //appTimerLog - this variable update value in UCD comp variable according to env to trace timer log for verification

            appSmoke(comp_name, wasSysOutLog, appTimerLog, applicationProcess, envName)    //appSmoke - function in shared library for reusability
          }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, deployEnv)
            error "Failed at '${deployEnv} Smoke Test', exiting now..."
          }
       }

       //send success mail
       Subject="Build Success : ${JOB_Name} #${BUILD_NUMBER}" 
       echo "${stage_step}"
       EmailFunction(To, CC, Subject, appName, stage_step, build_status, envName)

       try{
         
       }catch(Exception e){
            Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
            build_status="FAILURE"
            echo "${stage_step}"
            EmailFunction(To, CC, Subject, appName, stage_step, build_status, deployEnv)
            error "Failed at '${deployEnv} Smoke Test', exiting now..."
       }
       finally{
         if(Deploy_status=="FAILURE"){
           currentBuild.result='FAILURE'
         }
         if(Deploy_status=="ABORTED"){
           currentBuild.result='SUCCESS'
         }
       }         
  }catch(Exception e){  
    build_status="FAILURE"
    error "Failed due to some issue, exiting now..."
  }
  
}
