pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {

        stage('Verifikacija direktorijuma') {
            steps {
                sh 'echo "Trenutni direktorijum:"'
                sh 'pwd'
                sh 'ls -l'
            }
        }

        stage('Build Backend') {
            steps {
                dir('expensetracker') {
                    sh 'mvn clean install -Dmaven.compiler.arguments="--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED" -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('expense-tracker-frontend') {
                    sh 'npm install'
                    sh 'npx ng build --prod'
                }
            }
        }

        stage('Run Backend') {
            steps {
                dir('expensetracker') {
                    sh 'nohup java -jar target/*.jar &'
                }
            }
        }

        stage('Run Frontend') {
            steps {
                dir('expense-tracker-frontend') {
                    sh 'nohup npx ng serve --port 4200 --host 0.0.0.0 &'
                }
            }
        }

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
