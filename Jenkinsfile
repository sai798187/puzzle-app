pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rogan07sg/puzzle-app"
        NEXUS_URL = "http://13.127.65.135:8081"
        SONARQUBE_ENV = "SQ"
    }

    stages {

        stage('Build Artifact') {
            steps {
                sh 'echo "Building app..."'
                sh 'tar -czf app.tar.gz *'   
            }
        }

       stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    script {
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=puzzle-app \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=venv/**,__pycache__/**
                        """
                }
            }
        }
		}

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                curl -u admin:rogan@123 --upload-file app.tar.gz \
                ${NEXUS_URL}/repository/raw-repo/app.tar.gz
                '''
            }
        }
		
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: "USER",
                    passwordVariable: "PASS"
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    sed -i "s|IMAGE_TAG|$BUILD_NUMBER|g" k8s/deployment.yaml
                    kubectl apply -f k8s/
                    '''
                }
            }
        }
    }
}
