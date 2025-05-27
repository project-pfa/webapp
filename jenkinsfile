pipeline {
    agent any
    stages {
        stage('Preparation') {
            steps {
                checkout scm
                script {
                    commit_id = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                }
            }
        }
        stage('Image Build') {
            steps {
                echo "Building......"
                sh 'docker build -t webapp:${commit_id} .'
                echo "build complete"
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to Kubernetes"
                sh "sed -i 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|webapp:${commit_id}|' ./manifests/webapp.yaml"
                sh 'kubectl apply -f ./manifests/'
                sh 'kubectl get all'
                echo "deployment complete"
            }
        }
    }
}
