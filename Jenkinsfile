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
       stage('Build Angular App') {
            steps {
                dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        stage('Image Build') {
            steps {
                echo "Building Docker image..."
                dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
                    sh 'docker build -t ghada13/webapp:${commit_id} .'
                }
                echo "Build complete"
            }
        }
        stage('Push Image') {
            steps {
                echo "Pushing to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ghada13/webapp:${commit_id}'
                }
                echo "Push complete"
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to Kubernetes"
                dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
                    sh "sed -i 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|ghada13/webapp:${commit_id}|' manifests/webapp.yaml"
                    sh 'kubectl apply -f manifests/'
                    sh 'kubectl get pods -n default'
                }
                echo "Deployment complete"
            }
        }
    }
}
