pipeline {
  agent { label "jenkins-agent" }
  tools {
    jdk "java17"
    maven "maven3"
  }
  environment{
    app_name="register-app"
    docker_user="docker_user"
  }
  stages{
    stage("Cleanup workspace"){
      steps{
        cleanWs(
           deleteDirs: true,
           disableDeferredWipeout: true,
           notFailBuild: true
        )
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
    stage("SonarQube Analysis"){
      steps {
        script {   
          withSonarQubeEnv(credentialsId: 'jenkins-sonar-token') { 
            sh "mvn sonar:sonar"
          }
        }
      }
    }
    stage("Quality Gate"){
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-token'
        }
      }
    }
    stage("docker build & push"){
      steps{
        script{
          withCredentials([usernamePassword(credentialsId: "docker", usernameVariable: "docker_user", passwordVariable: "docker_pass")]){
              sh '''
              docker rmi -f $(docker images -aq)
              echo $docker_pass | docker login -u $docker_user --password-stdin
              docker build -t "${docker_user}/${app_name}:${BUILD_NUMBER}" .
              docker push "${docker_user}/${app_name}:${BUILD_NUMBER}"
              '''
          }
        }
      }
    }
    stage("trivy scanning"){
      steps{
        script{
          sh '''
             # Remove Trivy cache
            [ ! -d "$WORKSPACE/.trivy-cache" ] && mkdir -p "$WORKSPACE/.trivy-cache"
            chmod -R 755 $WORKSPACE/.trivy-cache
          '''
          sh """
            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v -v /tmp/trivy-cache:/root/.cache/trivy aquasec/trivy image rutvikg/register-app:${BUILD_NUMBER} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table
          """
        }
      }
    }
  }

    post {
      always {
        cleanWs(
        deleteDirs: true,
        disableDeferredWipeout: true,
        notFailBuild: true
        )
      }
    }
}
