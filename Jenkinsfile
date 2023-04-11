pipeline {
  agent none
  // agent {
  //   kubernetes {
  //           cloud 'kubernetes'
  //         }
  //   }
  environment {
    registry = "phamsyhung1110"
    registryCredential = 'docker-hub-credential'
    DOCKERHUB_TOKEN = credentials('docker-hub-credential')
    dockerhub_user = 'phamsyhung1110'
    imageName = "nodejs-app-jenkins"
    k8sNamespace = 'devops-tools'
  }
  stages {
    agent any
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
        agent {
          kubernetes {
                  cloud 'kubernetes'
                  defaultContainer 'kubectl'
                  yaml """
                  apiVersion: v1
                  kind: Pod
                  metadata:
                    labels:
                      app: jenkins-deploy
                  spec:
                    containers:
                    - name: kubectl
                      image: bitnami/kubectl
                      command:
                      - sleep
                      - infinity
                  """
          }
      }
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
