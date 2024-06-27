pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  environment {
        BACKEND_DIR = 'lms-backend'
    }
   stages {
        stage('Checkout') {
            steps {
                echo 'Passed' // Just for demonstration; you can remove this line
                git 'https://github.com/Monish247/library-management-system.git',branch: 'main'
            }
        }
   }   
        stage('Backend build and test') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = 'http://192.168.1.9:9000'
            }
            steps {
                withCredentials([string(credentialsId: 'Sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                 sh "cd ${BACKEND_DIR} && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                }   
            }
        } 
        stage('Docker build backend') {
            environment {
                BACKEND_Docker_image = "monish247/library_management_system:${BUILD_NUMBER}-backend"
            }
            steps {
                script {
                    sh "cd ${BACKEND_DIR} && docker build -t ${BACKEND_Docker_image} ."
                    echo "Docker image ${BACKEND_Docker_image} built successfully"
                    def dockerImage = docker.image("${BACKEND_Docker_image}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        echo 'Pushing Docker image...'
                        dockerImage.push()
                        echo 'Docker image pushed successfully'
                    }
                }
            }
        }

}       