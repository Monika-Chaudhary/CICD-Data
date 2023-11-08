# CICD-Data
How to create CICD pipeline

Dynamically take url, branch and credentails from respective variables
  Add one text file for data in another Git repo : xyz.txt (projectNameNightly.txt or projectNameRelease.txt)

  repositoryURL=sshURL
  credentialsId=gitSSHCredentials
  branch=branchName

Add Jenkins file on same repo where txt present

Create nightly and release pipeline/job separately on jenkins and sonar
  Nightly -> 'Checkout', 'SAST analysis', 'JUnit Testing' and 'Compile and Build EAR' stage


