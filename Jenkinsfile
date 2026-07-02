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
        git branch: "main", url: "git@github.com:Rutvikgalale/register-app_2jul26"
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
  }
}
