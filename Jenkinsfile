pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rogan07sg/puzzle-app"
        NEXUS_URL = "http://<nexus-ip>:8081"
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
                    sh '''
                     /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar-scanner/bin/sonar-scanner \
                    -Dsonar.projectKey=puzzle-app \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
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
                    credentialsId: docker-creds,
                    usernameVariable: rogan07sg,
                    passwordVariable: docker-cred
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
