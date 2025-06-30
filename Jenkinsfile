pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {
        // ✅ Provera trenutnog direktorijuma
        stage('Verifikacija direktorijuma') {
            steps {
                sh '''
                    echo "Trenutni direktorijum:"
                    pwd
                    ls -la
                '''
            }
        }

        // ✅ Build backend (Spring Boot)
        stage('Build Backend') {
            steps {
                dir('expensetracker') {
                    sh 'mvn clean install -Dmaven.test.skip=true'
                }
            }
        }

        // ✅ Build frontend (Angular)
        stage('Build Frontend') {
            steps {
                dir('expense-tracker-frontend') {
                    sh '''
                        npm install
                        npx ng build --prod
                    '''
                }
            }
        }

        // ✅ Deploy frontend fajlova
        stage('Deploy Frontend') {
            steps {
                dir('angular9-springboot-expensetracker/expense-tracker-frontend/dist/expense-tracker-frontend') {
                    sh '''
                        echo "Deploying frontend to /var/www/html"
                        mkdir -p /var/www/html
                         cp -r ./* /var/www/html/
                    '''
                }
            }
        }

        // ✅ Pokretanje backend aplikacije
        stage('Run Spring Boot') {
            steps {
                dir('angular9-springboot-expensetracker/expensetracker') {
                    sh '''
                        echo "Zaustavljanje postojeće aplikacije ako postoji..."
                        pkill -f 'java.*expensetracker' || true

                        echo "Pokretanje nove instance aplikacije na portu 8081"
                        nohup java -jar target/*.jar --server.port=8081 > springboot.log 2>&1 &
                    '''
                }
            }
        }

        // ✅ Health check backend aplikacije
        stage('Health Check') {
            steps {
                script {
                    def retries = 10
                    def success = false
                    while (retries > 0) {
                        def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8081/api/v1/expenses", returnStdout: true).trim()
                        if (response == '200') {
                            echo "✅ Backend is up and running!"
                            success = true
                            break
                        } else {
                            retries--
                            echo "Health check failed. Retrying... (${retries} attempts left)"
                            sleep(10)
                        }
                    }
                    if (!success) {
                        error "❌ Health check failed after multiple attempts!"
                    }
                }
            }
        }
    }
}
