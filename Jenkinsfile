pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Bharathbk2002/Podman_CreateImage_PushImage.git'
        IMAGE_NAME = 'web-app'  
        IMAGE_TAG = 'latest'  
        REGISTRY_URL = 'docker.io'  
        REGISTRY_CREDENTIALS = 'dockerhub-credentials'
    }
    

    stages {
        stage('Checkout') {
            steps {
                git url: REPO_URL, branch: 'main'
            }
        }
        
        stage('Build Image and test image') {
            parallel
            {
            stage('Build Image')
            {
            steps {
                script {
                    sh """
                        podman build --layers=true -t bharathbk02/${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh """
                        podman run --rm bharathbk02/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        }
        }
        
        stage('Push Image') {
            steps {
                script {
                    
                    withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')]) {
                        sh """
                            podman login -u $REGISTRY_USER -p $REGISTRY_PASS ${REGISTRY_URL}
                            podman push bharathbk02/${IMAGE_NAME}:${IMAGE_TAG} 
                        """
                    }
                }
            }
        }
        stage('Deployment')
        {
            steps
            {
                script
                {
                    sh """
                        podman rm ${IMAGE_NAME}
                        podman run -d -p 4002:4000 --name ${IMAGE_NAME} bharathbk02/${IMAGE_NAME}:${IMAGE_TAG}
                        curl http://localhost:4002
                    """
                }
            }
        }

        
    

    post {
        success {
            echo 'Image built,tested,pushed and deployed locally and to cloud VM Sucessfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
  }
}
