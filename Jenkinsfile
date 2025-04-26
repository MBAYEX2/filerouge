pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'accestoken' // ID Jenkins Credentials
        DOCKERHUB_USER = 'arafat2'       // ton nom d’utilisateur Docker Hub
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📥 Clonage du dépôt Git"
                checkout scm
            }
        }

        stage('Build & Test Backend (Django)') {
            steps {
                dir('Backend/odc') {
                    echo "⚙️ Création de l'environnement virtuel et test de Django"
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python manage.py test
                    '''
                }
            }
        }

        stage('Build & Test Frontend (React)') {
            steps {
                dir('Frontend') {
                    echo "⚙️ Installation et test du frontend React"
                    sh '''
                        export PATH=$PATH:/var/lib/jenkins/.nvm/versions/node/v22.15.0/bin/
                        npm install
                        npm run build
                       # npm test -- --watchAll=false
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo "🐳 Construction de l'image Docker Backend"
                    sh "docker build -t ${DOCKERHUB_USER}/odc_backend_image:latest -f ./Backend/odc/Dockerfile ./Backend/odc"

                    echo "🐳 Construction de l'image Docker Frontend"
                    sh "docker build -t ${DOCKERHUB_USER}/odc_frontend_image:latest ./Frontend"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                echo "🚀 Envoi des images Docker sur Docker Hub"
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_USER/Backend/odc:latest
                        docker push $DOCKER_USER/Frontend:latest
                    '''
                }
            }
        }
        stage('run'){
            steps{
                dir('cd ..'){
                sh '''
                docker-compose build
                docker-compose up
                #docker run --rm -d -p 8081:8081 ${DOCKERHUB_USER}/Fronten:latest
                '''
                }
            }
        }
    }

    post {
        success {
            mail to: 'mbayex2@gmail.com',
                 subject:"deploiement reussi",
                  body: "l'application a ete deploye avec succes"
        }
        failure {
            mail to: 'mbayex2@gmail.com',
                 subject:"echec du deploiement",
                  body: "veuillez corriger vos erreurs"
        }
    }
}
