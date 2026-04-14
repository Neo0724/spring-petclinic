pipeline {
    agent any

    environment {
        IMG_NM = "petclinic-app"
    }

    stages {
        // Check Docker Deamon is running
        stage('Check Tools') {
            steps {
                sh '''
                echo "Checking Docker Environment"
                    docker info || { echo "Docker daemon is not running. "; exit 1; }                '''
                }
            }

        stage('Start Database') {
            steps {
                sh 'docker compose down'
                sh 'docker compose up -d postgres'
                sleep 10
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Test and Coverage Report') {
            steps {
                sh './mvnw test jacoco:report'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id1', variable: 'SONAR_TOKEN')]) {
                echo "Performing Static Code Analysis..."
                sh """
                    ./mvnw sonar:sonar \
                    -Dsonar.token=${SONAR_TOKEN} \
                    -Dsonar.analysis.mode=publish
                """
                }
            }
        }

        stage('Upload Build and Test Artifacts') {
            steps {
                junit 'target/surefire-reports/*.xml'
    
                jacoco(
                    execPattern: 'target/jacoco.exec',
                    classPattern: 'target/classes',
                    sourcePattern: 'src/main/java'
                )

                archiveArtifacts artifacts: 'target/spring-petclinic-4.0.0-SNAPSHOT.jar'
                archiveArtifacts artifacts: 'target/site/jacoco/**/*'
                archiveArtifacts artifacts: 'target/surefire-reports/**/*'
                }
            }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMG_NM} .'   
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker stop myapp || true
                docker rm myapp || true
                docker run --name myapp -d -p 8080:8080 ${IMG_NM}
                '''
            }
        }
     }

    post {
        success {
            echo 'Project Build succeeded!!!'
        }

        failure {
            echo 'ProjectBuild failed!'
        }
    }
}
