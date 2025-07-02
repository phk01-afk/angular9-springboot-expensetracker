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
                    sh 'mvn clean install -Dmaven.test.skip=true -Dmaven.compiler.arguments="--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED"'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('expense-tracker-frontend') {
                    sh 'npm install'
                    sh 'ng serve --open'
                }
            }
        }

        stage('Run Spring Boot') {
            steps {
                script {
                    sh '''
                        echo "Zaustavljam prethodnu instancu backend-a (ako postoji)..."
                        pkill -f 'java -jar' || true

                        echo "Pokrećem backend..."
                        cd expensetracker
                        nohup java -jar target/*.jar > ../../backend.log 2>&1 &
                    '''
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
