pipeline {
    agent any


    environment {
        COMPOSE_PROJECT_NAME = 'pmng'
        COMPOSE_FILE         = 'docker-compose.yml'
        ENV_FILE             = credentials('pmng-env')
    }


    // fases - etapas
    stages {
        stage('Parar Servicios') {
            steps {
                echo 'Deteniendo contenedores'
                // block string para ejecutar varios comandos a la vez
                sh '''
                    docker compose -p ${COMPOSE_PROJECT_NAME} -f ${COMPOSE_FILE} --env-file ${ENV_FILE} down || true
                    docker rm -f pmng-cliente pmng-servicios || true
                '''
            }
        }


        stage('Eliminar imagenes') {
            steps {
                echo 'Eliminando imagenes antiguas'
                sh '''
                    IMAGES=$(docker compose -p ${COMPOSE_PROJECT_NAME} -f ${COMPOSE_FILE} --env-file ${ENV_FILE} config --images | sort -u)
                    for IMAGE in $IMAGES; do
                        echo "Eliminando imagen $IMAGE"
                        docker rmi "$IMAGE" || true
                    done
                '''
            }
        }


        stage('Construir las imagenes') {
            steps {
                echo 'Construyendo imagenes'
                sh '''
                    docker compose -p ${COMPOSE_PROJECT_NAME} -f ${COMPOSE_FILE} --env-file ${ENV_FILE} build --no-cache
                '''
            }
        }


        stage('Levantar servicios') {
            steps {
                echo 'Generando contenedores'
                sh '''
                    docker network inspect pmng-net >/dev/null 2>&1 || docker network create pmng-net
                    docker compose -p ${COMPOSE_PROJECT_NAME} -f ${COMPOSE_FILE} --env-file ${ENV_FILE} up -d --force-recreate
                '''
            }
        }
    }


    // siempre se ejecuta después de todo
    post {
        success {
            echo 'Construccion del proyecto exitosa'
        }
        failure {
            echo 'Construccion del proyecto fallida'
        }
    }
}
