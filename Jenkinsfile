pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t puzzle-app .'
            }
        }

        stage('Push') {
            steps {
                sh 'docker tag puzzle-app rogan07sg/puzzle-app'
                sh 'docker push rogan07sg/puzzle-app'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
    sh 'kubectl get nodes'
    sh 'kubectl apply -f k8s/'
}
            }
        }
    }
}
