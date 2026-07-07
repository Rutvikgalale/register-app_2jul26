pipeline {
  agent { label "jenkins-agent" }
  tools {
    jdk "java17"
    maven "maven3"
  }
  environment{
    app_name="register_app"
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
/*
stage("code quality analysis"){
        steps{
          withSonarQubeEnv('sonar'){ // 'sonar' is the name you configured in Manage Jenkins → System
            withCredentials([string(credentialsId: 'sonar-token', variable: 'sonar_token')]) {
              script{
                def scannerHome = tool 'sonar' //// 'sonar' must match the name in  manage jenkins -> tool configuraion
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=Register-app \
                -Dsonar.sources=. \
                -Dsonar.java.binaries=server/target/classes,webapp/target/webapp/WEB-INF/classes \
                -Dsonar.host.url=http://172.31.47.102:9000 \
                -Dsonar.login=$sonar_token
                """
              }
            }
          }
        }
      }
*/

    stage("Quality gate"){
      steps{
        script{
          waitForQualityGate abortPipeline: false, credentialsId: "Jenkins-sonar-token"
        }
      }
    }
    stage("docker build & push"){
      steps{
        script{
          withCredentials([usernamePassword(credentialsId: "docker", usernameVariable: "docker_user", passwordVariable: "docker_pass")]){
              
              def image_tag = "${docker_user}/${app_name}:${BUILD_NUMBER}"
              sh """
              echo $docker_pass | docker login -u $docker_user --password-stdin
              docker build -t ${image_tag} .
              docker push ${image_tag}
              ip r l
              whoami
              pwd
              id
              docker images
              """
          }
        }
      }
    }
    stage("trivy scanning"){
      steps{
        script{
            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image rutvikg/register_app:${BUILD_NUMBER} --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
        }
      }
    }
    stage("deploy"){
      steps{
        sh """
        docker rm -f ${app_name} || true
        docker run -d --name ${app_name} -p 8080:8080 ${image_tag} 
      }
    }
  }
    post {
  always {
    sh '''
      docker images "${user_name}/${app_name}" --format "{{.Repository}}:{{.Tag}}" |
      grep -v ":$BUILD_NUMBER$" |
      xargs -r docker rmi -f || true

      docker container prune -f || true
    '''
  }
}
}
