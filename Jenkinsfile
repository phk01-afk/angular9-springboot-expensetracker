pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {
        // Stage za verifikaciju trenutnog direktorijuma
        stage('Verifikacija direktorijuma') {
            steps {
                sh 'echo "Trenutni direktorijum:"'
                sh 'pwd'
                sh 'ls -l'
            }
        }

        // Stage za build backend-a (Spring Boot)
        stage('Build Backend') {
            steps {
                dir('expensetracker') {  // Ovdje je promenjeno u pravi folder
                    sh 'mvn clean install'
                }
            }
        }

        // Stage za build frontend-a (Angular)
        stage('Build Frontend') {
            steps {
                dir('angular9-springboot-expensetracker') {
                    sh 'npm install'
                    sh 'npx ng build --prod'
                }
            }
        }

        // Stage za pokretanje Spring Boot aplikacije
        stage('Run Spring Boot') {
            steps {
                dir('expensetracker') {  // Ovdje je promenjeno u pravi folder
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        }

        // Stage za health check aplikacije
        stage('Health Check') {
            steps {
                script {
                    sleep(10)
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/api/v1/expenses", returnStdout: true).trim()
                    if (response != '200') {
                        error "Health check failed! Status code: ${response}"
                    } else {
                        echo "Backend is up and running!"
                    }
                }
            }
        }
    }
}


