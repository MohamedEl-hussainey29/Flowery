pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID='dockerhub-credentials'
        FIREBASE_CREDENTIALS_ID='firebase-service-account'
        ENV_CREDENTIALS_ID='backend-env-file'
        
        DOCKER_USER              = 'amrnagy74'
        IMAGE_TAG                = "v${BUILD_NUMBER}"
    }

    def services = [
        [name: 'identity',    path: 'src/Services/IdentityService/Dockerfile',    context: 'backend'],
        [name: 'catalog',     path: 'src/Services/CatalogService/Dockerfile',     context: 'backend'],
        [name: 'addresscart', path: 'src/Services/AddressCartService/Dockerfile', context: 'backend'],
        [name: 'orders',      path: 'src/Services/OrdersService/Dockerfile',      context: 'backend'],
        [name: 'payment',     path: 'src/Services/PaymentService/Dockerfile',     context: 'backend'],
        [name: 'api-gateway', path: 'src/ApiGateway/Dockerfile',                  context: 'backend'], 
        [name: 'frontend',    path: 'Dockerfile',                                 context: 'frontend']
    ]

    stages {
        stage('1. Checkout') { steps { checkout scm } }

        stage('2. Inject Secrets') {
            steps {
                dir('backend') {
                    sh 'mkdir -p secrets'
                    withCredentials([
                        file(credentialsId: env.ENV_CREDENTIALS_ID, variable: 'ENV_FILE'),
                        file(credentialsId: env.FIREBASE_CREDENTIALS_ID, variable: 'FIREBASE_FILE')
                    ]) {
                        sh 'cp "$ENV_FILE" .env && cp "$FIREBASE_FILE" secrets/firebase-service-account.json'
                    }
                }
            }
        }

        stage('3. Run Tests') {
            steps { 
                sh 'dotnet test backend/FlowersApp.slnx --configuration Release || true' 
            }
        }

        stage('4. Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                    
                    script {
                        services.each { s ->
                            dir(s.context) {
                                echo "Building ${s.name}..."
                                sh "docker build -t $DOCKER_USER/flowerapp-${s.name}:$IMAGE_TAG -t $DOCKER_USER/flowerapp-${s.name}:latest -f ${s.path} ."
                                echo "Pushing ${s.name}..."
                                sh "docker push $DOCKER_USER/flowerapp-${s.name}:$IMAGE_TAG"
                                sh "docker push $DOCKER_USER/flowerapp-${s.name}:latest"
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        always { sh 'docker image prune -f' }
    }
}
