pipeline  {
    agent any
    
    environment {
        APP_NAME = "cicd-web-demo"
        STAGING_PORT = "8081"
        PROD_PORT = "8082"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages{

        stage('Checkout'){
            steps{
                echo "Descargando el codigo..."
                checkout scm
            }
        }

        stage('Lint / Validacion'){
            steps{
                echo "Validando estructura minima..."
                sh 'test -f Dockerfile'
                sh 'test -f docker-compose.yml'
                sh 'test -f app/index.html'
                sh 'test -x scripts/test.sh'
                echo "Validación OK"
            }
        }

        stage('Test') {
            steps {
                echo "Ejecutando pruebas..."
                sh './scripts/test.sh'
            }
        }

        stage('Build Imagen (staging)') {
            steps {
                echo "Construyendo imagen para staging..."
                sh "docker build -t ${APP_NAME}:staging ."
            }
        }

        stage('Deploy a Staging') {
            steps {
                echo "Desplegando en STAGING (puerto ${STAGING_PORT})..."
                sh 'docker compose up -d web-staging'
                echo "Staging actualizado. Verifica en: http://127.0.0.1:8081"
            }
        }

        stage('Aprobacion para Produccion'){
            steps{
                input message: '¿Aprobar despliegue a PRODUCCION?', ok: 'Si, desplegar'
            }
        }

        stage('Promover Image a Produccion'){
            steps{
                echo 'Promoviendo imagen a produccion...'
                sh "docker tag ${APP_NAME}:staging ${APP_NAME}:production"
            }
        }

        stage('Deploy a Produccion'){
            steps{
                echo "Desplegando en PRODUCCION (puerto ${PROD_PORT})"
                sh 'docker compose up -d web-production'
                echo "Produccion actualizada. Verifica en http://127.0.0.1:8082"
            }
        }
    }

    post {
        success {
            echo "CI/CD finalizado correctamente"
        }
        failure {
            echo "CI/CD falló. Revisar logs del build."
        }
        always {
            sh 'docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}" || true'
        }
    }
}

    