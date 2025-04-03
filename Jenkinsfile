pipeline {
    agent any

    stages {
        // Etapa 1: Obtener código del repositorio
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tu-usuario/flask-api.git'
            }
        }

        // Etapa 2: Ejecutar pruebas unitarias
        stage('Test') {
            steps {
                sh 'pytest -v tests/'
            }
        }

        // Etapa 3: Construir imagen Docker
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("tu-usuario/flask-api:${env.BUILD_ID}")
                }
            }
        }

        // Etapa 4: Subir imagen a Docker Hub (opcional)
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        dockerImage.push()
                    }
                }
            }
        }

        // Etapa 5: Desplegar en la VM Linux
        stage('Deploy') {
            steps {
                sshagent(['vm-ssh-credentials']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no usuario@ip-de-tu-vm \
                        "docker pull tu-usuario/flask-api:${env.BUILD_ID} && \
                         docker stop flask-api || true && \
                         docker rm flask-api || true && \
                         docker run -d --name flask-api -p 5000:5000 tu-usuario/flask-api:${env.BUILD_ID}"
                    """
                }
            }
        }
    }

    // Notificaciones (opcional)
    post {
        success {
            slackSend channel: '#devops', message: '✅ Pipeline exitoso! API desplegada.'
        }
        failure {
            slackSend channel: '#devops', message: '❌ Pipeline falló! Revisar Jenkins.'
        }
    }
}