pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    environment {
        BACKEND_DIR = 'expensetracker-backend'
        FRONTEND_DIR = 'expensetracker-frontend'
    }

    stages {
        stage('Clone') {
            steps {
                echo "Cloning repo..."
                // Automatically cloned via SCM config
            }
        }

        stage('Build Backend') {
            steps {
                    sh 'mvn clean install'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                    sh 'npm install'
                    sh 'ng build --prod'
                }
            }
        

        stage('Run Spring Boot App') {
            steps {
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        

        stage('Health Check') {
            steps {
                script {
                    sleep(10) // wait for app to start
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

