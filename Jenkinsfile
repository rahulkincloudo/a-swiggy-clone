pipeline{
     agent any
     
     tools{
         jdk 'jdk17'
         nodejs 'node16'
     }
     environment {
         SCANNER_HOME=tool 'sonarqube-scanner'
     }
     
     stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/rahulkincloudo/a-swiggy-clone.git'
             }
         }
         stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('SonarQube-Server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CI \
                     -Dsonar.projectKey=Swiggy-CI '''
                 }
             }
         }
         stage("Quality Gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token' 
                 }
             } 
         }
         stage('Install Dependencies') {
             steps {
                 sh "npm install"
             }
         }
         stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
          stage("Docker Build & Push"){
             steps{
                 script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                        sh "docker build -t swiggy-clone ."
                        sh "docker tag swiggy-clone iamrsk/swiggy-clone:latest "
                        sh "docker push iamrsk/swiggy-clone:latest "
                     }
                 }
             }
         }
         stage("TRIVY"){
             steps{
                 sh "trivy image iamrsk/swiggy-clone:latest > trivyimage.txt" 
             }
         }
          stage('Deploy to Kubernetes'){
             steps{
                 script{
                     dir('Kubernetes') {
                         kubeconfig(credentialsId: 'kubernetes', serverUrl: '') {
                         sh 'kubectl delete --all pods'
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                         }   
                     }
                 }
             }
         }/*
          post {
          always {
           emailext attachLog: true,
               subject: "'${currentBuild.result}'",
               body: "Project: ${env.JOB_NAME}<br/>" +
                   "Build Number: ${env.BUILD_NUMBER}<br/>" +
                   "URL: ${env.BUILD_URL}<br/>",
               to: 'ashfaque.s510@gmail.com',                              
               attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }*/



     }
 }
