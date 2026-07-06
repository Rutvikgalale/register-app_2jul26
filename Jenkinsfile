pipeline {
  agent { label "jenkins-agent" }
  tools {
    jdk "java17"
    maven "maven3"
  }
  stages{
    stage("Cleanup workspace"){
      steps{
        cleanWs()
      }
    }
    stage("checkout from SCM"){
      steps{
        git branch: "main", url: "https://github.com/Rutvikgalale/register-app_2jul26.git"
      }
    }
    stage("build application "){
      steps{
        sh "mvn clean package"
      }
    }
    stage("test application"){
      steps{
        sh "mvn test"
      }
    }
    stage("Sonarqube Analysis"){
      steps{
        script{
           withSonarQubeEnv(credentialsId: "jenkins-sonar-token"){
               sh "mvn sonar:sonar"
           }
        }
      }
    }
    stage("Quality gate"){
      steps{
        script{
          waitForQualityGate abortPipeline: false, credentialsId: "Jenkins-sonar-token"
        }
      }
    }
  }
}
