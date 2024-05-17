pipeline {
   agent any
   tools {
       maven "maven3"
       jdk "jdk11"
   }
    environment {
           NEXUS_VERSION = "nexus3"
           NEXUS_PROTOCOL = "http"
           NEXUS_URL = "52.2.103.2:8081"
           NEXUS_REPOSITORY = "vprofile-release"
   	       NEXUS_REPO_ID    = "vprofile-release"
           NEXUS_CREDENTIAL_ID = "nexus-cred"
           ARTVERSION = "${env.BUILD_ID}"
       }

   stages {

      stage("Build") {
         steps {
            sh 'mvn clean install -DskipTests'
         }
      }
      post {
         success {
            echo 'Now Archiving now'
               archiveArtifacts artifacts: '**/target/*.war'
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

      


   }
}
