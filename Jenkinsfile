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
                    docker info || {echo "Docker deamon is not running. "; exit 1; }
                '''
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
                    """
                    }
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

            stage ('Upload Artifacts and Report'){
                steps {
                    // archieve jar file and report for download purpose
                    archiveArtifacts artifacts: 'target/spring-petclinic-4.0.0-SNAPSHOT.jar'
                    archiveArtifacts artifacts: 'target/site/jacoco/**/*'
                    archiveArtifacts artifacts: 'target/surefire-reports/**/*'
                }
            }
     }

    post {
        always {
            echo 'Cleaning up workspace'
            deleteDir()
        }

        success {
            echo 'Project Build succeeded!!!'
        }

        failure {
            echo 'ProjectBuild failed!'
        }
    }
}
