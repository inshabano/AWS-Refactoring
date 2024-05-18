def COLOR_MAP = [
      'SUCCESS' = 'good',
      'FAILURE' = 'danger',
]

pipeline {
   agent any
   tools {
       maven "maven3"
       jdk "jdk11"

   }
    environment {
           NEXUS_VERSION = "nexus3"
           NEXUS_PROTOCOL = "http"
           NEXUS_URL = "172.31.61.230:8081"
           NEXUS_REPOSITORY = "vprofile-release"
   	       NEXUS_REPO_ID    = "vprofile-release"
           NEXUS_CREDENTIAL_ID = "nexus-cred"
           ARTVERSION = "${env.BUILD_ID}+${env.BUILD_TIMESTAMP}"

           registryCredential = 'ecr:us-east-1:ECRRole_AWS'
           registry = "637423320391.dkr.ecr.us-east-1.amazonaws.com/vprofile_img"
           registryURL = "http://637423320391.dkr.ecr.us-east-1.amazonaws.com"

           scannerHome = tool 'sonar-4.7'
       }

   stages {

      stage("Build") {
         steps {
            sh 'mvn clean install -DskipTests'
         }

      post {
         success {
            echo 'Now Archiving now'
               archiveArtifacts artifacts: '**/target/*.war'
         }
      }
      }
      stage("unit test"){
        steps{
             sh "mvn test"
        }
      }

      stage("Integration test"){
        steps{
           sh 'mvn verify -DskipUnitTests'
        }
      }

      stage("code analysis test with checkstyle"){
        steps{
             sh 'mvn checkstyle:checkstyle'
        }
      }

      stage("sonarqube code analysis"){
         steps{
            withSonarQubeEnv('sonar') {
                           sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                               -Dsonar.projectName=Vprofile \
                               -Dsonar.projectVersion=1.0 \
                               -Dsonar.sources=src/ \
                               -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                               -Dsonar.junit.reportsPath=target/surefire-reports/ \
                               -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                               -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

         }
      }
      stage('Quality Gate'){
        steps{
           timeout(time: 10, unit: 'MINUTES') {
             waitForQualityGate abortPipeline: true
           }
       }
      }

      stage('Build Image'){
         steps{
           script{
              dockerImage = docker.build( registry + ":$BUILD_NUMBER", "./Docker-files/Dockerfile")
           }
         }
       }

      stage("Publish to Nexus Repository Manager") {
         steps {
           script {
              pom = readMavenPom file: "pom.xml";
              filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
              echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
              artifactPath = filesByGlob[0].path;
              artifactExists = fileExists artifactPath;
              if(artifactExists) {
                echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                nexusArtifactUploader(
                  nexusVersion: NEXUS_VERSION,
                  protocol: NEXUS_PROTOCOL,
                  nexusUrl: NEXUS_URL,
                  groupId: pom.groupId,
                  version: ARTVERSION,
                  repository: NEXUS_REPOSITORY,
                  credentialsId: NEXUS_CREDENTIAL_ID,
                  artifacts: [
                     [artifactId: pom.artifactId,
                     classifier: '',
                     file: artifactPath,
                     type: pom.packaging],
                     [artifactId: pom.artifactId,
                     classifier: '',
                     file: "pom.xml",
                     type: "pom"]
                     ]
                );
              }
      		  else {
                  error "*** File: ${artifactPath}, could not be found";
              }
           }
         }
      }
   }
   post{
     always{
       echo "slack notification"
       slackSend channel : '#jenkinscicd',
                color : COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.Job_NAME} build ${env.BUILD_NUMBER} \n More info here: ${env.BUILD_URL}"
     }
   }
}
