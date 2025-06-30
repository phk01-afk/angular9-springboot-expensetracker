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
                dir('expensetracker') {  // Putanja do backend direktorijuma
                    sh 'mvn clean install -Dmaven.compiler.arguments="--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED"'
                }
            }
        }

        // Stage za build frontend-a (Angular)
        stage('Build Frontend') {
            steps {
                dir('expense-tracker-frontend') {  // Putanja do frontend direktorijuma
                    sh 'npm install'  // Instalira sve zavisnosti
                    sh 'npx ng build --prod'  // Pravi produkcijsku verziju aplikacije
                }
            }
        }

        // Stage za deploy frontend-a bez sudo
        stage('Deploy Frontend') {
            steps {
                dir('angular9-springboot-expensetracker/expense-tracker-frontend/dist/expense-tracker-frontend') {  // Putanja do generisanog dist direktorijuma
                    sh '''
                        echo "Deploying frontend to /home/jenkins/frontend-dist"
                        mkdir -p /home/jenkins/frontend-dist
                        cp -r ./* /home/jenkins/frontend-dist/
                    '''
                }
            }
        }

        // Stage za pokretanje Spring Boot aplikacije
        stage('Run Spring Boot') {
            steps {
                dir('angular9-springboot-expensetracker/expensetracker') {  // Putanja do backend direktorijuma
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        }

        // Stage za health check aplikacije
        stage('Health Check') {
            steps {
                script {
                    def retries = 10
                    def success = false
                    while (retries > 0) {
                        def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/api/v1/expenses", returnStdout: true).trim()
                        if (response == '200') {
                            echo "Backend is up and running!"
                            success = true
                            break
                        } else {
                            retries--
                            echo "Health check failed. Retrying... (${retries} attempts left)"
                            sleep(10)
                        }
                    }
                    if (!success) {
                        error "Health check failed after multiple attempts!"
                    }
                }
            }
        }
    }
}
