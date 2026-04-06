pipeline {    
    agent any
    
    stages {

        stage('Build') {
            steps {
                bat './mvnw clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                bat './mvnw test jacoco:report'
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
            jacoco(
                execPattern: 'target/jacoco.exec',
                classPattern: 'target/classes',
                sourcePattern: 'src/main/java'
            )
            archiveArtifacts artifacts: 'target/spring-petclinic-4.0.0-SNAPSHOT.jar', fingerprint: true
            archiveArtifacts artifacts: 'target/site/jacoco/**/*', fingerprint: true
            archiveArtifacts artifacts: 'target/surefire-reports/**/*', fingerprint: true
        }
    }
}
