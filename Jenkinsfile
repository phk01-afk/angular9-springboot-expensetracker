pipeline {
    agent any

    tools {
        maven 'Maven 3'
        nodejs 'Node 12'
    }

    stages {

        stage('Kloniraj Repozitorijum') {
            steps {
                echo "📥 Kloniram GitHub repozitorijum..."
                dir('angular9-springboot-expensetracker') {
                    deleteDir()
                }
                git url: 'https://github.com/phk01-afk/angular9-springboot-expensetracker.git', branch: 'patch-1'
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
                    '''
                }
            }
        }


        
        stage('Build & Serve Frontend') {
            steps {
                dir('angular9-springboot-expensetracker/expense-tracker-frontend') {
                    sh 'npm install'
                    sh 'pkill -f "ng serve" || true'
                    sh 'nohup npx ng serve --host 0.0.0.0 --port 4200 > frontend.log 2>&1 &'
                }
            }
        }
    }
}

        stage('Health Check Backend') {
            steps {
                script {
                    def retries = 5
                    def success = false
                    while (retries > 0) {
                        def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/api/v1/expenses", returnStdout: true).trim()
                        if (response == '200') {
                            echo "✅ Backend is up and running!"
                            success = true
                            break
                        } else {
                            retries--
                            echo "⏳ Backend nije spreman. Pokušavam ponovo... (${retries} pokušaja ostalo)"
                            sleep(10)
                        }
                    }
                    if (!success) {
                        error "❌ Backend health check nije uspeo nakon više pokušaja!"
                    }
                }
            }
        }

      
     
