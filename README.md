# CICD-Data
How to create CICD pipeline

Dynamically take url, branch and credentails from respective variables
  Add one text file for data in another Git repo : xyz.txt (projectNameNightly.txt or projectNameRelease.txt)

  repositoryURL=sshReleaseRepoURL
  credentialsId=gitSSHRepoCredentials
  destRepo=sshUrlForMasterRepo
  destCredentialsId=jenkinsCredentialsForMasterRepo
  srcFolder=nameOfReleaseRepo(eg Release)
  destFolder=nameOfMasterRepo(eg Master)
  branch=branchName
  #Putting all info together in variable Deployment_Environment: separate by ; for env and details of envs are seaparte by , as "env1;comp1;appProcess1;buildFileName1;smokeTestAppProcess1,env2;comp2;appProcess2;buildFileName2;smokeTestAppProcess2" 
 Deployment_Environment="env1;comp1;appProcess1;buildFileName1;smokeTestAppProcess1,env2;comp2;appProcess2;buildFileName2;smokeTestAppProcess2"
 Manual_Testing="Yes"
 Automated_Testing="No"
 repositoryUrlInfo="sshUrlOfDevOpsRepo"
 branchInfo="refs/heads/branchNameOfDevOpsRepo"
  file_pathinfo="C:/Users/serviceAccountOfDevOps/workspace/TI_RegressionTests"
  authUCDToken="ucdAuthToken"
  emailTo="gitTeamsNotifier;gitTeamsApprovers;gitTeamsDevOps"
  approverRecipient="gitTeamsApproverGroup"
  dbServiceAccount="SRVBDADEVOPS10"
  dbHost="dbHost"
  dbPort="dbPorteg-1521"
  dbServiceName="dbService"
  dbApplicationProcess="ucdAppProcess"
  SonarQualityGate="disable"
  appName="sonarAppName"
  DISTINGUISHER="appName"
  ucdApplicationName="ucdAppName"
  nexusRepo="nexusRepoName"
  nexusRepoGroupName="com.lbg"
  nexusRepoGroupNameWithSlash="com/lbg"
  nexusURL="http://nexusServerNameOrAddr:8081"
  primaryPreProdEnv="PREPROD(RK4-FMO)"
  secondaryPreProdEnv="PREPROD(PB2-FMO)"
  
Add Jenkins file on same repo where txt present

Create nightly and release pipeline/job separately on jenkins and sonar
  Nightly -> 'Checkout', 'SAST analysis', 'JUnit Testing' and 'Compile and Build EAR' stage


