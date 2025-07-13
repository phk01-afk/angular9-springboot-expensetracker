pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {
        stage('Verifikacija direktorijuma') {
            steps {
                sh 'echo "📁 Trenutni direktorijum:"'
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

        stage('Run Spring Boot') {
            steps {
                dir('expensetracker') {
                    sh '''
                        echo "✔ Provera .jar fajla:"
                        ls -l target/*.jar || { echo "❌ JAR fajl nije pronađen!"; exit 1; }

                        echo "⚙️  Pokretanje Spring Boot aplikacije..."
                        pkill -f 'expensetracker-v1.jar' || true
                        nohup java -jar target/expensetracker-v1.jar > backend.log 2>&1 &

                        echo "⏳ Čekam da backend startuje..."
                        sleep 25

                        retries=5
                        while [ $retries -gt 0 ]; do
                            response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9091/api/v1/expenses)
                            if [ "$response" -eq 200 ]; then
                                echo "✅ Backend is up and running!"
                                break
                            else
                                retries=$((retries - 1))
                                echo "⏳ Backend not available. Retrying... ($retries attempts left)"
                                sleep 10
                            fi
                        done

                        if [ $retries -eq 0 ]; then
                            echo "❌ Backend failed to start. Please check the logs for more details."
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Health Check Backend') {
            steps {
                script {
                    def retries = 10
                    def success = false
                    while (retries > 0) {
                        def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:9091/api/v1/expenses", returnStdout: true).trim()
                        if (response == '200') {
                            echo "✅ Backend is up and running!"
                            success = true
                            break
                        } else {
                            retries--
                            echo "⏳ Backend check failed. Retrying... (${retries} left)"
                            sleep(5)
                        }
                    }
                    if (!success) {
                        error "❌ Backend health check failed after multiple attempts!"
                    }
                }
            }
        }

        stage('Build & Serve Frontend') {
            steps {
                dir('expense-tracker-frontend') {
                    sh 'npm install'
                    sh 'pkill -f "ng serve" || true'
                    sh 'nohup npx ng serve --host 0.0.0.0 --port 4200 > frontend.log 2>&1 &'
                }
            }
        }
    }
}


