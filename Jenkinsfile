pipeline {
    agent { label "prashant" }

    environment {
        IMAGE_NAME = "prashant-app:1.0"
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/prashantver95/django-notes.git'
                echo 'Repository cloned successfully.'
            }
        }

        stage('Check & Install Docker') {
            steps {
                script {
                    echo 'Checking Docker installation...'
                    def dockerCheck = sh(script: 'docker --version', returnStatus: true)

                    if (dockerCheck == 0) {
                        echo '✅ Docker is already installed.'
                        sh 'docker --version'
                    } else {
                        echo '⚠️ Docker not found. Installing...'

                        sh '''
                            sudo apt-get update -y
                            sudo apt-get install -y ca-certificates curl gnupg lsb-release
                            sudo mkdir -p /etc/apt/keyrings
                            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
                            echo \
                              "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
                              $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
                            sudo apt-get update -y
                            sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
                        '''
                        echo '✅ Docker installed.'
                    }

                    // Check if user has Docker permission
                    echo 'Checking Docker socket access...'
                    def dockerSocketAccess = sh(script: 'docker ps', returnStatus: true)
                    if (dockerSocketAccess != 0) {
                        echo '⚠️ Jenkins user lacks Docker permission. Adding user to docker group...'
                        sh '''
                            sudo usermod -aG docker $USER && newgrp docker
                            sudo reboot
                            echo "Added jenkins to docker group. Restart required for changes to take effect."
                        '''
                        echo '⚠️ Please restart the Jenkins agent manually after this build for group changes to apply.'
                    } else {
                        echo '✅ Jenkins user has Docker permissions.'
                    }
                }
            }
        }
        
        stage('Clean and build image') {
            steps {
            
                sh "sudo docker compose down"
                script {
                    echo "Cleaning old Docker image if it exists: ${env.IMAGE_NAME}"
                    sh """
                        if sudo docker image inspect prashantverma95/${env.IMAGE_NAME} > /dev/null 2>&1; then
                            echo "🔄 Removing old image..."
                            sudo docker rmi -f prashantverma95/${env.IMAGE_NAME}
                        else
                            echo "✅ No existing image found."
                        fi
                    """
                }
              echo "🛠️ Building Docker image: ${env.IMAGE_NAME}"
              sh "docker build -t prashantverma95/${env.IMAGE_NAME} ."    
            }
        }
        
        stage("Push to Docker Hub"){ 
            steps{ echo "Pushing theimage to docker hub"
                    withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")])
                    { sh "docker tag prashantverma95/${env.IMAGE_NAME} ${env.dockerHubUser}/${env.IMAGE_NAME}" 
                      sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}" 
                      sh "docker push ${env.dockerHubUser}/${env.IMAGE_NAME}" 
                    }
                } 
            
        }
      
        stage('Run Docker Container') {
            steps {
                echo "Running Docker container from image: ${env.IMAGE_NAME}"
                sh "docker compose up -d"
                echo '✅ Docker container ran successfully.'
            }
        }
    }
}
