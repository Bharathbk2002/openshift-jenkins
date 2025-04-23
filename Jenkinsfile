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

                    def appRoute = sh(script: "oc get route web-application-route -o jsonpath='{.spec.host}'", returnStdout: true).trim()
                    echo "Verifying application at: https://${appRoute}"

                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
