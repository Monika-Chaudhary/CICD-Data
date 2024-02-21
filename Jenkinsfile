@Library('DevOpsSharedLib')_

def catchBlock(To, CC, Subject, appName, stage_step, build_status, e, DeploymentEnvironment, UCDProcess, Deploy_status, Approver, env){
  if(e.toString()=="org.jenkinsci.plugins.workflow.steps.FlowInterruptedException" || UCDProcess="SUCCESS"){
    echo "${Approver} has aborted in ${DeploymentEnvironment}"
    Deploy_status="ABORTED"
    Subject="Deployment Aborted by ${Approver} : ${JOB_Name} #${BUILD_NUMBER} ${DeploymentEnvironment}"
    echo "${DeploymentEnvironment} deploy status : Aborted"
    EmailFunction(To, CC, Subject, appName, stage_step, Deploy_status, env)
  }else if(e.toString()=="org.jenkinsci.plugins.workflow.steps.FlowInterruptedException" || UCDProcess="FAILURE"){
    Deploy_status="FAILURE"
    Subject="Deployment FAILURE : ${JOB_Name} #${BUILD_NUMBER} ${DeploymentEnvironment}"
    echo "${DeploymentEnvironment} deploy status : Failed, current user is ${Approver}"
    EmailFunction(To, CC, Subject, appName, stage_step, Deploy_status, env)
  }else{
    Deploy_status="FAILURE"
    Subject="Deployment FAILURE : ${JOB_Name} #${BUILD_NUMBER} ${DeploymentEnvironment}"
    echo "${DeploymentEnvironment} deploy status : Failed, current user is ${Approver}"
    EmailFunction(To, CC, Subject, appName, stage_step, Deploy_status, env)
  }
  throw e
}

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
                      //-Dsonar.java.binaries=$WORKSPACE/WebContent/WEB-INF/lib
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
                echo "\$envSpecificBuildFileNameValue"    // envSpecificBuildFileNameVale - variable which contains env specific build file name vale
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

       //parallel deployment with approval from people
       try{
         /*
         emailext to: 'MonikaChaudhary705@gmail; anotherMailId',    //approval people list
         mimeType: 'text/html',
         subject: "Approve: ${appName} code in Test Environments",
         body: """<br>Dear Approver,</br>
<br>Please approve build for Test Environments</br>
<br>'''<a href="${BUILD_URL}input">Click to approve for Test Environments</a>'''</br>
<br><br>Thanks & Regards,<br>
Jenkins Administration""" 
         */

         def receipient='gitApprovalTeam'
         DeploymentApproval(receipient, appName, BUILD_URL)

         parallel(
           "Stage SIT1A": {
             def env =sh(returnStdout:true, script: """
                source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                echo "\$SIT1AEnvName"               // defaultEnvName - variable which contains env value of UCD
             """
             )
             env = env.trim()
             println(env)

             def DeploymentEnvironment="SIT1A"
             def UCDProcess="SUCCESS"
             Approver=BUILD_USER_ID

             stage("Deployment(${DeploymentEnvironment})"){
               def inputuser=""
               try{
                 stage_step="${STAGE_NAME}"
                 approvalMap= input id: 'Deploy_SIT1A', message: 'Please find approval for SIT1A', ok: 'Deploy'
                 Approver=BUILD_USER_ID
                 echo "Approved by ${Approver} in SIT1A"

                buildFileName ="=sit1a"    // to distinguish upload comp based artifact as same comp not be uploaded to UCD

                def comp_name="ucdCompName"
                def applicationProcess="ucdAppProcess"
                def earName="buildArtifact.ear"    //buildArtifact - name will be whatever build artifact mentio in build.xml file
                def warName="buildArtifact.war"
                def folderName="buildArtifact"

                //change ear war name according to envs
                sh'''
                mv $WORKSPACE/build/target-EAR/buildArtifact*.ear $WORKSPACE/build/target-EAR/'''+earName+'''
                mv $WORKSPACE/build/target-EAR/buildArtifact*.war $WORKSPACE/build/target-EAR/'''+warName+'''
                '''
                appDeploy(comp_name, env, earName, warName, folderName, applicationProcess, UCDProcess, buildFileName)    //appDeploy - function in shared library for reusability

                echo "${Approver} is the current user in SIT1A"
                Subject="Deployment Success : ${JOB_Name} #${BUILD_NUMBER} ${DeploymentEnvironment} approved by ${Approver}"
                echo "${DeploymentEnvironment} deploy status : ${Deploy_status}"
                EmailFunction(To, CC, Subject, appName, stage_step, build_status, DeploymentEnvironment)
            }catch(Exception e){
              Approver=BUILD_USER_ID
              catchBlock(To, CC, Subject, appName, stage_step, build_status, e, DeploymentEnvironment, UCDProcess, Deploy_status, Approver, env)
            } 
          }
          stage("${DeploymentEnvironment} Smoke Test"){
            stage_step="${STAGE_NAME}"
            try{

              def comp_name="ucdCompName"
              def applicationProcess="ucdAppProcess"
              def wasSysOutLog="valueOfLogFilePath"     //wasSysOutLog - this variable update value in UCD comp variable according to env to trace system log for verification
              def appTimerLog="valueOfTimerLogFilePath"     //appTimerLog - this variable update value in UCD comp variable according to env to trace timer log for verification

              appSmoke(comp_name, wasSysOutLog, appTimerLog, applicationProcess, envName)    //appSmoke - function in shared library for reusability
            }catch(Exception e){
              Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
              build_status="FAILURE"
              echo "${stage_step}"
              EmailFunction(To, CC, Subject, appName, stage_step, build_status, DeploymentEnvironment)
              error "Failed at '${deployEnv} Smoke Test', exiting now..."
          }
       }
      },

       "Stage SIT1B": {
             def env =sh(returnStdout:true, script: """
                source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                echo "\$SIT1BEnvName"               // defaultEnvName - variable which contains env value of UCD
             """
             )
             env = env.trim()
             println(env)

             def DeploymentEnvironment="SIT1B"
             def UCDProcess="SUCCESS"
             Approver=BUILD_USER_ID

             stage("Deployment(${DeploymentEnvironment})"){
               def inputuser=""
               try{
                 stage_step="${STAGE_NAME}"
                 approvalMap= input id: 'Deploy_SIT1B', message: 'Please find approval for SIT1B', ok: 'Deploy'
                 Approver=BUILD_USER_ID
                 echo "Approved by ${Approver} in SIT1B"

                 buildFileName =sh(returnStdout:true, script: """    // to distinguish upload comp based artifact as same comp not be uploaded to UCD
                   source \$WORKSPACE/Depoly_Env_Name    //Depoly_Env_Name - this file contains env name of UCD in git
                   echo "\$SIT1BbuildFileName"    // envSpecificBuildFileNameVale - variable which contains env specific build file name vale
                 """
                 )
                buildFileName = buildFileName.trim()
                println(buildFileName)   

                def comp_name="ucdCompName"
                def applicationProcess="ucdAppProcess"
                def earName="buildArtifact"+""+"${buildFileName}.ear"   //buildArtifact - name will be whatever build artifact mentio in build.xml file
                def warName="buildArtifact"+""+"${buildFileName}.war"
                def folderName="buildArtifact"+""+"${buildFileName}"

                //change ear war name according to envs
                sh'''
                mv $WORKSPACE/build/target-EAR/buildArtifact*.ear $WORKSPACE/build/target-EAR/'''+earName+'''
                mv $WORKSPACE/build/target-EAR/buildArtifact*.war $WORKSPACE/build/target-EAR/'''+warName+'''
                '''
                appDeploy(comp_name, env, earName, warName, folderName, applicationProcess, UCDProcess, buildFileName)    //appDeploy - function in shared library for reusability

                echo "${Approver} is the current user in SIT1B"
                Subject="Deployment Success : ${JOB_Name} #${BUILD_NUMBER} ${DeploymentEnvironment} approved by ${Approver}"
                echo "${DeploymentEnvironment} deploy status : ${Deploy_status}"
                EmailFunction(To, CC, Subject, appName, stage_step, build_status, DeploymentEnvironment)
            }catch(Exception e){
              Approver=BUILD_USER_ID
              catchBlock(To, CC, Subject, appName, stage_step, build_status, e, DeploymentEnvironment, UCDProcess, Deploy_status, Approver, env)
            } 
          }
          stage("${DeploymentEnvironment} Smoke Test"){
            stage_step="${STAGE_NAME}"
            try{

              def comp_name="ucdCompName"
              def applicationProcess="ucdAppProcess"
              def wasSysOutLog="valueOfLogFilePath"     //wasSysOutLog - this variable update value in UCD comp variable according to env to trace system log for verification
              def appTimerLog="valueOfTimerLogFilePath"     //appTimerLog - this variable update value in UCD comp variable according to env to trace timer log for verification

              appSmoke(comp_name, wasSysOutLog, appTimerLog, applicationProcess, env)    //appSmoke - function in shared library for reusability
            }catch(Exception e){
              Subject="Build Failure : ${JOB_Name} #${BUILD_NUMBER}"  
              build_status="FAILURE"
              echo "${stage_step}"
              EmailFunction(To, CC, Subject, appName, stage_step, build_status, DeploymentEnvironment)
              error "Failed at '${deployEnv} Smoke Test', exiting now..."
          }
         }
       }
      )
         
       }catch(Exception e){
           
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
