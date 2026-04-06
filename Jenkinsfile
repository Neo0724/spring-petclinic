pipeline {    
    agent any

    environment {
        MYSQL_URL="jdbc:mysql://mysql-db:3306/petclinic"
        MYSQL_USER="petclinic"
        MYSQL_PASS="petclinic"
        MYSQL_DB="petclinic"
    }
    
    stages {

        stage('Build') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }

            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests -Dspring.profiles.active=mysql'
                stash name: 'spring-pet-clinic-jar', includes: 'target/*.jar'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                }
            }

            steps {
                sh 'chmod +x mvnw'
                sh './mvnw test jacoco:report'
            }
        }

        stage('Docker Build & Run Container') {
            steps {
                unstash 'spring-pet-clinic-jar'
                bat 'docker compose down'
                bat 'docker compose up -d'
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
