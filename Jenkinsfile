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
          sh '''
          sed -i 's/node-app:.*/node-app:$BUILD_NUMBER/g' ./ansible/dev.inventory
          rsync -i /home/ubuntu/.ssh/id_rsa.pub -avz --checksum ./ansible/dev.inventory ubuntu@10.0.0.76:/home/ubuntu/dev.inventory
          rsync -i /home/ubuntu/.ssh/id_rsa.pub -avz --checksum ./ansible/docker-deploy.yaml ubuntu@10.0.0.76:/home/ubuntu/docker-deploy.yaml
          ssh ubuntu@10.0.0.76 "ansible-playbook -i /home/ubuntu/dev.inventory docker-deploy.yml"
        '''
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
                        image: phamsyhung1110/jenkins-agent:2.0
                    '''
          }
      }
        steps {
          container('jnlp') {
            // withKubeConfig([credentialsId: 'jk-k8s']) {
            sh '''
                kubectl run jenkins-deploy --image $registry/$imageName:$BUILD_NUMBER -n $k8sNamespace
            '''
            // }
          }
            
        }
    }
  }
}
