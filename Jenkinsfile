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
            steps {
                bat './mvnw clean package -DskipTests "-Dspring.profiles.active=mysql"'
            }
        }

        stage('Test') {
            steps {
                bat './mvnw test jacoco:report'
            }
        }

        stage('Docker Build & Run Container') {
            steps {
                bat 'docker compose down'
                bat 'docker compose -f docker-compose.yml up -d'
            }
        }
    }

    post {
        success {
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
