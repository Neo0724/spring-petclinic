pipeline {
    agent any

    environment {
        IMG_NM = "petclinic-app"
        DB_URL = "jdbc:postgresql://localhost:5432/petclinic"
        DB_USER = "petclinic"
        DB_PASS = "petclinic"
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
                sleep 2
            }
        }

            stage('Build') {
                steps {
sh '''
./mvnw clean package -DskipTests \
-Dspring.profiles.active=postgres \
-Dspring.datasource.url=jdbc:postgresql://localhost:5432/petclinic
'''                }
            }

            stage('Test and Coverage Report') {
                steps {
sh '''
./mvnw test jacoco:report \
-Dspring.profiles.active=postgres \
-Dspring.datasource.url=jdbc:postgresql://localhost:5432/petclinic \
-Dspring.datasource.username=petclinic \
-Dspring.datasource.password=petclinic
'''                }
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
                    archiveArtifacts artifacts: 'target/sonar/report-task.txt'
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
