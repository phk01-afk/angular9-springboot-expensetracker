pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {
        stage('Provera strukture') {
            steps {
                sh 'echo "SADRŽAJ ROOT-a:"'
                sh 'ls -lah'
                sh 'echo "PRONALAZAK pom.xml:"'
                sh 'find . -name "pom.xml"'
            }
        }

        stage('Build Backend') {
            steps {
                dir('angular9-springboot-expensetracker') {
                    sh 'mvn clean install'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('angular9-springboot-expensetracker') {
                    sh 'npm install'
                    sh 'npx ng build --prod'
                }
            }
        }

        stage('Run Spring Boot') {
            steps {
                dir('angular9-springboot-expensetracker') {
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        }

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

