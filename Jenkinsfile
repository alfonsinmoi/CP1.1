pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                // Obtener codigo del repositorio
                git 'https://github.com/anieto-unir/helloworld.git'
                echo "WORKSPACE: ${WORKSPACE}"
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                echo 'No hay compilacion, es Python'
                sh 'ls -la'
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    export PYTHONPATH=${WORKSPACE}
                    pytest --junitxml=result-unit.xml test/unit/
                '''
            }
        }

        stage('Rest Tests') {
            steps {
                sh '''
                    export FLASK_APP=${WORKSPACE}/app/api.py
                    flask run --host=0.0.0.0 --port=5000 &
                    sleep 5

                    java -jar /Users/macminimoi/wiremock/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                    sleep 5

                    export PYTHONPATH=${WORKSPACE}
                    pytest --junitxml=result-rest.xml test/rest/
                '''
            }
        }
    }

    post {
        always {
            junit 'result*.xml'
            sh '''
                pkill -f "flask run" || true
                pkill -f "wiremock" || true
            '''
        }
    }
}
