pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test + Coverage') {
            steps {
                sh 'mvn test jacoco:report'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh "docker stop myapp || true"
                sh "docker rm myapp || true"
                sh 'docker run --name myapp -d -p 8080:8080 myapp:latest'
            }
        }
    }

    post {
        always {
            sh 'echo Checking target folder...'
            sh 'ls -l target || true'
    
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
