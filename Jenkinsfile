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
              checkout([$class: 'GitSCM', branches: [[name: 'refs/heads/branchNameWhereJenkinsfile]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout'], [$class: 'WipeWorspace']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'credentials', refspec: '+refs/heads/branchNameWhereJenkinsfile:refs/remotes/origin/branchNameWhereJenkinsfile', url: 'sshUrlWhereJenkinsAndTxtRepo']]])
              
              def githubRepoURL =sh(returnStdout:true, script: """
                source \$WORKSPACE/xyztxtFile.txt
                echo "\$repositoryURL"
              """
              )
              githubRepoURL = githubRepoURL.trim()
              println(githubRepoURL)
          }catch(Exception e){
              
          }
      }
      
  }catch(Exception e){
      
  }
  
}
