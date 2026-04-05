pipeline {    
    agent any
    
    tools {
        maven 'Maven3'
    }
    
    stages {

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t myapp:latest .'
            }
        }

        stage('Run Container') {
            steps {
                 bat '''
                        cmd /c "docker stop myapp || exit /b 0"
                        cmd /c "docker rm myapp || exit /b 0"
                        docker run -d --name myapp -p 8081:8080 myapp:latest
                    '''
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
