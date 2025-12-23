pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "srinathsidhu12/java_spring_boot_sample_app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_DEPLOYMENT_NAME = "springboot-demo"
    }
    tools {
       jdk 'JDK-21'
       maven 'Maven-3.9.11'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/srinathsidhu12/Java_spring_boot_app_sonar_analysis.git'	
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
               // Load SonarQube server URL and configuration from Jenkins
               // Retrieve SonarQube Global Analysis Token securely from Jenkins
               withSonarQubeEnv('SonarQube') {
                   withCredentials([string(
                       credentialsId: 'sonarqube-token',
                       variable: 'SONAR_TOKEN'
                )]) {
                // Run SonarQube analysis using Maven Sonar plugin
                // Unique identifier for the project in SonarQube
                // Token used by Jenkins to authenticate with SonarQube
                sh """
                  mvn sonar:sonar \
                    -Dsonar.projectKey=my-project \
                    -Dsonar.login=\$SONAR_TOKEN
                """
              }
           }
         }   
       }
        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                      docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                     sed -i "s|image:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|" ./k8s/k8s-deployment.yaml
                     kubectl apply -f ./k8s/k8s-deployment.yaml
                     kubectl apply -f ./k8s/k8s-service.yaml
                     kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME}
                    """
                  }
             }
         }    
    }

    post {
        success {
            echo " Successfully deployed image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}

