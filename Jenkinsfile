pipeline {
  agent any
  environment {
    registry = "https://hub.docker.com/repository/docker/phamsyhung1110/testjenkins/general"
    registryCredential = 'docker-hub-credential'
    DOCKERHUB_TOKEN = credentials('docker-hub-credential')
    dockerhub_user = 'phamsyhung1110'
    imageName = "nodejs-app-jenkins"
    containerPort = 8080
    k8sNamespace = 'devops-tools'
  }
  stages {
    // stage('Checkout') {
    //   steps {
    //     checkout([$class: 'GitSCM', 
    //               branches: [[name: 'test']], 
    //               userRemoteConfigs: [[url: 'https://github.com/your-github-repo.git']]])
    //   }
    // }
    stage('Build and Push Image') {
        when {
                branch 'jenkins-test'
            }
        steps {
            withCredentials([usernamePassword(credentialsId: registryCredential, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            script {
                sh '''
                docker build -t $registry/$imageName:$BUILD_NUMBER .
                docker login -u $dockerhub_user -p $DOCKERHUB_TOKEN
                docker push $registry/$imageName:$BUILD_NUMBER
                '''
            }
            }
        }
    }
    stage('Deploy to Kubernetes') {
        when {
                branch 'jenkins-test'
            }
        steps {
            withKubeConfig([credentialsId: 'jenkins-kubernetes-token']) {
            sh '''
                kubectl set image deployment/$imageName $imageName=$registry/$imageName:$BUILD_NUMBER -n $k8sNamespace
                kubectl rollout status deployment/$imageName -n $k8sNamespace
            '''
            }
        }
    }
  }
}
