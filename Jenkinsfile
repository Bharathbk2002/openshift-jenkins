pipeline {
    agent any

    stages {
        stage('Image-stream') {
            steps {
                script {
                    sh """
                        echo "Creating ImageStream..."
                        oc create imagestream web-application || echo "ImageStream already exists."
                    """
                }
            }
        }
        
        stage('Build configuration') {
            steps {
                script {
                    sh """
                        echo "Applying BuildConfig..."
                        oc apply -f openshift/buildconfig.yaml
                        echo "Starting build..."
                        oc start-build web-app-image --wait
                    """
                }
            }
        }

        stage('Deployment') {
            steps {
                script {
                    sh """
                        echo "Applying DeploymentConfig..."
                        oc apply -f openshift/deploymentconfig.yaml
                    """
                }
            }
        }

        stage('Service') {
            steps {
                script {
                    sh """
                        echo "Applying Service..."
                        oc apply -f openshift/service.yaml
                    """
                }
            }
        }

        stage('Route') {
            steps {
                script {
                    sh """
                        echo "Applying Route..."
                        oc apply -f openshift/route.yaml
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    echo "Waiting for the application to deploy and become accessible..."

                    def appRoute = sh(script: "oc get route web-application -o jsonpath='{.spec.host}'", returnStdout: true).trim()
                    echo "Verifying application at: http://${appRoute}"

                    sh "sleep 10"

                    sh """
                        curl --fail --retry 5 --retry-delay 5 http://${appRoute}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful! Application is live.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }
    }
}
