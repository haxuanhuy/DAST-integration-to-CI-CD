def scan_type
def target
def rootDir
def example
  pipeline {
   agent any
     stages {
         stage('Pipeline Info') {
                 steps {
                     script {
                        rootDir=pwd()
                        example=load "${rootDir}/profile-scan/parameters.Groovy"
                        scan_type=example.scanTypeProvider()
                        target=example.URLProvider()
                         echo "<--Parameter Initialization---->"
                         echo """
                         The current parameters are:
                             Scan Type: ${scan_type}
                             Target: ${target}
                         """
                     }
                 }
         }
 
         stage('Setting up OWASP ZAP docker container') {
             steps {
                 script {
                         echo "Pulling up last OWASP ZAP container --> Start"
                         sh 'docker pull helloworld0903/test'
                         echo "Pulling up last VMS container --> End"
                         echo "Starting container --> Start"
                         sh """
                         docker run -dt --name helloworld \
                         helloworld0903/test \
                         /bin/bash
                         """
                 }
             }
         }
 
 
         stage('Prepare wrk directory') {
             steps {
                 script {
                         sh """
                             docker exec helloworld \
                             mkdir /zap/wrk
                         """
                     }
                 }
         }
 
 
         stage('Scanning target on owasp container') {
             steps {
                 script {
                     echo "----> scan_type: $scan_type"
                     if(scan_type == "Baseline"){
                         sh """
                             docker exec helloworld \
                             zap-baseline.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                     }
                     else if(scan_type == "APIS"){
                         sh """
                             docker exec helloworld \
                             zap-api-scan.py \
                             -t $target \
                             -f openapi \
                             -r report.html \
                             -I
                         """
                     }
                     else if(scan_type == "Full"){
                         sh """
                             docker exec helloworld \
                             zap-full-scan.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                         //-x report-$(date +%d-%b-%Y).xml
                     }
                     else{
                         echo "Something went wrong..."
                     }
                 }
             }
         }
         stage('Copy Report to Workspace'){
             steps {
                 script {
                     sh '''
                         docker cp helloworld:/zap/wrk/report.html ${WORKSPACE}/report.html
                     '''
                 }
             }
         }
      
         stage('Git push'){
             steps {
                 script {
                     withCredentials([usernamePassword(credentialsId: '877a6730-7c94-41cc-a91c-962a904b67d6',
                                     usernameVariable: 'username',
                                     passwordVariable: 'password')]){
                                       //sh("git status")
                                       sh("git config --global user.email 'fakeemail@gmail.com'")
                                       sh("git config --global user.name 'haxuanhuy'")
                                       sh("git add report.html")
                                       sh("git commit -m 'Add ZAP report'")
                                       sh("git push https://$password@github.com/haxuanhuy/integration.git HEAD:develop")
                                     }
                  }
             }
         }
      
     }
     post {
             always {
                 echo "Removing container"
                 sh '''
                     docker stop helloworld
                     docker rm helloworld
                 '''
             }
         }
 }

