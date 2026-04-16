pipeline {
    agent any

    stages {

        stage ('Check Tools') {
            steps {
                // check docker deamon is running
                sh '''
                echo "Checking Docker Environment"
                    docker info || { echo "Docker daemon is not running. "; exit 1; }                '''
            }
        }

        stage('Initialize and Build Project') {
            parallel {

                // stop exisiting application in docker
                stage ('Stop Exisiting Application') {
                    steps {
                        sh 'docker compose down'
                    }
                }

                // build project
                stage('Build') {
                    steps {
                        sh './mvnw clean package -DskipTests'
                    }
                }
            }
        }

        stage ('Test and Quality Analysis') {
            parallel {
                // testing and generate necessary report
                stage('Test and Coverage Report') {
                    steps {
                        sh './mvnw test jacoco:report surefire-report:report-only'
                    }
                }

                // SonarQube Analysis
                stage('SonarQube Analysis') {
                    steps {
                        withCredentials([string(credentialsId: 'sonar_id1', variable: 'SONAR_TOKEN')]) {
                            sh "./mvnw sonar:sonar -Dsonar.token=${SONAR_TOKEN} -Dsonar.projectKey=petclinic"
                        }
                    }
                }
            }
        }

        // Run application docker container
        // Close the previous running container, rebuild the application image and start the container
        stage('Build Image and Run Project') {
            steps {
                sh '''
                docker compose down
                docker compose build petclinic
                docker compose up postgres petclinic     
                '''
            }
        }
     }

    post {
        always {

            // get surefire reports
            junit 'target/surefire-reports/*.xml'
    
            // generate jacoco report
            jacoco(
                execPattern: 'target/jacoco.exec',
                classPattern: 'target/classes',
                sourcePattern: 'src/main/java'
            )

            // archieve report for download
            archiveArtifacts artifacts: 'target/spring-petclinic-4.0.0-SNAPSHOT.jar'
            archiveArtifacts artifacts: 'target/site/jacoco/**/*'
            archiveArtifacts artifacts: 'target/surefire-reports/**/*'
            archiveArtifacts artifacts: 'target/reports/surefire-report.html'
        }

        // success message
        success {
            echo 'Project Build succeeded!'
        }

        // failure message
        failure {
            echo 'Project Build failed!'
        }
    }
}
