pipeline {
      agent any
    // agent {
    //      docker {
    //      image 'node:latest'
    //      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    //  }
    // }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/praveenece431/youtube-clone.git']])
            }
        }
    
        stage('Build') {
            steps {
                  sh '''
                    echo *** Building the Nodejs Code ****** 
                    hostname
                    pwd
                    ls -l
                    echo "Directly building in the docker image
                    sleep 10"
                  '''
            }
        }
        
        stage('Unit Tests') {
            steps {
                  sh 'echo "Running dummy unit tests. No unit tests are configured"'
                  sh 'sleep 5'
            }
        }
        
        stage('SonarQube Analysis') {
           environment {
           SONAR_URL = "http://172.176.73.21:9000"
           }
           steps {
                 sh 'echo "Dummy sonar analysis"'
                 sh 'sleep 5'
           }
         }
        
        // stage('OWASP Scan') {
        //   steps {
        //       dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
        //       dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //   }
        // }
        
        stage('Docker build') {
          environment {
          DOCKER_IMAGE = "$ECR_REGISTRY_ID/youtube:${BUILD_NUMBER}"
          }
           steps {
           script {
              sh 'docker build -t ${DOCKER_IMAGE} .'
           }
        }
      }
      
       stage('Push Docker Image to ECR') {
         environment {
           DOCKER_IMAGE = "$ECR_REGISTRY_ID/youtube:${BUILD_NUMBER}"
         }
         steps {
              withAWS(credentials: 'AWS_CREDENTIALS_ID', region: 'us-east-1') {
                  sh 'echo "*** aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REGISTRY_ID" ***'
                  sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REGISTRY_ID'
                  //sh 'docker tag $DOCKER_IMAGE $DOCKER_IMAGE'
                  sh 'docker push $DOCKER_IMAGE'
              }
         }
        }
        
        stage('Deploy to AWS EKS Kubernetes') {
            environment {
              EKS_CLUSTER_NAME = "dev"
            }
      steps {
             withAWS(credentials: 'AWS_CREDENTIALS_ID', region: 'us-east-1') {	
                script {
                  sh '''
                    sed -i "s/tagversion/${BUILD_NUMBER}/g" deploy-service.yaml
                    echo "** Print the certificate **"
                    echo "$kube"|base64 -d > /tmp/config
                    kubectl apply -f deploy-service.yaml --kubeconfig /tmp/config
                    cat deploy-service.yaml
                    sleep 20
                    kubectl get pods -n dev-env --kubeconfig /tmp/config
                  '''
              }
              }
      }
    }

  }
}
