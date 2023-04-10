pipeline {
  agent { labels "kubeagent"}
     
  environment {
    registry = "phamsyhung1110"
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
        // when {
        //         branch 'jenkins-test'
        //     }
        steps {
            withCredentials([usernamePassword(credentialsId: registryCredential, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            script {
                sh '''
                docker build -t $registry/$imageName:$BUILD_NUMBER .
                docker login -u $dockerhub_user -p $DOCKER_PASSWORD
                docker push $registry/$imageName:$BUILD_NUMBER
                '''
            }
            }
        }
    }
    stage('Deploy to Kubernetes') {
        // when {
        //         branch 'jenkins-test'
        //     }
        steps {
            withKubeConfig([credentialsId: 'jk-k8s']) {
            sh '''
               kubectl run jenkins-deploy --image $registry/$imageName:$BUILD_NUMBER -n $k8sNamespace
            '''
            }
        }
    }
  }
}
