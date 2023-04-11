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
    stage('Build and Push Image') {
        agent any
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
    stage('Run ansible playbook') {
        agent any
        steps {
          sshagent(['ssh-agent-ansible']) {
            sh '''
              sed -i "s/nodejs-app-jenkins:.*/nodejs-app-jenkins:$BUILD_NUMBER/g" ./ansible/docker-deploy.yml
              cat ./ansible/docker-deploy.yml
              ls -la
              rsync -avz --checksum  ./ansible/docker-deploy.yml ubuntu@10.0.0.76:/home/ubuntu/docker-deploy.yml
              ssh ubuntu@10.0.0.76 "cat docker-deploy.yml && ansible-playbook  docker-deploy.yml"
            '''
          }
          
        }
        
    }
    stage('Deploy to Kubernetes') {
        agent {
          kubernetes {
                  cloud 'kubernetes'
                  yaml '''
                    apiVersion: v1
                    kind: Pod
                    metadata:
                      namespace: devops-tools
                    spec:
                      serviceAccountName: jenkins-admin
                      containers:
                      - name: jnlp
                        image: phamsyhung1110/jenkins-agent:3.0
                    '''
          }
      }
        steps {
          container('jnlp') {
            // withKubeConfig([credentialsId: 'jk-k8s']) {
            sh '''
                sed -i "s/tag:.*/tag:$BUILD_NUMBER/g" ./helm/node-app/values.yaml
                helm upgrade --install node-app ./helm/node-app/ -n $k8sNamespace
            '''
            // }
          }
            
        }
    }
  }
}
