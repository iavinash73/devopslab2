pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}" // Adjust if necessary
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // Replace with your Docker Hub credentials ID
        DOCKER_USERNAME = 'avinash73' // Replace with your Docker Hub username
    }

    tools {
        maven 'Maven 3.x' // Use the name you provided in Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm // Checkout the source code from the configured SCM
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests' // Build the project and skip tests
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test' // Run tests
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "Logging into Docker..."
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    def dockerImage = docker.build("${DOCKER_USERNAME}/another-app:${env.BUILD_NUMBER}") // Build the Docker image
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_USERNAME}/another-app:${env.BUILD_NUMBER}").push() // Push the image to Docker Hub
                        docker.image("${DOCKER_USERNAME}/another-app:${env.BUILD_NUMBER}").push("latest") // Optionally push as latest
                    }
                }
            }
        }

        stage('Deploy') { // Start of Deploy stage
            steps {
                script {
                    echo "Cloning deployment repository..."
                    // Remove existing directory if it exists
                    sh '''
                        if [ -d "microservices-deployment" ]; then
                            echo "Removing existing microservices-deployment directory..."
                            rm -rf microservices-deployment
                        fi
                        git clone https://github.com/iavinash73/microservices-deployment.git
                    ''' // Clone the deployment repo

                    dir('microservices-deployment') { // Change directory to the cloned repo
                        echo "Running Docker Compose to start services..."
                        sh 'docker-compose up -d' // Start services using Docker Compose
                    }
                }
            }
        } // End of Deploy stage

    } // End of stages

    post {
        always {
            script {
                echo "Post-build actions can be added here if needed."
                // Teardown step has been removed.
            }
        }
    } // End of post actions

} // End of pipeline
