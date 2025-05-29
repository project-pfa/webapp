pipeline {
    agent any
    environment {
        NVM_DIR = "${env.WORKSPACE}/.nvm"
    }
    stages {
        stage('Preparation') {
            steps {
                checkout scm
                script {
                    commit_id = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                }
            }
        }

        stage('Install Node.js') {
            steps {
                echo "Installation de Node.js via NVM..."
                sh '''
                    export NVM_DIR="$WORKSPACE/.nvm"
                    mkdir -p $NVM_DIR
                    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
                    . $NVM_DIR/nvm.sh
                    nvm install 16
                    nvm alias default 16
                    node -v
                    npm -v
                '''
            }
        }

       stage('Build Angular App') {
    steps {
        dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
            sh '''
                #!/bin/bash
                if [ ! -f "package.json" ]; then
                    echo "Erreur : package.json non trouvé !"
                    exit 1
                fi
                export NVM_DIR="$WORKSPACE/.nvm"
                if [ ! -f "$NVM_DIR/nvm.sh" ]; then
                    echo "Erreur : nvm.sh non trouvé !"
                    exit 1
                fi
                . $NVM_DIR/nvm.sh
                nvm use 16 || { echo "Erreur : nvm use 16 a échoué"; exit 1; }
                node -v || { echo "Erreur : node non trouvé"; exit 1; }
                npm -v || { echo "Erreur : npm non trouvé"; exit 1; }
                npm install || { echo "Erreur : npm install a échoué"; exit 1; }
                npm run build || { echo "Erreur : npm run build a échoué"; exit 1; }
            '''
        }
    }
}

        stage('Check workspace') {
            steps {
                sh 'echo "📂 Répertoire actuel : $(pwd)"'
                sh 'ls -l microservice-source-code/release2/k8s-fleetman-webapp-angular'
            }
        }

        stage('Image Build') {
            steps {
                echo "Construction de l'image Docker..."
                dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
                    sh 'docker build -t ghada13/webapp:${commit_id} .'
                }
                echo "Construction terminée"
            }
        }

        stage('Push Image') {
            steps {
                echo "Pousser vers Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ghada13/webapp:${commit_id}'
                }
                echo "Pousse terminée"
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement vers Kubernetes"
                dir('microservice-source-code/release2/k8s-fleetman-webapp-angular') {
                    sh "sed -i 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|ghada13/webapp:${commit_id}|' manifests/webapp.yaml"
                    sh 'kubectl apply -f manifests/'
                    sh 'kubectl get pods -n default'
                }
                echo "Déploiement terminé"
            }
        }
    }
}
