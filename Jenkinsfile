pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                // Obtener codigo del repositorio
                git 'https://github.com/alfonsinmoi/CP1.1.git'
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
                    python3 -m pytest --junitxml=result-unit.xml test/unit/
                '''
            }
        }

        stage('Rest Tests') {
            steps {
                sh '''
                    # Iniciar Flask
                    export FLASK_APP=${WORKSPACE}/app/api.py
                    python3 -m flask run --host=0.0.0.0 --port=5050 &
                    FLASK_PID=$!

                    # Iniciar Wiremock
                    java -jar /Users/macminimoi/wiremock/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                    WIREMOCK_PID=$!

                    # SOLUCION RETO 4: Esperar activamente a que los servicios esten listos
                    # en lugar de usar sleep fijo que puede fallar en sistemas lentos
                    echo "Esperando a que Flask este listo..."
                    for i in $(seq 1 30); do
                        if curl -s http://localhost:5050/ > /dev/null 2>&1; then
                            echo "Flask listo despues de $i intentos"
                            break
                        fi
                        sleep 1
                    done

                    echo "Esperando a que Wiremock este listo..."
                    for i in $(seq 1 30); do
                        if curl -s http://localhost:9090/__admin/ > /dev/null 2>&1; then
                            echo "Wiremock listo despues de $i intentos"
                            break
                        fi
                        sleep 1
                    done

                    # Verificar que ambos servicios estan funcionando
                    curl -s http://localhost:5050/ || (echo "ERROR: Flask no responde" && exit 1)
                    curl -s http://localhost:9090/__admin/ || (echo "ERROR: Wiremock no responde" && exit 1)

                    # Ejecutar tests
                    export PYTHONPATH=${WORKSPACE}
                    python3 -m pytest --junitxml=result-rest.xml test/rest/
                '''
            }
        }
    }

    post {
        always {
            junit 'result*.xml'
            sh '''
                pkill -f "flask" || true
                pkill -f "wiremock" || true
            '''
        }
    }
}
